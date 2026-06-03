# Cordova Hot Code Push & Compat Versions

Concise guide for **manager-mobile** (Meteor + Cordova): how updates reach users, what `METEOR_CORDOVA_COMPAT_VERSION` does, and how to ship safely.

---

## Default workflow (no pinning)

**Most deploys should not set `METEOR_CORDOVA_COMPAT_VERSION`.** That is the normal, healthy path.

- Deploy server + web client on the usual schedule; HCP delivers JS to store apps whose native compat still matches the deployment.
- Keep **API and client changes backward compatible** so older local JS (and users who never update the store app) keep working.
- Ship a **new store build** when native changes are required (Cordova plugins, icons, platform bumps, etc.) — aligned with the repo, not as an afterthought.

**Pinning is optional** — a safety valve for **specific situations** (see below), not a requirement for every release. Use it when you deliberately need to decouple “what’s in the repo” from “what production native can run,” or when recovering from a bad deploy.

---

## Two layers: native vs web (JS)

| Layer | What it is | How users get updates |
|-------|------------|------------------------|
| **Native** | APK/IPA — Cordova shell, plugins, icons | **App Store / Play Store** only |
| **Web (JS bundle)** | Meteor client — UI, logic, most app code | **Hot Code Push (HCP)** from server, or embedded in the APK |

- **HCP** = mechanism that downloads a **new web bundle** from the server and runs it on the next launch (no store release required).
- **Embedded bundle** = JS shipped inside the APK at build time (fallback if HCP never runs or is blocked).

---

## Cordova compat version (the “allowlist key”)

**Not** the App Store version (`1.2.3`). **Not** “how old the bundle feels.”

**Compat version** = fingerprint of the **native Cordova setup** on the device (platforms + Cordova plugins as built into that APK). Meteor uses it to decide **who may receive** a hot-update bundle from a given server deploy.

- Same native build in the store → same compat id on every install.
- Bumping Cordova plugins or native config in the repo **without** a new store build → phones still send the **old** compat id.
- `METEOR_CORDOVA_COMPAT_VERSION` (optional, server, at **deploy** time) = override: “build and serve hot-update JS for **this** native generation,” instead of the default from the current repo build.

Think of it as an **allowlist**, not “make app and server versions equal”:

| Phone compat | Server deploy (default or pinned) | Hot update? |
|--------------|-----------------------------------|-------------|
| **Match** | Same compat line | **Allowed** — may download new JS from server |
| **No match** | Different compat line | **Blocked** — keeps JS already on device |

**Match does not guarantee the JS is safe** — it only means “this phone is eligible for hot update.” The server must still ship JS that runs on that native shell.

---

## What a deploy changes (and what it doesn’t)

Each `meteor deploy` publishes **one** current web client for the server. Meteor does **not** keep a library of every historical bundle on the server — history lives in **git + redeploys**.

| Action | Effect |
|--------|--------|
| **Deploy without pin** | Hot-update JS is built from **current** repo (Meteor version, Cordova plugin config, etc.) |
| **Deploy with `METEOR_CORDOVA_COMPAT_VERSION=…`** | That deploy’s hot-update JS is built **for that compat line** (e.g. production store native) |
| **Pin alone (no redeploy)** | Nothing changes until you **redeploy** with the variable set |

---

## What went wrong (white screen) — mental model

Typical failure mode we hit:

1. Store users still on an **old APK** (unchanged native → compat id still `OLD`).
2. A **large server deploy** (Meteor + Cordova plugin bumps in the repo) shipped **new web JS** that does not run on that native shell.
3. Compat id on the phone could still be **`OLD`**, so HCP **allowed** an update — but the **content** of the bundle was built for a newer project state → crash on startup → **white screen**.
4. **Fix (exceptional):** set `METEOR_CORDOVA_COMPAT_VERSION` to the **production** compat (from store APK / manifest), **redeploy** → server again serves JS appropriate for `OLD` native. After recovery, return to **unpinned** deploys once store and repo are aligned again.

**White screen** = JS that actually ran failed before UI rendered.  
**Not** “compat mismatch causes white screen” — mismatch usually **blocks** download; the incident was **match + incompatible JS**.

---

## Practical release guide

### Routine server deploys (default — no pin)

- Deploy **without** `METEOR_CORDOVA_COMPAT_VERSION` unless you hit a case in “When to pin” below.
- Prefer **backward-compatible** server/API and client changes so old local JS still works.
- **Server first**, store app later is fine when old bundles still run on the new server (common for weekly-style updates).

### Native or Cordova plugin changes (still default: coordinate, don’t pin by default)

**Preferred approach:**

1. Change Cordova plugins / native config in the repo when you are ready for a **store release**.
2. Build and ship the **new APK/IPA**.
3. Deploy server **without pin** (normal deploy) once store build matches what you intend production to run.

Pinning is **not** a substitute for releasing aligned native builds — it is a **bridge** when timelines don’t line up.

### When to pin `METEOR_CORDOVA_COMPAT_VERSION` (special cases only)

