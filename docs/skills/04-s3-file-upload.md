# Skill: File Upload to S3

**When NOT to use this:** Generating a PDF in-memory and returning it directly in the HTTP response without storing it. Serving an already-stored file — use `refreshS3UrlPath()` (see below).

---

## Two buckets, three upload functions

| Bucket | Function to use | Key prefix rule |
|---|---|---|
| Main (audio, images, PDFs, attachments) | `uploadFileToS3(buffer, key)` | `uploads/<category>/...` |
| Main (PDF only, returns signed URL) | `uploadPDFToS3(buffer, key)` | `uploads/pdfs/...` |
| eFax | `uploadFileToEfaxS3(buffer, key, contentType)` | **Must** start with `efax-attachment/` |

All three are exported from `src/common/utils/s3/s3.ts` (the legacy facade — import from here, not from `src/common/s3/` directly).

---

## General upload: buffer → S3 → signed URL → DB

From `user.controller.ts` (exact):
```ts
import fs from 'fs';
import { uploadFileToS3, getSignedUrl1 } from '@utils/s3/s3';

const { originalFilename, path: filePath } = userImage;
const buffer = fs.readFileSync(filePath);
const fileKey = `uploads/user_image/${loggedInUser.id}-${Date.now()}-${originalFilename}`;

await uploadFileToS3(buffer, fileKey);
const signedUrl = await getSignedUrl1(fileKey);

userData['profile_image'] = signedUrl;
userData['uploaded_at'] = new Date();   // ← REQUIRED alongside every signed URL
```

`uploaded_at` is required — `refreshS3UrlPath()` uses it to decide when to renew the URL.

---

## PDF upload (returns signed URL already)

```ts
import { uploadPDFToS3 } from '@utils/s3/s3';

const signedUrl = await uploadPDFToS3(pdfBuffer, `uploads/pdfs/${name}-${Date.now()}.pdf`);
// No need to call getSignedUrl1 separately
```

---

## eFax upload

```ts
import { uploadFileToEfaxS3 } from '@utils/s3/s3';

const signedUrl = await uploadFileToEfaxS3(
    buffer,
    `efax-attachment/${visitId}-${Date.now()}.pdf`,  // prefix enforced by efaxS3Service
    'application/pdf',
);
```

Using `uploadFileToS3` for eFax bypasses the prefix validation in `efaxS3Service` — use `uploadFileToEfaxS3`.

---

## Multipart upload (large audio files — > a few MB)

```ts
import { initiateS3MultipartUpload, uploadPartCopyToS3, completeS3MultipartUpload } from '@utils/s3/s3';

const uploadId = await initiateS3MultipartUpload(fileKey);
const etag = await uploadPartCopyToS3(uploadId, partNumber, sourceKey, targetKey);
await completeS3MultipartUpload(uploadId, parts, fileKey);
```

The `chunkProcessWorker.ts` uses `uploadFileToS3` for already-merged audio buffers (the merge produces a single WAV file before upload).

---

## Multer routes: mandatory body-parser bypass

Routes that receive files via `multipart/form-data` must be excluded from `express-form-data` in `src/app.ts`. If the route is not on the bypass list, `express-form-data` consumes the stream before Multer can parse it.

Check `src/app.ts` for the existing bypass array and add your route path there.

File validation middleware to apply **before** the controller:

```ts
import { validationAttachments } from '@middlewares/middleware';   // images/PDFs ≤ 15 MB each, ≤ 100 MB total
import { validateAudioFile }    from '@middlewares/middleware';   // audio/video MIME types

router.post('/my-upload', authMiddleware, validationAttachments(), myController);
```

---

## Serving stored files: refreshS3UrlPath

When a GET endpoint returns a field that holds a signed URL, call `refreshS3UrlPath` — not `getSignedUrl1`:

```ts
import { refreshS3UrlPath } from '@utils/common.utils';

userDetails.profile_image = userDetails?.profile_image
    ? await refreshS3UrlPath(userDetails.profile_image, userDetails.uploaded_at, userDetails.id, 'User', User)
    : null;
```

The third and fourth arguments are the entity's primary key and model class. Check the existing callers in `user.controller.ts` for the correct model names (`'User'`, `'TranscriptFile'`, `'Attachments'`).

---

## Checklist
- [ ] Import from `@utils/s3/s3`, not from `@common/s3`
- [ ] `uploaded_at: new Date()` stored in DB alongside every signed URL
- [ ] eFax uses `uploadFileToEfaxS3` with `efax-attachment/` key prefix
- [ ] Multer route added to bypass list in `src/app.ts`
- [ ] `validationAttachments()` or `validateAudioFile()` applied before controller
- [ ] GET endpoints call `refreshS3UrlPath()` before returning file URLs
- [ ] Large audio files use multipart API, not `uploadFileToS3`
