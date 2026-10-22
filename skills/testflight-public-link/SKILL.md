---
name: testflight-public-link
description: The TestFlight standard — one public link and QR per app, never a manual tester list, plus the exact App Store Connect recipe in order and the join-link shape.
---

## 🔒 STANDARD — TestFlight testers = PUBLIC LINK + QR (never manual adds)

Locked 2026-09-16. Never add testers one by one, never keep an email list. Every app gets a TestFlight
PUBLIC LINK on its external beta group — testers scan the QR / tap the link → install TestFlight → Install → done (10,000 cap).
API recipe, in order: attach the newest VALID build to the external group → ensure betaAppReviewDetail +
betaAppLocalizations en-US + build `usesNonExemptEncryption` answered → submit beta review if none exists →
`PATCH /v1/betaGroups/{id} {publicLinkEnabled:true,publicLinkLimitEnabled:false}` → share
`https://testflight.apple.com/join/XXXXXXXX`. Script: `facelink/scripts/asc-public-link.js`
(key AuthKey_DASF43QDRT.p8, KEY_ID DASF43QDRT, team 0418454e-4e54-4fc7-a387-b3b2460fd5e8).
Testers can install only after Beta App Review approves the build.
