---
description: Add a file upload endpoint end-to-end (Multer route + S3 upload + signed URL storage)
---

1. Read `docs/skills/04-s3-multipart-upload.md` for bucket rules and the Multer bypass requirement.

2. Ask the user for:
   - What entity the file belongs to (patient attachment, org logo, user profile, eFax)
   - File types allowed (images, PDFs, audio)
   - Max file size
   - S3 key prefix (e.g. `uploads/lab_results/`)

3. **Backend route** — add the route in the module's route file:
   - Apply `validationAttachments()` or `validateAudioFile()` middleware before the controller
   - If using Multer directly, add the route path to the bypass list in `src/app.ts`

4. **Backend controller** — follow the exact pattern from `user.controller.ts`:
   - Read file buffer: `fs.readFileSync(file.path)`
   - Build S3 key: `uploads/<category>/<userId>-${Date.now()}-${originalFilename}`
   - Upload: `await uploadFileToS3(buffer, fileKey)`
   - Get signed URL: `await getSignedUrl1(fileKey)`
   - Store both `signedUrl` and `uploaded_at: new Date()` in the DB

5. **Backend GET endpoint** — if this file will be returned in a GET response, call `refreshS3UrlPath()` before responding.

6. **Frontend service function** — add to the appropriate `src/api/<domain>Services.ts`:
   ```ts
   export const uploadFile = async (entityId: number, formData: FormData) =>
       axiosPostFormData<UploadResponse>(`/<path>/${entityId}/upload`, formData);
   ```
   Never set `Content-Type` manually.

7. Validate:
// turbo
```bash
cd subqdocs-backend && npx tsc --noEmit
```

8. Report the upload path, S3 key prefix, and files modified.
