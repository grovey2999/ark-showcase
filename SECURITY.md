# Security

This page describes how ARK protects children's data, and where the protection stops. Every control below was checked against the private source on 2026-11-24 by reading the file named in the Where column. The source is private, so the paths are given for traceability, not as links. The test suite was not run for this page. No pass counts are claimed.

## What is protected and how

| Control | What it does | Where |
|---|---|---|
| Per-record encryption of medical notes | Medical and allergy fields are encrypted with AES-256-GCM using the Node standard library. Each district has its own data key. That key is itself encrypted under a master key before it is stored. The ciphertext is bound to the district and record it belongs to, so a copy moved into another row or district fails to decrypt. | `src/crypto/envelope.ts`, `src/crypto/keyRegistry.ts`, `src/dal/medical.ts`, `tests/envelopeEncryption.test.ts` |
| District isolation | Each district is a separate database file. The data access layer requires a district context for every call. Tests attack the boundary. | `docs/adr/0001-one-sqlite-file-per-tenant.md`, `src/dal/context.ts`, `tests/tenantIsolation.test.ts`, `tests/ministryIsolationAttack.test.ts` |
| Password hashing | Passwords use scrypt with a per-user random salt and a versioned hash string, compared in constant time. An unknown email and a wrong password return the same response and cost comparable time. | `src/auth/password.ts`, `THREATS.md` |
| Sessions | The browser holds a random token. Only its SHA-256 hash is stored, so a copy of the database alone yields no working session. | `src/http/session.ts` |
| Emailed codes and links | A six-digit emailed code is a second factor, and a signed-in device can be remembered for a period. Sign-in links are single use. | `src/auth/mfaEmailCode.ts`, `src/auth/magicLink.ts` |
| Authenticator codes | Time-based one-time codes implemented against the published RFC test vectors, using only the standard library. | `src/auth/totp.ts`, `tests/mfa.test.ts` |
| Required second factor | A defined set of staff roles must use a second factor when a district has required it. | `src/http/routes/authRoutes.ts` (`MFA_REQUIRED_ROLES`) |
| Step-up re-authentication | Sensitive actions, such as resetting someone's sign-in, exporting a roster, refunding a payment or changing second-factor settings, ask for the password again even inside a valid session. | `src/auth/stepUp.ts`, `src/http/stepUpGuard.ts` |
| Rate limits | Failed sign-ins are counted and lock out by account and source pair and by source alone. A separate in-memory limiter throttles raw request volume before any other work. | `src/auth/rateLimit.ts`, `src/http/generalRateLimit.ts` |
| CSRF protection | State-changing routes require a token checked against the session. A test checks that every form route is covered. | `src/http/server.ts`, `tests/formCsrfRouteCoverage.test.ts` |
| Content-Security-Policy | Scripts and styles may load only from the application's own origin. Pages may not be framed. | `src/pages/shared/layout.ts` |
| Role-based access | One table maps role, resource and action. Every route calls the same permission function. A snapshot test and a generated matrix document fail or change when the table changes. | `src/policy/rbac.ts`, `tests/rbac.test.ts`, `tests/rbacMatrix.test.ts`, `docs/rbac_matrix.md` |
| Hash-chained audit log | Each audit entry stores a hash of the previous entry, so editing or deleting history breaks every later hash. A verify function reports where the chain first breaks. | `src/audit/auditLog.ts`, `tests/auditLog.test.ts` |
| Access notices | Opening a child's medical record can notify the family. | `src/notify/medicalAccessNotice.ts`, `tests/medicalAccessNotify.test.ts` |
| Background checks hold status only | The record stores status, expiry, who verified and when. No document, scan or file is stored. Only district-level roles can write a status. | `src/dal/backgroundChecks.ts` |
| Pickup codes and approved adults | Checking a child out can require a code, and the family keeps a list of approved pickup adults. | `src/dal/pickupCodes.ts`, `src/dal/checkins.ts`, `src/dal/authorizedPickupAdults.ts` |
| Secret scan | A script searches the repository for key-shaped strings and fails on a match. | `scripts/check-secrets.mjs` |
| Threat model | A written threat model lists each asset, threat, current control and known gap. It is kept in the repository and updated as controls change. | `THREATS.md` |
| Backups and restore drill | Configuration for continuous replication to object storage, a script that copies backups off the machine, and a drill that opens each backup read-only, runs an integrity check and compares row counts. | `litestream.yml`, `scripts/backup-offbox.ps1`, `scripts/restore-drill.ps1`, `docs/adr/0006-backups-proven-by-restore.md` |
| Fewer dependencies | No runtime npm packages, so no package supply chain in the running server. | `package.json`, `docs/adr/0002-zero-npm-dependencies-in-core.md` |

## What is not done, or not proven

- **No outside penetration test is claimed.** No report or engagement record was found in the repository. All testing described here is the project's own.
- **No certification is claimed.** There is no SOC 2, ISO 27001 or similar attestation.
- **The offline check-in bundle is a design stub.** The source says so in its header. The design stores only an allergy flag on the device, encrypts the bundle and expires it after 72 hours, but it is not shipped and should not be treated as working.
- **Master key source.** By default the master key comes from an environment variable. A provider for AWS Key Management Service exists in the source. This page does not claim which one any deployment uses.
- **Scrypt, not argon2id.** The zero-dependency rule rules out an argon2 library. Scrypt is the standard-library choice and the threat model records the tradeoff.
- **The audit log detects tampering and does not prevent it.** Someone with raw access to the database file can still edit rows. The chain makes that visible.
- **Rate limiting is in process.** The request-volume limiter lives in memory in one process. There is no CAPTCHA.
- **The secret scan is simple pattern matching.** It is not a replacement for a dedicated scanner.
- **Backup schedule is not claimed.** The scripts and configuration exist. This page does not claim how often they run in production or when the last drill ran.
- **Test results are not claimed.** There are 777 test files. They were not run for this page.
- **Mixed role defaults.** The second factor is required for a defined set of staff roles when a district turns it on. Parents sign in with a password or emailed link without it.

## Reporting a problem

Contact the maintainer through the owner of this GitHub account.
