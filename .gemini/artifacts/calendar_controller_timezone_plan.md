# 🕐 Timezone-Aware Calendar Controller — Final Plan

## Core Strategy

**Use only `start_date` and `end_date` for both date AND time.** These two `timestamp with time zone` fields hold everything. Stop using `start_time`/`end_time` columns.

**Payload can change** — responses will return `start_date`/`end_date` as full ISO timestamps. No need to decompose into separate date+time fields.

---

## 📊 Current vs New

| | Current | New |
|---|---------|-----|
| **`start_date`** | `2026-02-17 05:30:00+05:30` (midnight — date only) | `2026-02-17 19:30:00+05:30` (date + time combined) |
| **`end_date`** | `2026-02-17 05:30:00+05:30` (midnight — date only) | `2026-02-17 22:30:00+05:30` (date + time combined) |
| **`start_time`** | `19:30:00` (actual time) | **Not used** |
| **`end_time`** | `22:30:00` (actual time) | **Not used** |
| **Model type** | `DataType.DATEONLY` (strips time!) | `DataType.DATE` (preserves time+tz) |

---

## 🔧 All Changes (7 Steps)

---

### Step 1: Create Migration — Column Type + Data Backfill

**New file:** `subqdocs-backend/src/sequelize/migrations/YYYYMMDDHHMMSS-update-unavailability-dates-to-timestamp.js`

This migration does two things:

#### 1a. Change column type from `DATE` to `TIMESTAMPTZ` (explicit)

The DB already has `timestamp with time zone` (from the original `Sequelize.DATE` migration), but the model says `DATEONLY`. To be safe and explicit, the migration should ensure the column type is correct. This also documents the intent.

```javascript
"use strict";

/** @type {import('sequelize-cli').Migration} */
module.exports = {
    async up(queryInterface, Sequelize) {
        // Step 1: Ensure columns are TIMESTAMPTZ (they likely already are, 
        // but this makes the migration idempotent and explicit)
        await queryInterface.changeColumn("doctor_unavailability", "start_date", {
            type: Sequelize.DATE,  // TIMESTAMPTZ in PostgreSQL
            allowNull: true,
        });
        await queryInterface.changeColumn("doctor_unavailability", "end_date", {
            type: Sequelize.DATE,  // TIMESTAMPTZ in PostgreSQL
            allowNull: true,
        });

        // Step 2: Backfill existing records — combine start_date midnight + start_time
        // into a proper start_date timestamp. Same for end_date + end_time.
        // Uses America/Denver as the default timezone for existing data.
        await queryInterface.sequelize.query(`
            UPDATE doctor_unavailability
            SET 
                start_date = CASE
                    WHEN start_time IS NOT NULL AND start_date IS NOT NULL
                    THEN (start_date::date || ' ' || start_time::text)::timestamp AT TIME ZONE 'America/Denver'
                    ELSE start_date
                END,
                end_date = CASE
                    WHEN end_time IS NOT NULL AND end_date IS NOT NULL
                    THEN (end_date::date || ' ' || end_time::text)::timestamp AT TIME ZONE 'America/Denver'
                    ELSE end_date
                END
            WHERE deleted_at IS NULL;
        `);
    },

    async down(queryInterface, Sequelize) {
        // Reverse: extract time back into start_time/end_time, reset dates to midnight
        await queryInterface.sequelize.query(`
            UPDATE doctor_unavailability
            SET
                start_time = CASE
                    WHEN start_date IS NOT NULL AND (start_date AT TIME ZONE 'America/Denver')::time != '00:00:00'
                    THEN (start_date AT TIME ZONE 'America/Denver')::time
                    ELSE start_time
                END,
                end_time = CASE
                    WHEN end_date IS NOT NULL AND (end_date AT TIME ZONE 'America/Denver')::time != '00:00:00'
                    THEN (end_date AT TIME ZONE 'America/Denver')::time
                    ELSE end_time
                END,
                start_date = CASE
                    WHEN start_date IS NOT NULL
                    THEN date_trunc('day', start_date AT TIME ZONE 'America/Denver') AT TIME ZONE 'America/Denver'
                    ELSE start_date
                END,
                end_date = CASE
                    WHEN end_date IS NOT NULL
                    THEN date_trunc('day', end_date AT TIME ZONE 'America/Denver') AT TIME ZONE 'America/Denver'
                    ELSE end_date
                END
            WHERE deleted_at IS NULL;
        `);
    },
};
```

