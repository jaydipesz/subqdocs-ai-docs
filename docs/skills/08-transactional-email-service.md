# Skill: Send a Transactional Email

**When NOT to use this:** Sending to many recipients at once (use the `email-triggers` queue in `src/modules/email-triggers/`). Real-time in-app notifications (use socket events, skill 06).

---

## Files involved
```
subqdocs-backend/src/
  common/utils/nodemailer/nodemailer.mail.ts   ← sendEmail()
  templates/email-templates/*.html             ← Handlebars templates
```

---

## The one rule: never await sendEmail

`sendEmail` must be **fire-and-forget**. Awaiting it blocks the HTTP response and turns an SMTP failure into a 500 for the user.

```ts
// ✅ Correct
sendEmail({ receiverEmail: email, subject: '...', html: htmlTemplate });

// ❌ Wrong
await sendEmail({ receiverEmail: email, subject: '...', html: htmlTemplate });
```

---

## Standard pattern (with Handlebars template)

From `user.controller.ts` invitation email (exact):

```ts
import fs from 'fs';
import path from 'path';
import handlebars from 'handlebars';
import { sendEmail } from '@utils/nodemailer/nodemailer.mail';
import { FRONTEND_URL } from '@config/index';

const filePath = path.join(__dirname, '../../../../templates/email-templates/invitationRequestEmail.html');
const source = fs.readFileSync(filePath, 'utf-8');
const template = handlebars.compile(source);
const htmlTemplate = template({
    receiver_name: recipientName,
    receiver_email: recipientDetail?.email,
    sender_name: loggedInUser?.first_name + ' ' + loggedInUser?.last_name,
    login_url: `${FRONTEND_URL}/login`,
    accept_url: `${FRONTEND_URL}/invitation/${createInviteRequest?.id}/${token}`,
    org_name: organizationDetails?.name || organizationDetails?.id,
    logo_image_url: `${FRONTEND_URL}/logo.png`,
    banner_image_url: `${FRONTEND_URL}/banner.png`,
    btn_name: 'Accept Invitation',
    isInvitationAcceptRequest: true,
    isShowLoginSection: true,
});

sendEmail({
    receiverEmail: email,
    subject: `Invitation to join ${organizationDetails?.name} on subQdocs`,
    html: htmlTemplate,
});
```

The relative `path.join(__dirname, '../../../../templates/...')` from a controller in `src/modules/<module>/controller/` reaches `src/templates/email-templates/`. Adjust the `../../../../` depth for your file location.

---

## Validate before sending

If the recipient email comes from DB data (could be null for unfinished profiles), guard it:

```ts
// Pattern used throughout user.controller.ts
const recipientEmail = user?.email;
if (recipientEmail && typeof recipientEmail === 'string' && recipientEmail.trim().length > 0) {
    sendEmail({
        receiverEmail: recipientEmail.trim(),
        subject: 'Subject',
        html: htmlTemplate,
    });
} else {
    logger.error(`Cannot send email: no valid email for user ${userId}`, { userId, email: recipientEmail });
}
```

---

## Simple text email (no template)

```ts
sendEmail({
    receiverEmail: fromUser.email,
    message: `${user.first_name} has accepted your invitation and joined the organization`,
});
```

---

## Common template variables

All existing templates use these keys consistently:
- `receiver_name`, `receiver_email`, `sender_name`, `org_name`
- `logo_image_url` → `${FRONTEND_URL}/logo.png`
- `banner_image_url` → `${FRONTEND_URL}/banner.png` (or `banner4.png` for password reset)
- `accept_url`, `login_url`, `forgot_password_url` — CTA deep links using `FRONTEND_URL`
- `btn_name` — CTA button label

---

## PHI rule

Patient names, dates of birth, diagnoses, and any clinical data must **never** appear in the email `subject` line. They may appear in the HTML body only when required for an explicit clinical workflow (e.g., patient intake confirmation).

---

## Checklist
- [ ] `sendEmail(...)` called without `await`
- [ ] Recipient email validated before sending (non-null, non-empty string)
- [ ] All URLs use `FRONTEND_URL` from `@config/index`, not hardcoded strings
- [ ] `path.join(__dirname, ...)` depth verified for the controller's location
- [ ] No PHI in the `subject` string
- [ ] `logger.error(...)` written when email cannot be sent (audit trail)
