# Universal Links — `apple-app-site-association`

This file enables iOS Universal Links for `https://jineo.app/invite/CODE`
to open the Jineo app instead of Safari (or a 404).

## Critical deploy requirements

GitHub Pages serves this file fine, but Apple has strict requirements:

1. **Path must be exact:** `https://jineo.app/.well-known/apple-app-site-association`
2. **No file extension** — note the file has no `.json` extension. Don't add one.
3. **Content-Type must be `application/json`** — GitHub Pages does this automatically for files in `.well-known/` since 2019. Verify with: `curl -I https://jineo.app/.well-known/apple-app-site-association`
4. **HTTPS only** — `http://` requests are rejected by iOS.
5. **Must NOT redirect** — the URL must return 200 directly. A redirect to a CDN breaks Universal Links silently.
6. **No JWT signing required** since iOS 9.

## How to verify after deploy

```bash
# Should return 200 with content-type: application/json
curl -I https://jineo.app/.well-known/apple-app-site-association

# Should return the JSON content
curl https://jineo.app/.well-known/apple-app-site-association

# Apple's CDN cache check — if Apple has cached an old (or missing) version,
# you can force a refresh by submitting the app to Apple's verification:
# https://search.developer.apple.com/appsearch-validation-tool/
```

## Wired in the app at

- `app.config.js` → `ios.associatedDomains: ["applinks:jineo.app"]`
- `lib/deepLink.ts` → `parseInviteUrl()` matches `https://jineo.app/invite/CODE`

## After deploy, test

On a real iOS device (Universal Links don't work in Simulator with random URLs):

1. Send yourself an invite link: `https://jineo.app/invite/TESTCODE` via Messages
2. Long-press the link → should show "Open in Jineo" option
3. Tap → app opens, and `incomingReferralCode` should be set to `TESTCODE`
4. Welcome screen shows the gold pill: "🎉 Invited by TESTCODE"

If long-press shows only "Open in Safari," Apple hasn't picked up the AASA yet — wait 24h or force a re-fetch by reinstalling the app.

## Adding more paths later

If you ship `/i/CODE` shortlinks (TikTok-bio-friendlier than `/invite/CODE`), they're already in the paths array — just need to add the route to `lib/deepLink.ts:parseInviteUrl()`.
