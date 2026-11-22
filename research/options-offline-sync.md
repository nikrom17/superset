# Options brief: offline storage and sync (P-09)

**Date:** 2026-11-22 · **Feeds:** PRD v4 §6.7 (R-6.7.1–3), §7, §10, P-09, Q-3 · **Status:** research only, nothing proven on the iPhone yet.

Tags: **Verified** = checked in an official doc or WebKit post on 2026-11-22 (URL given). **Unverified** = my reasoning, a third-party source, or an estimate.

## 1. What has to work offline

| Data | Offline need | Rough size (Unverified estimate) |
|---|---|---|
| Workouts, supersets | Read (Today, Session) and edit (Setup) | ~5 workouts, <10 KB |
| Exercises (setup, rep range, equipment link) | Read and edit | ~50 exercises, <25 KB |
| Equipment and weight lists | Read and edit | ~20 items, <10 KB |
| Sessions, sets, ratings | Write (log), read (History, targets) | ~200 B per set; 3 sessions/week × ~25 sets ≈ 0.8 MB/year |
| In-progress session | Write on every tap; must survive an app kill | <5 KB |
| Today's chosen workout | Read and write; phone only | bytes |
| App shell (JS, CSS, HTML, icons) | Must load with no signal on a cold start | ~1–3 MB |

Total: well under 10 MB after several years. Size is not the problem. Durability is. (Unverified estimate.)

Targets are derived (§7), so they are recomputed on the phone from the local copy. No server call is needed to show Today. (Unverified: depends on keeping the progression logic in shared client code.)

## 2. iOS facts every option inherits

| Fact | Tag |
|---|---|
| Safari has no Background Sync API (SyncManager), on iOS or macOS, through iOS 27.2. Sync can only run while the app is open. | **Verified** https://caniuse.com/background-sync (2026-11-22) |
| A Home Screen web app gets the same origin quota as in Safari: up to 60% of disk for browser apps (iOS 17+). | **Verified** https://webkit.org/blog/14403/updates-to-storage-policy/ (2026-11-22) |
| Default storage is "best-effort" and can be evicted under storage pressure or after a site goes unused "for some time", least-recently-used first. | **Verified** same URL |
| An origin is excluded from eviction if its storage is in persistent mode or it has an active page. `navigator.storage.persist()` asks for persistent mode; WebKit grants it by heuristics, for example "whether the website is opened as a Home Screen Web App". | **Verified** same URL |
| The 7-day cap on script-writable storage (IndexedDB, localStorage, SW cache) counts days of use. Home Screen apps have their own counter, and WebKit does "not expect the first-party in such a web application to have its website data deleted". | **Verified** https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/ (2026-11-22) |
| `estimate()` quota is not a guarantee; writes can still fail with quota errors. | **Verified** webkit.org/blog/14403 |
| Whether `persist()` actually returns true in a Home Screen app on iOS 26. | **Unverified** (POC 1) |
| An IndexedDB transaction either commits fully or not at all if the app is killed mid-write. Safari's flush-to-disk timing on kill is not documented. | **Unverified** (POC 2) |
| A service worker can serve a cached app shell on a cold offline launch from the Home Screen. | **Unverified** on Nik's phone (POC 3) |
| Next.js recommends Serwist for full offline caching, and has an experimental `useOffline` hook that retries failed navigations and Server Actions. | **Verified** https://nextjs.org/docs/app/guides/progressive-web-apps (2026-11-22) |

What this means for design: write every tap to IndexedDB at once, request `persist()` on first launch, sync on app open, on `visibilitychange` to visible, and on the `online` event. Never rely on the service worker to sync after the app closes. (Unverified: design inference.)

Coupling with P-01: the app must open offline without a fresh sign-in, so the auth session must be usable from local state. (Unverified: needs the P-01 choice.)

## 3. Options