---

### Step 2: Fix Sequelize Model + Type

**File:** `subqdocs-backend/src/sequelize/models/doctor-unavailability.model.ts`

```diff
-    @Column(DataType.DATEONLY)
-    start_date: string;
+    @Column(DataType.DATE)
+    start_date: Date;

-    @Column(DataType.DATEONLY)
-    end_date: string;
+    @Column(DataType.DATE)
+    end_date: Date;
```

**File:** `subqdocs-backend/src/sequelize/models/types/doctor-unavailability.model.type.ts`

```diff
-    start_date?: string | null; // DATEONLY format: YYYY-MM-DD
-    end_date?: string | null; // DATEONLY format: YYYY-MM-DD
+    start_date?: Date | string | null; // TIMESTAMPTZ — holds date + time combined
+    end_date?: Date | string | null; // TIMESTAMPTZ — holds date + time combined
```

---

### Step 3: Add Helper Function

**File:** `calendar.controller.ts` (near the top, after existing helpers)

```typescript
/**
 * Combines a date string and optional time string into a timezone-aware Date.
 * @param dateStr - Date in YYYY-MM-DD format
 * @param timeStr - Optional time in HH:MM:SS format (defaults to start of day)
 * @param timezone - IANA timezone string
 * @returns Date object representing the exact moment in UTC
 */
const combineDateTimeWithTimezone = (
    dateStr: string,
    timeStr: string | null | undefined,
    timezone: string
): Date => {
    const time = timeStr || "00:00:00";
    return moment.tz(`${dateStr} ${time}`, "YYYY-MM-DD HH:mm:ss", timezone).toDate();
};
```

---

### Step 4: Update WRITE Paths — `createUnavailabilitySlot`

**File:** `calendar.controller.ts`, function `createUnavailabilitySlot` (lines 469–560)

**Add at line ~491:** Read timezone from headers
```typescript
const userTimezone = (req.headers["x-timezone"] as string) || DEFAULT_TIMEZONE;
```

**Replace date/time payload building (lines 498–521):**

Before (current):
```typescript
if (start_time) payload["start_time"] = start_time;
if (end_time) payload["end_time"] = end_time;

if (recurrence_pattern == RecurrencePatternEnum.ONE_TIME) {
    payload["start_date"] = start_date;
    payload["end_date"] = end_date || start_date;
}
if (recurrence_pattern == RecurrencePatternEnum.DAILY) {
    payload["start_date"] = start_date;
    payload["end_date"] = end_date;
}
if (recurrence_pattern == RecurrencePatternEnum.WEEKLY) {
    if (start_date) payload["start_date"] = start_date;
    if (end_date) payload["end_date"] = end_date;
    if (days_of_week) payload["days_of_week"] = days_of_week;
}
if (recurrence_pattern == RecurrencePatternEnum.MONTHLY) {
    if (start_date) payload["start_date"] = start_date;
    if (end_date) payload["end_date"] = end_date;
    if (days_of_week) payload["days_of_week"] = days_of_week;
    if (months) payload["months"] = months;
}
```

