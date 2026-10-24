# NeextPay store payments and SMS sender

## Result
Upgrade the existing NeextPay website into a store-controlled payment gateway, with consistent branding and a separate downloadable Android SMS sender.

## 1. Store settings and API access
- Treat each existing merchant as a store, retaining its collection accounts.
- Add admin settings for store website origin, success redirect URL, callback URL and callback API key. Keep server notification and customer redirect separate.
- Issue store-bound API keys for store servers and separately scoped phone-ingest keys. Existing unassigned keys must be explicitly assigned or replaced; they will not receive unrestricted store access.
- Require the store API key and matching `storeId` for every store operation. Never allow one store to read or change another store's transactions.

## 2. Payment lifecycle
- Payment creation accepts amount, store customer `userId`, `storeTrx`, and an allowed success return URL, then returns a checkout link and automatically generated `gatewayTrx`.
- Keep `gatewayTrx` distinct from the wallet's verified transaction ID. Save the customer's submitted ID separately as `userVerifyTrx`, including unsuccessful verification attempts.
- Use the existing SMS matching rules to verify amount, receiving account, provider and unused wallet transaction ID. Prevent duplicate completion and wallet-ID reuse.
- Notify the store on successful verification using its callback API key and a signed payload; record delivery results and provide a controlled retry action.
- Show success and redirect only after the payment is confirmed; store servers must trust the authenticated callback or transaction API, not the browser redirect alone.

## 3. Store transaction APIs
- Support create, paginated list, details by gateway transaction ID, cancellation and deletion.
- Make repeated creation with the same store transaction reference safe rather than creating duplicate charges.
- Cancel only pending payments. Deletion hides records from normal store lists without destroying financial audit history; completed payments cannot be cancelled or erased.
- Provide an integration reference and sample store requests explaining authentication, fields, responses, status values and callback validation.

## 4. One-hour checkout
- Save the deadline as exactly one hour after link creation; reopening or refreshing never resets it.
- Display the countdown on wallet selection and transaction submission screens using the saved server deadline.
- At the deadline, cancel all unpaid payments, including submitted IDs that remain unverified, as selected.
- Enforce expiration during every completion path, including incoming SMS and admin actions, not just in the countdown.
- Persist cancellation with a supported time-based mechanism and show the expired state consistently in checkout, admin and store APIs.

## 5. Logos and Android sender
- Create two distinct NeextPay logo concepts, inspect their spelling and small-size readability, select the stronger fit and use it consistently on the website and app. Keep the existing official wallet logos.
- Build a separate Android SMS sender with server address, phone-ingest key, receiving phone number, permission consent, connection check and delivery status.
- Forward only supported payment SMS, keep an offline queue, retry safely and avoid embedding credentials in the APK. Include a manual recent-message send action.
- Attempt installation of the missing Android build tools and produce an installable signed APK with a download link. Retain source and document signing continuity for future updates.
- Android hardware SMS receipt and manufacturer-specific background behavior require a real phone. Report emulator/server tests separately from physical-device checks; do not claim guaranteed instant delivery or Google Play approval.

## 6. Verification and release readiness
- Test admin store settings, key creation/revocation, payment creation and transaction controls in the live browser.
- Test all four wallet formats, wrong IDs, wrong amounts/accounts, duplicates, cross-store access denial, callback credentials, success redirect and exact deadline boundaries.
- Test offline retries and server acceptance for the Android sender wherever the available tools permit; remove only temporary QA records afterward.
- Check phone and desktop layouts, logo rendering, page titles and error states; fix issues found in these requested flows.
- Deliver the APK only if it builds successfully and distinguish tested functionality from remaining device checks.
- Do not publish the website without your permission. Android-to-hosted-server testing needs a publicly reachable NeextPay address; local server tests can run before publication.

## Technical approach
- Keep TanStack Start and the current Lovable Cloud integration; add schema fields and protected server functions rather than replacing the app.
- Reuse merchants as stores. Add scoped key metadata, transaction references, saved expiry and soft-deletion fields through additive migrations.
- Centralize atomic completion/cancellation rules in database functions so verification, SMS ingest and admin updates cannot disagree or race.
- Restrict callback and redirect destinations to approved HTTPS store origins and block local/private destinations. Keep callback keys out of routine list responses and browser logs.
- Maintain semantic shared styling and shared client-safe branding/metadata. Add concrete rule tests alongside the existing tests and run them.

## Deliverables
Updated website and store APIs, NeextPay logo assets, store integration reference, test results, and a downloadable Android APK if the toolchain build succeeds.