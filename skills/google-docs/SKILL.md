---
name: google-docs
description: Create, import, inspect, edit, or share native Google Docs through the locally installed `gws` Google Workspace CLI. Use for Google Docs and document-focused Google Drive requests; do not use for unrelated Drive file management.
---

# Google Docs through gws

Use the installed `gws` CLI for Google Docs and the document files that contain them in Drive.

## Preflight

- Confirm `gws` is available with `command -v gws` and `gws --version`.
- Check access with a bounded read such as `gws drive files list --params '{"pageSize":1,"fields":"files(id,name,mimeType),nextPageToken"}'`.
- If authentication is missing or expired, follow the current host authentication policy and run `gws auth login`; never print tokens or credential files.
- Use `gws schema <service.resource.method>` before a less familiar call instead of guessing request fields.

`gws` may print a keyring notice before JSON. When parsing output, begin at the first JSON object or array and keep stderr separate where possible.

## Create a native Google Doc

For authored documents where layout matters, use the available document-creation workflow first: produce a local DOCX using the Google Docs-native preset, sanitize its title block, render it, and inspect every rendered page. Then import that verified DOCX through Drive:

```bash
gws drive files create \
  --json '{"name":"Document title","mimeType":"application/vnd.google-apps.document"}' \
  --upload /absolute/path/to/verified.docx \
  --upload-content-type application/vnd.openxmlformats-officedocument.wordprocessingml.document \
  --params '{"fields":"id,name,mimeType,webViewLink,createdTime,modifiedTime"}'
```

Before creating, search for an existing non-trashed Google Doc with the exact intended name. Do not silently create a duplicate or replace an existing document; reuse, revise, or create another only when the user's intent establishes which outcome is wanted.

After creation, verify the returned ID with both Drive metadata and the Docs API:

```bash
gws drive files get --params '{"fileId":"DOCUMENT_ID","fields":"id,name,mimeType,webViewLink,createdTime,modifiedTime"}'
gws docs documents get --params '{"documentId":"DOCUMENT_ID","includeTabsContent":false}'
```

Return the canonical `https://docs.google.com/document/d/DOCUMENT_ID/edit` link when `webViewLink` is absent.

## Read or edit an existing document

- Resolve the exact document ID before reading or changing anything. Prefer an ID or URL the user supplied; otherwise use a narrow Drive query and disambiguate genuine collisions.
- Read the latest document structure with `gws docs documents get` before constructing index-based updates.
- Use `gws docs documents batchUpdate` for changes. When preserving concurrent edits matters, include the current `revisionId` in `writeControl.requiredRevisionId`.
- Verify the document again after a mutation. Do not claim that an edit landed from command success alone.

## Sharing and attribution

Drive permission roles map to Google Docs collaboration as follows:

- `writer`: edit directly.
- `commenter`: comment and propose suggestions without direct editing.
- `reader`: view only.

Adding or widening access is an external side effect. Do it only for recipients and roles the user explicitly names. Never create `anyone` or domain-wide access unless the user explicitly requests it.

```bash
gws drive permissions create \
  --params '{"fileId":"DOCUMENT_ID","sendNotificationEmail":true,"fields":"id,type,role,emailAddress"}' \
  --json '{"type":"user","role":"commenter","emailAddress":"person@example.com"}'
```

Verify sharing with `gws drive permissions list`. Google version history attributes changes to each collaborator's authenticated Google account; the plugin does not invent or rewrite attribution.

## Safety and handoff

- Treat deleting, replacing, moving, publishing, transferring ownership, or widening link access as separate actions requiring clear user intent.
- Keep the verified local source document as the controlled baseline when the workflow calls for one; the Google Doc is the collaboration surface.
- Report the document title, verified Google Docs link, and any permissions actually applied. State plainly when sharing was not configured because collaborator addresses or roles were not provided.