After (new):
```typescript
// Do NOT write to start_time/end_time columns — time is embedded in start_date/end_date

if (recurrence_pattern == RecurrencePatternEnum.ONE_TIME) {
    payload["start_date"] = combineDateTimeWithTimezone(start_date, start_time, userTimezone);
    payload["end_date"] = combineDateTimeWithTimezone(end_date || start_date, end_time, userTimezone);
}
if (recurrence_pattern == RecurrencePatternEnum.DAILY) {
    payload["start_date"] = combineDateTimeWithTimezone(start_date, start_time, userTimezone);
    payload["end_date"] = combineDateTimeWithTimezone(end_date, end_time, userTimezone);
}
if (recurrence_pattern == RecurrencePatternEnum.WEEKLY) {
    if (start_date) payload["start_date"] = combineDateTimeWithTimezone(start_date, start_time, userTimezone);
    if (end_date) payload["end_date"] = combineDateTimeWithTimezone(end_date, end_time, userTimezone);
    if (days_of_week) payload["days_of_week"] = days_of_week;
}
if (recurrence_pattern == RecurrencePatternEnum.MONTHLY) {
    if (start_date) payload["start_date"] = combineDateTimeWithTimezone(start_date, start_time, userTimezone);
    if (end_date) payload["end_date"] = combineDateTimeWithTimezone(end_date, end_time, userTimezone);
    if (days_of_week) payload["days_of_week"] = days_of_week;
    if (months) payload["months"] = months;
}
```

**Exception records (lines 524–543):**
```typescript
// Change from:
start_date: exp_date,
end_date: exp_date,
start_time: exception_start_time !== undefined ? exception_start_time : null,
end_time: exception_end_time !== undefined ? exception_end_time : null,

// Change to:
start_date: combineDateTimeWithTimezone(exp_date, exception_start_time, userTimezone),
end_date: combineDateTimeWithTimezone(exp_date, exception_end_time, userTimezone),
// Remove start_time and end_time from exception payload
```

---

### Step 5: Update WRITE Paths — `updateUnavailabilitySlot`

**File:** `calendar.controller.ts`, function `updateUnavailabilitySlot` (lines 562–718)

**Add near line ~581:** Read timezone
```typescript
const userTimezone = (req.headers["x-timezone"] as string) || DEFAULT_TIMEZONE;
```

**Exception record payloads (lines 613–628):**
```typescript
// Change from:
start_date: exp_date,
end_date: exp_date,
start_time: exception_start_time !== undefined ? exception_start_time : null,
end_time: exception_end_time !== undefined ? exception_end_time : null,

// Change to:
start_date: combineDateTimeWithTimezone(exp_date, exception_start_time, userTimezone),
end_date: combineDateTimeWithTimezone(exp_date, exception_end_time, userTimezone),
// Remove start_time and end_time from exception payload
```

**Main update payload (lines 672–701):**
```typescript
// Change from:
if (start_time !== undefined) payload["start_time"] = start_time;
if (end_time !== undefined) payload["end_time"] = end_time;
...
payload["start_date"] = start_date;
payload["end_date"] = start_date;

// Change to (remove start_time/end_time writes, use combineDateTimeWithTimezone for dates):
// Do NOT write to start_time/end_time columns

if (recurrence_pattern == RecurrencePatternEnum.ONE_TIME) {
    payload["start_date"] = combineDateTimeWithTimezone(start_date, start_time, userTimezone);
    payload["end_date"] = combineDateTimeWithTimezone(start_date, end_time, userTimezone);
    payload["days_of_week"] = null;
    payload["months"] = null;
}
if (recurrence_pattern == RecurrencePatternEnum.DAILY) {
    payload["start_date"] = combineDateTimeWithTimezone(start_date, start_time, userTimezone);
    payload["end_date"] = combineDateTimeWithTimezone(end_date, end_time, userTimezone);
    payload["days_of_week"] = null;
    payload["months"] = null;
}
if (recurrence_pattern == RecurrencePatternEnum.WEEKLY) {
    if (start_date !== undefined) payload["start_date"] = combineDateTimeWithTimezone(start_date, start_time, userTimezone);
    if (end_date !== undefined) payload["end_date"] = combineDateTimeWithTimezone(end_date, end_time, userTimezone);
    if (days_of_week !== undefined) payload["days_of_week"] = days_of_week;
    payload["months"] = null;
}
if (recurrence_pattern == RecurrencePatternEnum.MONTHLY) {
    if (start_date !== undefined) payload["start_date"] = combineDateTimeWithTimezone(start_date, start_time, userTimezone);
    if (end_date !== undefined) payload["end_date"] = combineDateTimeWithTimezone(end_date, end_time, userTimezone);
    if (days_of_week !== undefined) payload["days_of_week"] = days_of_week;
    if (months !== undefined) payload["months"] = months;
}
```

