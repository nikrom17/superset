# Options brief: sign-in (P-01)

**Status:** Stage 1 research, 2026-11-22. For Nik's review. Feeds P-01 and flows D-11 / D-12.
**Question:** How does Nik get into Superset with Face ID first and a backup, with no sign-up, on an iPhone Home Screen app that must keep working offline?
**Tags:** **V** = Verified (source, checked 2026-11-22). **U** = Unverified (inference or no official source found). All "V" sources are listed at the end by number.

## Constraints from the PRD
- One user, one account made by hand in the backend (§4, D-11). Every screen and all data are behind sign-in (R-6.1.2).
- iPhone Air, installed to the Home Screen (standalone). Gym may have no signal (§6.7).
- Default stack: Next.js 16 on Vercel, Supabase Postgres + Supabase Auth. The meal-plan skeleton has **no user sign-in** to reuse: it is single-user, and only gates cron and admin routes with a bearer secret (`skeleton/lib/auth.ts`). So this is new ground on our stack.

## Key facts
| # | Fact | Tag |
|---|---|---|
| F1 | Supabase Auth has passkeys (WebAuthn). Beta since 2026-05-28; the JS API is "experimental, may change without notice" and needs `auth: { experimental: { passkey: true } }`, supabase-js 2.105.0+. | V [1][2] |
| F2 | A passkey can only be registered by a user who already exists, is confirmed, and is signed in. You can't sign up with a passkey alone. So the backup method is also how the first passkey gets made. | V [1] |
| F3 | Passkey sign-in uses discoverable credentials: no email or phone to type. | V [1] |
| F4 | The relying party ID is a bare domain. Changing it later breaks every passkey. Up to 5 origins. | V [1] |
| F5 | Since iOS 17.4, WebAuthn no longer needs a tap first. Apple replaced the gesture rule with rate limiting. So Face ID can be asked for when the sign-in screen opens. | V [3] |
| F6 | No official Apple or WebKit doc found that says WebAuthn works (or doesn't) in a standalone Home Screen web app. One forum report (Feb 2026, iOS 18.6) describes doubled or stalled passkey prompts from a Home Screen shortcut; no Apple reply. | U (gap) [4] |
| F7 | Passkeys sync through iCloud Keychain. A new iPhone signed into the same Apple Account with iCloud Keychain gets them. | V [5] |
| F8 | Supabase access tokens last 1 hour by default. Refresh tokens never expire but are single-use. Session time-box and inactivity timeout exist only on Pro. | V [6] |
| F9 | `@supabase/ssr` (the Next.js server pattern) stores the session in cookies. | V [7] |
| F10 | Safari caps cookies set by JavaScript (`document.cookie`) at 7 days. Cookies set by the server (HttpOnly) are not capped. | V [8] |
| F11 | Safari's 7-day storage wipe counts days of use with no interaction. A Home Screen app counts its own days, and using the app resets the timer. | V [9] |
| F12 | Home Screen apps can ask for persistent storage; WebKit grants it partly on "opened as a Home Screen web app". | V [10] |
| F13 | Supabase phone sign-in needs a third-party SMS provider (Twilio, Twilio Verify, MessageBird, Vonage, TextLocal). Default limit 30 SMS per hour. | V [11][12] |
| F14 | Supabase email can send a 6-digit code instead of a link (edit the template to `{{ .Token }}`). Codes expire after 1 hour by default. Built-in email is limited to 2 per hour. | V [13][12] |
| F15 | Supabase Free projects pause after 1 week of inactivity. Pro is from $25/month. | V [14] |

## Options

### A. Passkey (Face ID) via Supabase Auth's built-in support
- **Day to day:** Open the app. Face ID sheet appears (F5). Look at the phone. You're in. In practice this happens rarely, because the session persists (see "Staying signed in").
- **Setup:** Pick the final domain first (F4). Turn on Passkeys in the Supabase dashboard; set RP ID, origin, display name (F1). Opt in on the client (F1). Create the account by hand. Sign in once with the backup, then register the passkey (F2). ~half a day including UI. **U** (estimate).
- **Cost:** $0 extra. Passkeys pricing is not stated on Supabase's pages. **U**
- **How it fails:**
  - Standalone mode may be buggy (F6). This is the main unknown.
  - Beta API may change (F1). Low risk for one user, but an upgrade could break sign-in. **U**
  - Sign-in needs a network: the server checks the Face ID signature. **U** (by design of WebAuthn, not tested here).
  - New phone: passkey arrives by iCloud Keychain (F7). If iCloud Keychain is off, Nik needs the backup.
- **Fit:** Native to the stack. No extra library. A DIY library (e.g. SimpleWebAuthn) is the fallback if the beta disappoints, at the cost of minting Supabase sessions by hand. **U**

### B. Texted 6-digit code (Supabase phone auth + SMS provider)
- **Day to day:** Tap "Text me a code". iOS offers the code above the keyboard. Tap, done. **U** (the one-tap fill needs `autocomplete="one-time-code"`; not checked in standalone).
- **Setup:** Twilio account; in Supabase, enable Phone and paste Twilio Verify credentials (F13). Twilio Verify uses a managed number pool, so no number to buy [15] (V). Whether plain Twilio SMS also needs US A2P 10DLC or toll-free registration paperwork: **U** (Twilio's page mentions 10DLC fees but no amounts [16]).
- **Cost:** Twilio Verify: $0.05 per successful check + $0.0083 per SMS in the US [15] (V). At a few sign-ins a year, well under $1/year. **U** (estimate of usage).
- **How it fails:** Needs signal to receive the text, which the gym may not have. Code expires (F13). SIM swap risk is real but small for a gym log. **U** New phone: works as long as the number moves.
- **Fit:** Native to Supabase. One more vendor account and secret to keep.

### C. Email: magic link or 6-digit code
- **Magic link** (not recommended): the link opens in Safari, not in the Home Screen app, so the session lands in the wrong place. **U** (widely reported; Safari and Home Screen apps appear to keep separate storage; no official doc found). Testable in one POC.
- **Email code:** same flow as the link, but Nik types 6 digits into the app (F14). Stays inside the Home Screen app, so it avoids the problem above.
- **Setup:** Change one template in Supabase (F14). Optional custom SMTP if the 2/hour built-in limit bites (F14); for one user it likely won't. **U**
- **Cost:** $0.
- **How it fails:** Needs signal and access to mail. iOS may not offer the code above the keyboard as smoothly as SMS. **U** New phone: works anywhere Nik can read his mail.
- **Fit:** Native to Supabase, zero vendors.

### D. Recovery codes
- **Day to day:** Never. A printed or saved list of one-time codes for when Face ID and the backup both fail.
- **Setup:** Supabase Auth has no built-in recovery codes **U** (not found in docs), so this is custom code: a table of hashed codes and a server route that mints a session. More code to own than A to C.
- **Fit:** Poor value for one user whose account Nik can also reset in the Supabase dashboard himself. **U**

## Staying signed in and offline (applies to every option)
- Sign-in itself needs signal. After that, the app should **not** need a valid token to open or log. Local data is shown if a session exists on the phone. Tokens refresh when signal returns. Refresh tokens don't expire (F8), so a weeks-long gap is fine on the Supabase side.
- Risk 1: if the session lives in a cookie written by JavaScript, Safari caps it at 7 days (F10). Skip the gym 8 days and you'd be signed out. Fix: keep the session in local storage on the client (supabase-js), or set it only as a server HttpOnly cookie. **U** which `@supabase/ssr` does in the browser.
- Risk 2: iOS clearing storage. Using the app resets the 7-day timer (F11), and asking for persistent storage helps (F12). A full wipe means a re-sign-in **and** loss of unsynced logs; the second part is P-09.
- Risk 3: the supabase-js client trying to refresh with no signal and signing itself out. **U** Must be tested.
- Free plan pauses after a quiet week (F15). Not a sign-in problem, but it would look like one. Pro or a keep-alive ping avoids it.

## Comparison
| | A. Passkey | B. Text code | C. Email code | D. Recovery codes |
|---|---|---|---|---|
| Daily effort | Look at phone | Tap, 1-tap fill | Switch to mail, type | n/a |
| Setup | Low, beta API | Medium, Twilio | Very low | Custom code |
| Cost | $0 | ~$0.06 per use | $0 | $0 |
| Works in standalone | **Unknown (F6)** | Likely **U** | Likely **U** | Likely **U** |
| New phone | iCloud Keychain | Same number | Mail access | Have the list |
| Can bootstrap the passkey (F2) | No | Yes | Yes | No |

## Recommendation
**Passkey (A) via Supabase's built-in support, with an email code (C) as the backup and bootstrap.** Skip recovery codes; the Supabase dashboard is the last resort.
- The email code meets the design's "backup" job for $0 and no extra vendor. The texted code (B) works too, at well under $1/year plus a Twilio account. **This is D-12 for Nik:** if a texted code matters to you, B swaps in with the same Supabase pattern.
- Keep the session out of JavaScript cookies (Risk 1), never gate local data on a live token, and pin the domain before the first passkey (F4).
- If the POC shows passkeys are flaky in standalone mode, fall back to the code alone with a long-lived session. Day to day, Nik would rarely see either.

## What needs proving (stage 2 POCs, on Nik's iPhone, Home Screen install)
1. **Passkey in standalone:** register and sign in with Supabase passkeys from the Home Screen app. Face ID appears on screen open, no double prompt (F6), session lands in the app. Repeat 10 times.
2. **Bootstrap:** fresh install, sign in with the email code (or text code), register the passkey, sign out, sign back in with Face ID only.
3. **Magic link vs code:** confirm a magic link opens Safari and doesn't sign in the Home Screen app; confirm the code flow does.
4. **Session survives 8+ days unopened:** sign in, don't open for 8 days, open with no signal. Still signed in, data visible (Risk 1, F10/F11).
5. **Offline mid-session:** airplane mode for 2+ hours (past the 1-hour token), keep logging, reconnect. No sign-out, sync resumes (Risk 3).
6. **App kill and phone restart:** still signed in after both.
7. **New phone / reinstall:** delete and reinstall the Home Screen app (stand-in for a new phone). The passkey from iCloud Keychain still works (F7).
8. **If B is chosen:** Twilio Verify text arrives, iOS offers the code above the keyboard in standalone, and whether any US registration is needed.

## Sources (all checked 2026-11-22)
1. Supabase, Passkey authentication: https://supabase.com/docs/guides/auth/passkeys
2. Supabase changelog, Passkeys beta: https://supabase.com/changelog/46458-passkeys-for-supabase-auth-beta
3. Apple Developer Forums, WebAuthn gesture rule removed in iOS 17.4 (Apple engineer reply): https://developer.apple.com/forums/thread/747036
4. Apple Developer Forums, passkey issues from Home Screen shortcuts (user report, unanswered): https://developer.apple.com/forums/thread/815784
5. Apple, passkeys and iCloud Keychain security: https://support.apple.com/en-us/102195
6. Supabase, User sessions: https://supabase.com/docs/guides/auth/sessions
7. Supabase, Creating a server-side client: https://supabase.com/docs/guides/auth/server-side/creating-a-client
8. WebKit, ITP 2.1 (7-day cap on script cookies): https://webkit.org/blog/8613/intelligent-tracking-prevention-2-1/
9. WebKit, Full third-party cookie blocking and more (Home Screen app days of use): https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/
10. WebKit, Updates to storage policy: https://webkit.org/blog/14403/updates-to-storage-policy/
11. Supabase, Phone login: https://supabase.com/docs/guides/auth/phone-login
12. Supabase, Rate limits: https://supabase.com/docs/guides/auth/rate-limits
13. Supabase, Passwordless email: https://supabase.com/docs/guides/auth/auth-email-passwordless
14. Supabase, Pricing: https://supabase.com/pricing
15. Twilio, Verify pricing: https://www.twilio.com/en-us/verify/pricing
16. Twilio, US SMS pricing: https://www.twilio.com/en-us/sms/pricing/us
