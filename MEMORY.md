# Memory

## App Store Connect — delete an old / duplicate app record (2026-08)

**Why:** used to clear a Guideline 4.3(a) (Design: Spam) rejection caused by a near-identical duplicate test record on the same account (GeezEasy's `old to delete old`, Apple ID 6760924194).

**The 3 steps (in order — "Remove App" silently bounces if 1 & 2 are skipped):**
1. **Pricing and Availability → make the app unavailable in ALL territories.** Open the section → **Manage Availability** → choose **None** to clear every country. Even a never-released app starts at "175 Available".
2. **In-App Purchases → remove any associated IAPs from sale.** Check each app via the API first — some old apps carry zero IAPs.
3. **App Information → scroll to the bottom → Additional Information → Remove App → Remove.**

**Gotchas:**
- The **version / build can never be deleted** — API DELETE returns 409 with permanent "last version of an app cannot be deleted", "a version cannot be deleted if any build has been uploaded", and "a new version must be created on another platform" reasons. An *expired* TestFlight build still counts (expire ≠ delete). Stop trying — it's a dead end.
- App removal also wipes the old app's TestFlight builds (API returns 0 builds for that app id afterwards).
- New App Store Connect UI at a narrow window has **no visible sidebar** — the section links (Pricing and Availability, In-App Purchases, App Information) live in a dropdown that opens when you click the **section's heading button** at the top of the page.
- `/apps/{id}/distribution/ios/pricing` redirects; the real pricing breadcrumb is `/apps/{id}/distribution/pricing`.
- The API **cannot** clear availability for an app that never had it set: `GET/PATCH /v1/apps/{id}/appAvailability` → 404 "relationship does not exist". The territories step is UI-only.
