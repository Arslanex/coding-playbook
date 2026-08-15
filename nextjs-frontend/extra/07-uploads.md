# Extra 07 · Uploads (UI)

WHEN: the user picks a file and this app sends it (or a signed URL).
LOAD: this file **and** 08, 09, 15. Backend: [15-security.md](../../python-fastapi-backend/15-security.md) (size cap, MIME from magic bytes, signed URL, no token in the query) and [08-infra.md](../../python-fastapi-backend/08-infra.md) (`infra/storage/`) when the contract is in doubt.
SCOPE: the picker and progress. Bytes do not belong in Postgres; the API says where.

MUST: auth on the page; size client-check matching `config` max (04) **and** the API still rejects. MUST NOT [critical]: trust only the client.

Prefer: API returns a signed PUT URL → client uploads to storage → API confirm. MUST NOT: base64 in JSON as the default.

Progress on the client leaf. Errors: `error_code` + `message`. MUST NOT [critical]: a token in the upload query the user can copy into a chat (15).

`accept` + real type from the API after magic-byte check — the UI `accept` is a hint.

---

## Done

- [ ] Size/type hinted on the client; enforced on the API
- [ ] Signed URL or API multipart; no token in a shareable query
