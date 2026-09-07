# Tenant signing — one-off setup

Signing links travel through the **drop-box** Supabase project (the same one
used for tenant intake), never the main database. Two steps:

## 1. Upload to your web directory
- `sign.html` (the tenant-facing signing page — handles condition reports AND leases)
- re-upload `index.html` (carries the report + lease features)
- `config.js` (the same one the intake form uses — download from Settings → tenant intake). With it deployed and the "short links" toggle on, signing links shrink to `https://<your-site>/sign.html#sg-…` instead of carrying the drop-box address.

## 2. Run once in the DROP-BOX project's SQL editor
```sql
create table sign_requests (
  token text primary key,
  payload jsonb,
  signed boolean default false,
  signer_name text,
  signature text,
  signed_at timestamptz,
  created_at timestamptz default now(),
  expires_at timestamptz
);
alter table sign_requests enable row level security;
create policy "anon read" on sign_requests for select using (true);
create policy "anon sign" on sign_requests for update using (true) with check (true);
```

Trust model: same as intake — tokens are long and random, links expire after
30 days, and the row is deleted as soon as the signature is pulled into the
report. Photos travel as small embedded thumbnails; originals never leave the
main project's private storage.
