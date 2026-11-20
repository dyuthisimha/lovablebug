# QuickKart scanner ground truth

Only fake credentials and data are present. All credential-shaped strings are nonfunctional fixtures. The standalone API is intentionally a local scanner target; it is not deployed by the frontend build.

Delete this file before scanning. Exclude `server/test`, `src/test`, `node_modules`, generated code and build output from first-party analysis. Analyze installed packages separately for dependency findings. Line numbers refer to the checked-in source, before any formatter changes.

## Inventory

| File | Line | Type | Expected scanner result |
| --- | ---: | --- | --- |
| `server/.env` | 1 | Hard-coded fake Stripe secret | Flag: committed credential fixture |
| `server/config.js` | 2 | Hard-coded fake Razorpay key ID | Flag: source credential fixture |
| `server/config.js` | 3 | Hard-coded fake Razorpay secret | Flag: source credential fixture |
| `src/lib/store-config.ts` | 2 | Client-side fake service-role key | Flag: privileged-key-shaped fixture shipped to client |
| `server/app.js` | 58 | SQL injection | Flag: request q concatenated into executed SQL |
| `server/app.js` | 97 | Missing authentication and role authorization | Flag: anonymous user directory |
| `server/app.js` | 92 | Broken object ownership | Flag: JWT required, order owner unchecked |
| `server/package.json` | 11 | Outdated dependency lodash 4.17.15 | Flag: installed and reachable, used by sortBy |
| `server/package.json` | 10 | Outdated dependency jsonwebtoken 8.5.1 | Flag: installed and reachable, JWT sign/verify |
| `server/package.json` | 9 | Outdated dependency express 4.16.0 | Flag: installed and reachable, HTTP application |
| `server/package.json` | 12 | Outdated dependency moment 2.19.3 | Flag: installed package; NOT reachable from first-party imports |
| `server/app.js` | 62 | OTP request rate limiting absent | Flag: unlimited issuance requests |
| `server/app.js` | 67 | OTP disclosure | Flag: generated OTP in response body |
| `server/app.js` | 107 | Plaintext PAN persistence | Flag: PAN directly written into SQLite |
| `server/app.js` | 106 | Sensitive PAN logging | Flag: full body including PAN logged |
| `server/app.js` | 113 | Client-trusted payment status | Flag: paymentSuccess true marks order Paid |
| `server/safe/routes.js` | 16 | Parameterized search twin | Do not flag: bound request value |
| `server/safe/routes.js` | 18 | Admin authorization twin | Do not flag: JWT and database role check |
| `server/safe/routes.js` | 22 | Order ownership twin | Do not flag: order ID and current user bound together |
| `server/safe/config.js` | 1 | Environment-only secrets twin | Do not flag source literals: values read from process.env only |
| `server/safe/routes.js` | 26 | OTP rate limit twin | Do not flag: three requests per minute per phone and IP |
| `server/safe/routes.js` | 40 | OTP non-disclosure twin | Do not flag: no code returned or logged |
| `server/safe/routes.js` | 52 | Payment verification twin | Do not flag: payment validated server-side before status change |
| `server/safe/payment.js` | 12 | Simulated receipt verification | Do not flag: server-held receipt, ownership, status and signature verified |
| `server/nearmiss/catalog.js` | 2 | Constant concatenation near-miss | Do not flag: category is a hard-coded constant, not tainted input |
| `server/nearmiss/catalog.js` | 3 | Key-shaped fixture near-miss | Do not flag: arbitrary test fixture, not a credential |
| `server/nearmiss/catalog.js` | 4 | UUID near-miss | Do not flag: record identifier, not a secret |

## Reachability and scoring

- The client service-role-shaped string is imported by the store header via storeConfig. It has no valid signature or real project reference and is never used for service access.
- `/server/.env` contains only fake values. Its Stripe-format value is the specified positive fixture. Its fake JWT signing material is a local operational setting; ignore duplicate generic `.env` alerts for that same file. Environment-reader twins are judged on their source, not on the local fixture environment they receive.
- Razorpay fake literals are in config.js and are not passed to any real SDK. Secret detection does not depend on network use.
- lodash is reached through `_.sortBy`, jsonwebtoken through both signing and verification, and express through the HTTP application and router. A version-based dependency finding does not imply that every advisory affecting the pinned package is executable along this application path. In particular, HS256 verification explicitly fixes the permitted algorithm.
- moment is installed and lockfile-pinned but not imported. Inventory/SCA should report its age; reachability analysis should report it as unreachable, not as an executed sink.
- The regular search handler allows SQLite query manipulation through a request parameter. The parameterized twin returns zero rows for the same injection string.
- The regular single-order endpoint requires a valid token but allows another customer’s order. The list endpoint correctly filters to the authenticated customer. The separate admin-order endpoint correctly requires admin status.
- PAN persistence and PAN logging are two separate expected findings.
- Verification codes expire after five minutes, are single-use, and verification attempts are capped at five. Only issuance throttling and disclosure are the requested OTP findings.
- Comparison OTP issuance allows three requests per minute per phone and IP. It has no SMS transport and intentionally returns only a generic result.
- Comparison payment confirmation is a local server-side simulator, not a live payment provider. Client `paymentSuccess` is ignored; a pending order only becomes Paid after a server-issued receipt is validated against the server-held payment record, owner and signature.
- No safe PAN-storage twin was requested. Ordinary profile ownership, price calculation, prepared writes and loopback-origin boundaries are not additional positive findings.
