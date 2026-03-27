---
description: Add a transactional email with Handlebars template and fire-and-forget sendEmail
---

1. Read `docs/skills/08-send-email.md` for the fire-and-forget rule and template variable conventions.

2. Ask the user for:
   - Email purpose (invitation, password reset, notification, etc.)
   - Recipient source (from user record, from request body, hardcoded admin)
   - Subject line (must NOT contain PHI — no patient names, DOBs, diagnoses)
   - Whether to use an existing template or create a new one
   - Template variables needed

3. **If creating a new template**, create the HTML file at `src/templates/email-templates/<name>.html` using Handlebars syntax. Use the standard variables: `receiver_name`, `logo_image_url`, `banner_image_url`, `btn_name`, `action_url`.

4. **In the controller**, add the email sending block:
   - `path.join(__dirname, '../../../../templates/email-templates/<name>.html')` — verify the relative depth matches the controller's location
   - Compile with `handlebars.compile(source)`
   - All URLs use `FRONTEND_URL` from `@config/index`
   - Call `sendEmail()` WITHOUT `await` (fire-and-forget)
   - Guard the recipient email: check non-null, non-empty string before sending
   - Log an error if email can't be sent

5. Validate:
// turbo
```bash
cd subqdocs-backend && npx tsc --noEmit
```

6. Report the template file, subject line, and controller modified.
