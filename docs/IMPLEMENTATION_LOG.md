# Keystore implementation log

Updated: 2026-11-26 (Africa/Lagos)

## Goals

- Keep the encrypted local clipboard archive usable offline, and offer opt-in cloud synchronization through Supabase.
- Make saved text available on Android and a signed-in web archive.
- Preserve edited formatting where formats support it and export PDF, DOCX, and TXT.
- Improve keyboard hit accuracy and keep changes traceable.

## Decisions

- Room remains the local archive. Clipboard content is encrypted with Android Keystore before it is saved.
- Sync requires Google sign-in, explicit user opt-in, and a network connection. WorkManager retries transient errors.
- Cloud content is sent over HTTPS and protected with per-user Supabase RLS and provider encryption at rest. It is not end-to-end encrypted.
- Password-like and otherwise sensitive entries are excluded from cloud sync.
- Local records remember their cloud account owner to avoid sending pending edits between accounts on a shared device.
- The static web archive uses the Supabase publishable key and user JWT; it never uses a service-role key.

## Progress — 2026-11-26

- Created Supabase project `Keystore Cloud` in `eu-west-1` (ref `ggrohhhnmcakcrmojrjs`) after approval of the $0/month quote.
- Added `clipboard_entries` with owner-scoped RLS, rich text metadata, sync IDs, and authenticated CRUD grants only. Security advisor reports no lints.
- Added Android Google Credential Manager token exchange, encrypted Supabase session storage, opt-in sync, connected-network background jobs, account ownership, last-write-wins conflict checks, and deletion tombstones.
- Added web archive sign-in, search, edit, favorite, trash/restore, copy, rich HTML copy when available, TXT export, and print-to-PDF.
- Added Android PDF, DOCX, and TXT exports from the existing rich text editor.
- Improved keyboard hit testing so the small gaps between character keys resolve to the nearest character key.
- Updated local privacy disclosures, README, website, and Google OAuth setup instructions.
- Published the Keystore home page, Google sign-in web archive entry point, and updated privacy policy to the existing public GitHub Pages site: `https://mr-willlss.github.io/key-store-privacy-policy/`.
- Raised the Android minimum from API 23 to API 24 because the supported Google Credential Manager identity library requires API 24.

## Remaining

1. Configure Google web and Android OAuth clients, package/SHA-1 fingerprints, Supabase Google provider secret, and allowed web redirects.
2. Finish a clean Android build, fix compile issues, and install the APK on a physical device.
3. Verify account isolation, login, opt-in sync, offline retry, edits/conflicts, deletes, and sensitive-entry exclusion.
4. Configure `https://mr-willlss.github.io/key-store-privacy-policy/web/` and its origin in Supabase Auth and Google OAuth redirect allow-lists.
5. Validate PDF/DOCX in common readers; web-side DOCX export still needs implementation.
6. Review keyboard comfort across device sizes and orientations.
7. Complete release privacy/data-safety details, including developer contact information.

## Verification so far

- Supabase project is healthy; migrations, schema, policies, and grants were inspected live.
- Live table check confirms `public.clipboard_entries` exists with RLS enabled (0 rows before the first signed-in sync).
- GitHub Pages responds 200 for the home page and `/web/` archive; account sign-in will work after Google OAuth is configured.
- No Android build, Google OAuth login, or physical-device validation has completed.
- The first SDK archive transfer failed its published SHA-256 check and showed a decompression error. It is not being used; a clean download is underway.

