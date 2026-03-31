# SubQDocs — anti-patterns and forbidden patterns

**Last reviewed:** 2026-03-31 · **Doc set:** 1

> **Parent:** [`project/overview.md`](./project/overview.md), [`.agents/rules/production-rules.md`](../../.agents/rules/production-rules.md)  
> **Conventions:** [`frontend/conventions.md`](./frontend/conventions.md), [`backend/conventions.md`](./backend/conventions.md)

Use this file for **explicit “never”** lists and deprecated choices. Hard enforcement rules stay in `production-rules.md`; this doc expands **routing** and **style** bans for agents.

---

## `<anti_patterns>`

- **Never** use `React.lazy()` directly — use `lazyWithRetry()` from `helper/laxywithRetry.tsx`.
- **Never** read `process.env.*` in application code — use `config/index.ts` (backend) or the project’s config pattern.
- **Never** use `console.log` / `console.warn` / `console.error` / `console.info` — use `logger` from `@utils/logger` (backend).
- **Never** put server-fetched data in Redux — use TanStack Query (`useQuery` / `useMutation`).
- **Never** call `res.json()` or `res.send()` in Express controllers — use `generalResponse()`.
- **Never** return raw Sequelize instances from controllers — use `parse()` from `@utils/common.utils`.
- **Never** add frontend routes outside `src/constants/routePath.tsx`.
- **Never** add SVG icons outside `src/components/common/svg/Svg.tsx`.
- **Never** use raw `axios` for API calls — use typed wrappers in `src/api/` (`axiosGet`, etc.).
- **Never** mix Formik and React Hook Form in the **same** component.
- **Never** `await sendEmail(...)` — fire-and-forget (see production rules).
- **Never** add `NOT NULL` without `defaultValue` on new columns on **existing** tables without migration-safety review.
- **TypeScript:** avoid `any` for new code — prefer explicit interfaces/types; narrow with `unknown` + guards when needed.
- **Do not** introduce new usages of deprecated packages without team review — prefer versions in each app’s `package.json` and existing patterns.

</anti_patterns>

---

## If task X → open doc Y

| Task wording / area | Open first |
|---------------------|------------|
| New backend module, migration, model | [`../skills/INDEX.md`](../skills/INDEX.md) → 01, 17; [`subqdocs-backend/INDEX.md`](./subqdocs-backend/INDEX.md) |
| New frontend page / route | Skills 02, 11; [`subqdocs-frontend/routing-and-state.md`](./subqdocs-frontend/routing-and-state.md) |
| File upload / S3 | Skill 04 |
| Sockets | Skill 06 |
| Patient / PHI queries | Skill 10; [`project/overview.md`](./project/overview.md) |
| Figma / UI verification | Skills 12, 16 |
| BullMQ / jobs | Skill 07 |
| Email | Skill 08 |

---

## Maintenance

When you **deprecate** a library, **ban** a pattern repo-wide, or **add** a new “never,” update this file in the same change set as the code or rule change. Cross-reference [`maintenance.md`](./maintenance.md).
