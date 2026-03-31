# Skill: Query Patient Data Safely

**When NOT to use this:** Querying non-patient data models (users, organizations, visit types) — `organization_id` scoping still applies but PHI rules do not. Internal admin scripts that intentionally operate across all orgs — must still never hard-delete.

---

## Non-negotiable rules

1. **Every patient/visit query must include `organization_id: loggedInUser.organization_id`** in the WHERE clause.
2. **Never hard-delete** — both `patients` and `patient_visits` use `paranoid: true`. Use the repository `deleteData` function.
3. **Never return PHI fields** that aren't required by the endpoint.
4. **Always call `parse(model)`** before returning a Sequelize instance.

---

## Fetching a single patient

```ts
import { getPatientRepo } from '@repositories/patient.repository';
import { parse } from '@utils/common.utils';

// ✅ org-scoped — cannot return a patient from another org
const patient = await getPatientRepo({
    where: {
        id: patient_id,
        organization_id: loggedInUser.organization_id,  // ← MANDATORY
    },
});

if (!patient) {
    return generalResponse(res, null, 'Patient does not exist', 'error', true, 200);
}

return generalResponse(res, parse(patient), 'Fetched', 'success', false, 200);
```

From `patient-visit.controller.ts` (exact):
```ts
const patient = await getPatientRepo({
    where: { id: patient_id, organization_id: loggedInUser?.organization_id },
});
if (!patient) {
    return generalResponse(res, null, 'Patient does not exist', 'error', true, 200);
}
```

---

## Fetching a visit with org-scope enforced via join

When you need the visit directly (without `organizationMemberMiddleware` having already verified it):

```ts
import { getPatientVisitRepo } from '@repositories/patient_visit.repository';
import Patient from '@models/patient.model';

const visit = await getPatientVisitRepo({
    where: { id: visit_id },
    include: [{
        model: Patient,
        required: true,               // INNER JOIN — if no patient match, returns null
        attributes: [],               // don't pull patient columns into the result
        where: { organization_id: loggedInUser.organization_id },
    }],
});
```

From `auth.middleware.ts` (exact):
```ts
const visit = await getPatientVisitRepo({
    where: { id: parsedVisitId },
    include: [{ model: Patient, required: true, attributes: [], where: { organization_id: loggedInUser?.organization_id } }],
});
```

---

## Fetching a list with pagination

```ts
const { page = 1, limit = 20 } = req.query;
const offset = (Number(page) - 1) * Number(limit);

const { count, rows } = await PatientRepo.getAllData({
    where: {
        organization_id: loggedInUser.organization_id,  // ← MANDATORY
        ...(search ? { first_name: { [Op.iLike]: `${search}%` } } : {}),
    },
    attributes: ['id', 'first_name', 'last_name', 'age', 'gender', 'existing_patient'],
    limit: Number(limit),
    offset,
    order: [['created_at', 'DESC']],
});

return generalResponse(res, { count, rows: parse(rows) }, 'Fetched', 'success', false, 200);
```

---

## Soft delete (not hard delete)

```ts
// ✅ Safe — sets deleted_at (paranoid: true)
await deletePatientRepo({
    where: { id: patient_id, organization_id: loggedInUser.organization_id },
});

// ❌ NEVER — bypasses paranoid
// await sequelize.query('DELETE FROM patients WHERE id = ?', { replacements: [patient_id] });
```

---

## PHI fields — never include unless the endpoint explicitly requires them

> See `docs/kb/project/overview.md` (section **PHI Fields — Never Log or Expose**) for the canonical, comprehensive list.

```
patients table:
  date_of_birth, contact_no, cellphone, home_phone,
  email, address, street_address, zipcode,
  race, ethnicity, gender_identity, sexual_orientation,
  profile_image (S3 URL), sticky_note

users table:
  password, secret_2fa, otp, otp_token, pin, token,
  optum_password, optum_username
```

All of these are safe to include in the `attributes` exclude list or simply never reference in the `attributes` array.

---

## When organizationMemberMiddleware is already on the route

If the route uses `organizationMemberMiddleware`, it has already verified that any `patient_id` or `visit_id` in `req.params`, `req.query`, or `req.body` belongs to the logged-in user's org. In that case you do not need the redundant `organization_id` WHERE check for the initial patient/visit lookup — but you still need it on any **secondary** queries inside the same controller.

---

## Checklist
- [ ] `organization_id: loggedInUser.organization_id` in WHERE for every patient/visit query
- [ ] `parse(result)` called before returning any model instance or array
- [ ] Soft delete via repository `deleteData` (not raw SQL or `Model.destroy({ force: true })`)
- [ ] `attributes` list excludes PHI fields that aren't needed by this endpoint
- [ ] Join-based org scope (`required: true` with `where: { organization_id }`) used when hitting visits directly
- [ ] No raw `sequelize.query('SELECT * FROM patients ...')` without org_id in WHERE