---

### Step 6: Update WRITE Paths — `deleteUnavailabilitySlot`

**File:** `calendar.controller.ts`, function `deleteUnavailabilitySlot` (lines 720–803)

**Add near line ~724:** Read timezone
```typescript
const userTimezone = (req.headers["x-timezone"] as string) || DEFAULT_TIMEZONE;
```

**Cancellation exception payload (lines 760–775):**
```typescript
// Change from:
start_date: exceptionDateStr,
end_date: exceptionDateStr,
start_time: unavailabilitySlot.start_time,
end_time: unavailabilitySlot.end_time,

// Change to: Extract time from parent's start_date/end_date timestamps
const parentStartTime = unavailabilitySlot.start_date
    ? moment(unavailabilitySlot.start_date).tz(userTimezone).format("HH:mm:ss")
    : null;
const parentEndTime = unavailabilitySlot.end_date
    ? moment(unavailabilitySlot.end_date).tz(userTimezone).format("HH:mm:ss")
    : null;

// In payload:
start_date: combineDateTimeWithTimezone(exceptionDateStr, parentStartTime, userTimezone),
end_date: combineDateTimeWithTimezone(exceptionDateStr, parentEndTime, userTimezone),
// Remove start_time and end_time from payload
```

---

### Step 7: Update READ Paths — `generateAvailableSlotsForDate`

**File:** `calendar.controller.ts`, function `generateAvailableSlotsForDate` (lines 1032–1586)

#### 7a. Fix `parseDateStringAsLocal` (lines 1016–1030)

```typescript
// Change from:
const parseDateStringAsLocal = (dateStr: string): { date: Date; dayOfWeek: string } => {
    // ...
    const date = new Date(year, month - 1, day); // SERVER timezone!
    const dayOfWeek = date.toLocaleDateString("en-US", { weekday: "long" }).toLowerCase();
    return { date, dayOfWeek };
};

// Change to:
const parseDateStringAsLocal = (dateStr: string, timezone: string = DEFAULT_TIMEZONE): { date: Date; dayOfWeek: string } => {
    if (!/^\d{4}-\d{2}-\d{2}$/.test(dateStr)) {
        throw new Error(`Invalid date format. Expected YYYY-MM-DD, got: ${dateStr}`);
    }
    const m = moment.tz(dateStr, "YYYY-MM-DD", timezone);
    const date = m.toDate();
    const dayOfWeek = m.format("dddd").toLowerCase();
    return { date, dayOfWeek };
};
```

**Update ALL callers to pass timezone:**
| Line | Current | New |
|------|---------|-----|
| 1047 | `parseDateStringAsLocal(dateStr)` | `parseDateStringAsLocal(dateStr, timezone)` |
| 1174 | `parseDateStringAsLocal(dateStr)` | `parseDateStringAsLocal(dateStr, timezone)` |
| 1707 | `parseDateStringAsLocal(dateStr)` | `parseDateStringAsLocal(dateStr, userTimezone)` |
| 1801 | `parseDateStringAsLocal(searchDateStr)` | `parseDateStringAsLocal(searchDateStr, userTimezone)` |

#### 7b. Fix unavailability query (lines 1177–1206)