### A. Dexie (IndexedDB) + outbox + Next.js API routes (hand-rolled)
- **How it works:** Dexie tables mirror the Prisma models. Every write goes to Dexie and appends an outbox row (`op`, `table`, `id`, `payload`, `clientUpdatedAt`) in one transaction. A sync loop POSTs the outbox in order to `/api/sync/push` (Prisma upserts, idempotent on a client UUID), then pulls `/api/sync/pull?since=cursor`. Serwist caches the app shell.
- **Setup:** Medium. ~3 tables of sync plumbing, one push and one pull route, `updatedAt` plus `deletedAt` columns. Free.
- **Cost:** $0 beyond Vercel and Supabase. Dexie core is open source. (Unverified: pricing for Dexie Cloud, not needed here, not checked.)
- **Conflicts (two devices):** Client UUIDs, so new sessions and sets never collide. Sets are append-only. Setup records use last write wins per record by `clientUpdatedAt`; deletes are tombstones. This matches the §10 fallback. (Unverified: design.)
- **iOS failure modes:** Same as §2. All under our control: the outbox survives a kill if the transaction committed; a failed push leaves rows in the outbox and shows "failed · Retry".
- **Fit:** Best. Prisma stays the only write path to Postgres. No new vendor.
- **Proven pieces (meal-plan repo, read only):**
  - Outbox with one retry, "failed · Retry" status, status listeners, sync on start and on `visibilitychange`, injectable deps: `/Users/nroman/Claude/Projects/Weekly Meal Plan Auto Scheduler/skeleton/lib/alerts/outbox.ts`, unit test `tests/unit/outbox.test.ts`, e2e `tests/e2e/alert-sync.spec.ts` (E-5, Retry flow in Playwright WebKit). Built and tested; single-record only, localStorage, not tested offline on iPhone.
  - Server ignores stale writes by `clientUpdatedAt` inside a locked transaction: `skeleton/lib/alerts/server.ts` (`replaceStageAlert`). Reusable LWW pattern.
  - Versioned, zod-checked, try/catch storage wrapper: `skeleton/lib/state/storage.ts`.
  - Service worker with `skipWaiting`/`clients.claim` and `Cache-Control: no-cache` on `/sw.js`: `skeleton/public/sw.js`, `skeleton/next.config.ts`. It caches nothing ("the app assumes a network connection"), so no offline caching is proven.
  - Installed iOS Home Screen PWA ran for 3 months on iOS 26 (push only): `/Users/nroman/Claude/Projects/Weekly Meal Plan Auto Scheduler/pwa-notifications-hld-input.md` §1. Proves install and SW registration, not storage durability.
  - POC-16, 18, 19 have nothing on offline storage or service worker caching.

### B. PowerSync (SQLite in the browser) + Supabase
- **How it works:** PowerSync streams chosen Postgres rows into a client SQLite database. Local writes queue in PowerSync's upload queue; your `uploadData` connector sends them to your backend (Supabase Data API or your own route). **Verified** https://docs.powersync.com/integrations/supabase/guide (2026-11-22)
- **Setup:** Medium–high. Logical replication on Supabase, sync rules, a hosted or self-hosted PowerSync service, a second schema (client SQLite) next to Prisma.
- **Cost:** Free tier: 2 GB synced/month, 500 MB hosted, but the instance is deactivated after about 1 week of inactivity (the page also says 2 weeks). Pro from $49/month. Self-hosted Open Edition is free. **Verified** https://www.powersync.com/pricing (2026-11-22). A gym gap longer than a week could pause the free instance. (Unverified impact.)
- **Conflicts:** Server is authority; the upload queue replays in order. LWW per row unless you write custom logic. (Unverified.)
- **iOS failure modes:** JS SDK defaults to an IndexedDB-backed SQLite VFS on Safari; docs note possible stability issues on Safari and known OPFS issues. **Unverified** (from a search summary of docs.powersync.com/resources/supported-platforms; not read directly). Same eviction rules as §2.
- **Fit:** Fair. Adds a vendor and a second data model. Strong if multi-device use grows.