Use a pin only when an **unpinned** deploy would hot-push JS that production native cannot run, or you need an explicit recovery.

| Situation | Pin? |
|-----------|------|
| Regular feature/fix deploy, no Meteor/Cordova stack change | **No** |
| Server/backend only, mobile client unchanged | **No** |
| Repo bumped Meteor or Cordova plugins, **store not updated yet** | **Yes** — pin to **current store** compat until new APK is live |
| Recovering from white screen / bad HCP deploy | **Yes** — pin to production compat, redeploy; then **remove pin** when aligned |
| New APK live, repo and store in sync | **No** — unpinned deploy |
| Temporary bridge: new APK must get HCP before server moves to new compat line | **Yes** — rare; pin **same** compat on server **and** Cordova build until you bump both together |

**How to get the pin value:** production APK (`manifest` / Cordova metadata) or a build from the branch that matches the store app.

Always: set env at **deploy** time and **redeploy** — the variable does nothing without a new deployment.

### Shipping a new APK (store release)

**Normal path:** release APK → deploy server **without pin**. HCP works for users on the new native line; keep server API compatible for users still on old store builds.

**If you use a pin** (only when needed):

| Scenario | Server pin | Result |
|----------|------------|--------|
| New APK live, **no pin** (default) | — | New installs get HCP when compat matches deploy; manage old users via API compatibility |
| Deploy **before** store, risky stack change | `OLD` (store compat) | Protects production until new APK ships |
| New APK + pin to **`NEW`** only | `NEW` | New installs get HCP; **`OLD`** installs stop receiving HCP |
| New APK + pin still **`OLD`** (bridge) | `OLD` | Rare — both lines get HCP until you unpinned / bump together |

### Order of operations

- **Default:** don’t break API for old mobile JS; deploy server often; store release when native must change.
- **Avoid:** large Meteor + Cordova plugin jumps in repo while store APK is years behind **without** either a store release or a **temporary pin** to production compat.

---

## Checklist (quick)

**Routine deploy (most of the time)**

- [ ] Deploy **without** pin
- [ ] Changes backward compatible for JS already on users’ devices?

**Before a risky server deploy** (Meteor or Cordova plugin bump, store not caught up)

- [ ] Can you **ship store first** instead of pinning? (preferred)
- [ ] If server must go first → **pin** to **store** compat, then redeploy
- [ ] After store catches up → **remove pin** on next normal deploy

**Before a store release**

- [ ] Release APK aligned with repo native changes
- [ ] Next server deploy: default **no pin** unless still bridging

**After incidents**

- [ ] Recover production compat from **Play Store / App Store APK**
- [ ] Pin + redeploy to stabilize, then plan return to **unpinned** workflow

---

## FAQ

**Do we need to pin every deploy?**  
**No.** Default is **no pin**. Pin only for recovery, deploy-before-store gaps, or other special cases in this doc.

**What does `METEOR_CORDOVA_COMPAT_VERSION` do?**  
Optional deploy-time override: build/serve hot-update JS for that **native compat line** only. Allowlist for HCP, not the App Store version.

**Does mismatch cause a white screen?**  
Usually **no**. Mismatch → no hot update → app keeps existing JS. White screen = **JS that ran** crashed (often: new JS on old native, or very old JS on a server that moved too far).

**If compat matched before pinning, why did the app break?**  
Same compat **id**, **different** JS artifact on the server after the big deploy. Match = allowed to download; content was wrong for the APK.

**What changed after we pinned?**  
We **redeployed** so the server’s hot-update bundle was built for **production native**, not the bumped repo stack. Phone id unchanged; **server payload** changed.

**Does Meteor store every old bundle?**  
**No.** Latest deploy wins. Roll back via git + redeploy (with correct pin).

**New APK + pin server to new compat — do old apps break immediately?**  
**Not from the pin alone.** They stop getting HCP and keep local JS. They can break later if the **server** no longer works with that old JS.

**New APK + pin still to old compat?**  
Possible **bridge**: new binary intentionally aligned to `OLD` so HCP keeps working until you bump pin and native together.

**Is compat hash = Meteor version + plugin versions?**  
Mostly **Cordova native fingerprint** (platforms + plugins in the native build). Meteor/plugin changes in the **repo** change **deployed JS**; the phone’s compat id only changes when the **store native** build changes.

**HCP vs “new bundle”?**  
Same thing for the web layer: HCP **delivers** the updated Meteor web bundle.

**Weekly server deploys were fine; one big deploy broke. Why?**  
Small deploys stayed compatible with old native/JS. One deploy crossed the line (Meteor + plugins) while store native was unchanged — a case where a **temporary pin** (or avoiding that deploy until store release) would have helped.

**When should we go back to unpinned deploys?**  
After production store APK and repo Cordova/Meteor stack are aligned again, and a normal deploy’s hot-update JS is safe for that native line.

---

## Related

- [Meteor: Hot Code Push for Cordova](https://guide.meteor.com/hot-code-push) — official reference