```typescript
// Add before the query:
const dayStart = moment.tz(dateStr, "YYYY-MM-DD", timezone).startOf("day").toDate();
const dayEnd = moment.tz(dateStr, "YYYY-MM-DD", timezone).endOf("day").toDate();

// Change from:
{
    pattern: RecurrencePatternEnum.ONE_TIME,
    start_date: dateStr,  // string vs timestamp comparison — BROKEN
}

// Change to:
{
    pattern: RecurrencePatternEnum.ONE_TIME,
    start_date: { [Op.between]: [dayStart, dayEnd] },
}

// For DAILY/WEEKLY/MONTHLY:
// Change from:
start_date: { [Op.lte]: dateStr },
[Op.or]: [{ end_date: { [Op.gte]: dateStr } }, { end_date: null }],

// Change to:
start_date: { [Op.lte]: dayEnd },
[Op.or]: [{ end_date: { [Op.gte]: dayStart } }, { end_date: null }],
```

#### 7c. Fix unavailability time extraction (lines 1237–1268)

```typescript
// Change from:
if (!slot.start_time || !slot.end_time) {
    // ... full-day logic (keep unchanged)
} else {
    startMinutes = timeStringToMinutes(slot.start_time);
    endMinutes = timeStringToMinutes(slot.end_time);
}

// Change to:
// Extract time from start_date/end_date timestamps
const slotStartMoment = slot.start_date ? moment(slot.start_date).tz(timezone) : null;
const slotEndMoment = slot.end_date ? moment(slot.end_date).tz(timezone) : null;
const hasStartTime = slotStartMoment && slotStartMoment.format("HH:mm:ss") !== "00:00:00";
const hasEndTime = slotEndMoment && slotEndMoment.format("HH:mm:ss") !== "00:00:00";

if (!hasStartTime || !hasEndTime) {
    // ... full-day logic (keep unchanged — uses office hours range)
} else {
    startMinutes = slotStartMoment.hours() * 60 + slotStartMoment.minutes() + slotStartMoment.seconds() / 60;
    endMinutes = slotEndMoment.hours() * 60 + slotEndMoment.minutes() + slotEndMoment.seconds() / 60;
}
```

**Same change** for `addedExceptionSlots` processing block (lines 1317–1394) — replace `exception.start_time`/`exception.end_time` with time extracted from `exception.start_date`/`exception.end_date`.

#### 7d. Fix `findSlotsForDoctorLocation` (line 1796)

```typescript
// Change from:
return { slots: [], date: startDate.toISOString().split("T")[0] };

// Change to:
return { slots: [], date: moment(startDate).tz(userTimezone).format("YYYY-MM-DD") };
```

---

## 📋 Final Checklist

| # | File | Change | Complexity |
|---|------|--------|------------|
| 1 | **New migration file** | Column type assert + data backfill (combine `start_date` + `start_time` → timestamp) | High |
| 2 | `doctor-unavailability.model.ts` | `DATEONLY` → `DATE`, `string` → `Date` | Low |
| 2b | `doctor-unavailability.model.type.ts` | `string` → `Date \| string` | Low |
| 3 | `calendar.controller.ts` (top) | Add `combineDateTimeWithTimezone` helper | Low |
| 4 | `calendar.controller.ts` — `createUnavailabilitySlot` | Use helper, remove `start_time`/`end_time` writes | Medium |
| 5 | `calendar.controller.ts` — `updateUnavailabilitySlot` | Use helper, remove `start_time`/`end_time` writes | Medium |
| 6 | `calendar.controller.ts` — `deleteUnavailabilitySlot` | Extract time from parent's `start_date`, use helper | Medium |
| 7a | `calendar.controller.ts` — `parseDateStringAsLocal` | Add timezone param, use `moment.tz` | Medium |
| 7b | `calendar.controller.ts` — unavailability query | Timezone-aware day boundaries | High |
| 7c | `calendar.controller.ts` — unavailability time extraction | Read time from `start_date`/`end_date` instead of `start_time`/`end_time` | High |
| 7d | `calendar.controller.ts` — `findSlotsForDoctorLocation` | Fix `toISOString()` → `moment.tz().format()` | Low |

**Total: 3 files modified, 1 file created**
