# Skill: Visit Time and Date Handling

**When NOT to use this:** Reading or writing `visit_date` alone (it is stored as the local date the user entered, no UTC conversion needed). Reading `created_at` or `updated_at` timestamps (standard Sequelize handling).

---

## The storage contract

| Column | DB type | Stored as | Never store |
|---|---|---|---|
| `visit_date` | `DATE` | `YYYY-MM-DD` — the local calendar date the provider entered | ISO timestamp, Date object |
| `visit_time` | `TIME` | `HH:MM:SS` in **UTC** | Time with TZ offset, full ISO string |
| `end_time` | `TIME` | `HH:MM:SS` in **UTC** | Same as above |

`visit_date` is **never shifted to UTC** — only `visit_time` and `end_time` are normalized.

---

## Writing visit_time / end_time (create or update)

```ts
import { getFormattedVisitDateAndTime, getFormattedEndTime } from '@utils/common.utils';

// Handles all three frontend formats:
//   "14:30:00+05:30"   → strips offset, converts to UTC
//   "14:30:00"         → stored as-is
//   "2026-01-16 00:26:00.000 -0700" → converts to UTC time portion
const [formattedVisitDate, formattedVisitTime] = getFormattedVisitDateAndTime(visit_date, visit_time);
payload.visit_date = formattedVisitDate;
payload.visit_time = formattedVisitTime;

// end_time always needs visit_date to handle midnight crossings (23:30 + 1h = 00:30 next day)
const formattedEndTime = end_time ? getFormattedEndTime(visit_date, end_time) : null;
payload.end_time = formattedEndTime;
```

From `patient-visit.controller.ts` (exact):
```ts
if (visit_date && visit_time) {
    const [formattedVisitDate, formattedVisitTime] = getFormattedVisitDateAndTime(visit_date, visit_time);
    payload.visit_date = formattedVisitDate;
    payload.visit_time = formattedVisitTime;
}

if (end_time !== undefined) {
    const dateForEndTime = visit_date || (scheduledVisit?.visit_date
        ? moment(scheduledVisit.visit_date).format('YYYY-MM-DD')
        : null);
    if (dateForEndTime) {
        const formattedEndTime = end_time ? getFormattedEndTime(dateForEndTime, end_time) : null;
        payload.end_time = formattedEndTime;
    }
}
```

---

## Recalculating end_time when visit_time changes

When rescheduling, always recalculate `end_time` from the `VisitType.default_duration`:

```ts
import moment from 'moment-timezone';

const visitType = await VisitType.findOne({ where: { id: visitTypeId, is_deleted: false } });
const durationMinutes = visitType?.default_duration > 0 ? visitType.default_duration : 15;

const visitMoment = moment.utc(payload.visit_time, 'YYYY-MM-DD HH:mm:ss');
payload.end_time = visitMoment.clone().add(durationMinutes, 'minutes').format('HH:mm:ss');
```

---

## Reading visit_time for display (calendar / list views)

The frontend sends `x-timezone` in every request (set by `src/api/axios.ts`). Backend display endpoints must convert UTC → local:

```ts
import moment from 'moment-timezone';

const convertUTCToLocalTime = (utcTime: string, visitDate: string, timezone: string): string => {
    return moment.utc(`${visitDate} ${utcTime}`, 'YYYY-MM-DD HH:mm:ss')
        .tz(timezone)
        .format('HH:mm:ss');
};

// In controller:
const timezone = (req.headers['x-timezone'] as string) || 'America/Chicago';
visit.visit_time = convertUTCToLocalTime(visit.visit_time, visit.visit_date, timezone);
```

This function lives in `src/modules/Calendar/controller/calendar.controller.ts`.

---

## Time-range filtering

`visit_time` is a `TIME` column (not a full timestamp). For appointment-window queries, combine `visit_date` and `visit_time`:

```ts
// ✅ Correct — filter by date then compare time
where: {
    visit_date: targetDate,
    visit_time: { [Op.between]: [startTimeUTC, endTimeUTC] },
}

// ❌ Wrong — visit_date has no time component, can't do datetime range on it alone
where: { visit_date: { [Op.between]: [startDateTime, endDateTime] } }
```

---

## Checklist
- [ ] `getFormattedVisitDateAndTime()` called for every `visit_time` write
- [ ] `getFormattedEndTime(dateForEndTime, end_time)` called with the date string, not a Date object
- [ ] Time-range DB filters target `visit_time`, not `visit_date`
- [ ] Display endpoints read `x-timezone` header with `|| 'America/Chicago'` fallback
- [ ] `end_time` recalculated when `visit_time` changes using `VisitType.default_duration`
- [ ] `moment-timezone` used for conversion, not `new Date()`