### C. RxDB + Supabase replication plugin
- **How it works:** RxDB collections in IndexedDB; the plugin pulls via PostgREST with a `(modified, id)` checkpoint, pushes with optimistic guards, and uses Supabase Realtime for live updates. Needs `_modified` and `_deleted` columns and soft deletes. **Verified** https://rxdb.info/replication-supabase.html (2026-11-22)
- **Setup:** Medium. Tables must fit RxDB's shape and be exposed through the Data API with RLS. Bypasses Prisma for sync.
- **Cost:** Free tier uses the Dexie storage; the native IndexedDB and OPFS storages are in Pro at $99/month. **Verified** https://rxdb.info/premium/ (2026-11-22)
- **Conflicts:** Built-in conflict handler (default strategy not stated on that page). **Unverified.**
- **iOS failure modes:** Same as §2; Realtime socket drops when iOS suspends the app. (Unverified.)
- **Fit:** Weak. Two write paths into Postgres (Prisma and PostgREST), RLS to maintain.

### D. Ruled out
| Option | Why | Tag |
|---|---|---|
| Service worker + Background Sync | Not in Safari or iOS at all | **Verified** caniuse (above) |
| Zero (Rocicorp) | "Zero does not support offline writes"; writes are rejected when disconnected | **Verified** https://zero.rocicorp.dev/docs/offline (2026-11-22) |
| Replicache | Maintenance mode, open-sourced, no new features | **Verified** https://replicache.dev/ (2026-11-22) |
| ElectricSQL + TanStack DB | Electric joined Databricks on 2026-08-11 (open source "stays open source"); offline writes need TanStack DB 0.6 plus its offline-transactions package, which is young. Third-party claim that Electric Cloud shut down 2026-09-28. | **Verified** https://electric.ax/blog (2026-11-22) for the join; shutdown claim **Unverified** (rxdb.info only) |

## 4. Comparison

| | A. Dexie + outbox | B. PowerSync | C. RxDB |
|---|---|---|---|
| New vendor | None | PowerSync | None (library) |
| Monthly cost | $0 | $0 (pauses) or $49+ | $0 (Dexie storage) |
| Write path to Postgres | Prisma only | Your connector | PostgREST |
| Code we own | Most | Least | Middle |
| Reuses meal-plan pieces | Outbox, LWW, storage wrapper | Little | Little |
| Multi-device sync quality | Good enough for rare use (Q-3) | Best | Good |
| Risk | Our own bugs in sync | Safari stability, vendor pause | Schema shape, RLS |

## 5. Recommendation

**Option A: Dexie + an outbox + two Next.js sync routes, with Serwist caching the app shell.** One user, a few MB of data, and rare second-device use (Q-3) don't justify a sync vendor. Conflict rule: client UUIDs, append-only sets, last write wins per Setup record, tombstones for deletes. Sync on launch, on return to the app, on `online`, and after each logged round. Request `persist()` on first launch. Show "unsynced since X" if the outbox is older than a day (the §10 fallback). Revisit B only if Q-3 turns into regular two-device use. (Recommendation; Unverified until the POCs below pass.)

## 6. What needs proving (stage 2 POC candidates, on Nik's iPhone, Home Screen install)

1. **Persistent storage.** Does `navigator.storage.persist()` return true in the installed app on iOS 26? Log `persisted()` and `estimate()` on screen.
2. **Kill mid-write.** Airplane mode, log 3 rounds, swipe the app away during a round write, reopen. Every committed set and the in-progress session are there; no half-written round.
3. **Cold offline launch.** Airplane mode, force-quit, reboot the phone, open from the Home Screen. Serwist-cached shell loads Today, History, and Setup with no network.
4. **Reconnect sync.** Log a full session offline, reconnect. Outbox drains with no duplicates (idempotent push), status goes offline → syncing → synced. Then block the API and confirm "failed · Retry" and recovery.
5. **Long gap.** Leave unsynced data, don't open the app for 7+ days, reopen. Data still there. (Answers the §10 risk; slow, so start early.)
6. **SW update offline.** Deploy a new version while the phone is offline; confirm the old shell still loads and the update applies cleanly next online launch, with no lost outbox rows.
7. **Offline auth.** With the P-01 sign-in choice, the app opens offline days after the last online sign-in.
8. **Two devices (only if Q-3 says yes).** Edit the same exercise on phone and a laptop offline, sync both; the later edit wins and nothing else is lost.
