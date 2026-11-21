# Build Plan — TG Heads (Even Realities "Heads Up", Messaging & News)

**Build week:** 23–29 Nov 2026 · **Submit deadline:** 29 Nov 23:59 Dubai =
**30 Nov 2026 02:59 WIB** · **Demo Day:** 3 Dec (remote allowed)
**Team:** solo (Cipoz) · **Hardware:** none — Even Hub simulator only

## What already exists before 23 Nov (per hackathon rules)

- This repo, `README.md`, this `PLAN.md` — prep only, no app features.
- Research notes: `~/workspace/research_notes/even-realities-telegram-track.md`
- Starter template `evenhub-templates/asr` — official scaffold, **not yet
  moved into the repo** (see "Scaffold status" below). No app code written.
- Open-source libraries + public APIs (Even Hub SDK, GramJS, Bot API,
  Whisper/LLM providers) are pre-existing and allowed.

## The app (one UI, two backends)

- **Glasses UI (one card per screen):** unread message / digest card / draft
  card. Up/Down = next, Tap = act, Double-tap = exit (system dialog).
- **Backend A — MTProto (GramJS, primary):** user's own account; unread
  dialogs + send as user. Session string mode `0600` on our own server.
- **Backend B — Bot API (fallback):** companion bot + channel digest; same UI.
- **Voice:** glasses mic PCM → STT (Whisper) → text; LLM drafts replies.
- **Packaging:** `app.json` — proposed `package_id: com.cipoz.tgheads`,
  name `TG Heads` (≤20 chars), permissions `network` (whitelist own backend)
  + `g2-microphone`. No secrets in the app or `.ehpk`.

## Scaffold status (21 Nov — READ BEFORE 23 Nov)

- Template `asr` was downloaded (degit) to `asr-template-tmp/` for
  inspection **only**. Moving its files into the repo / installing / running
  it is **BLOCKED pending a user go-ahead** (execution-risk check on the
  downloaded content). On 23 Nov, either get that approval and move the
  files in, or re-scaffold cleanly with:
  `npx degit even-realities/evenhub-templates/asr .` (then `npm install`).
- ⚠️ **Template pins OLD versions:** SDK `^0.0.10`, CLI `^0.1.12`,
  simulator `^0.7.2`. Bump on H1 to the current ones: SDK **0.0.14**,
  CLI **0.2.0**, simulator **0.9.3** (verified docs, 29 Aug 2026).
  Also replace `app.json` (`com.example.evenhubasr` / "Even Hub ASR") with
  our own `package_id` / name, and add the `network` permission.
- Template gotcha (from its README): STT is a blank stub in
  `src/asr/stt.ts` — wire a provider there. Tap events arrive without an
  `eventType` field (`CLICK_EVENT` = 0, dropped by protobuf) — copy the
  handler pattern from the template's `src/main.ts`.

## 7-day plan

### H1 — Mon 23 Nov: scaffold + simulator + Telegram auth
- [ ] Scaffold `asr` template into repo root (see Scaffold status above);
      bump SDK/CLI/simulator to 0.0.14 / 0.2.0 / 0.9.3; `npm install`.
- [ ] **Simulator check FIRST** (flagged day-1 risk — see "Simulator" below).
      Get one line of text + one tap working in the simulator, as the
      organisers suggest.
- [ ] User creates Telegram `api_id` / `api_hash` at
      https://my.telegram.org/apps (2 min, user only — never shared).
- [ ] GramJS login on the backend (phone + code / QR); session string saved
      mode `0600`. Check the Even Hub catalog to confirm the idea isn't a
      duplicate.

### H2 — Tue 24 Nov: MTProto backend
- [ ] Backend: list unread dialogs, fetch one message per screen.
- [ ] Backend: send message as user (quick reply + free text).
- [ ] Push updates to the glasses app via SSE / WebSocket.

### H3 — Wed 25 Nov: glasses UI + voice
- [ ] UI: inbox list, message card, quick replies, tap navigation.
- [ ] STT: mic audio → Whisper → text shown on glasses; voice reply sends.

### H4 — Thu 26 Nov: AI drafts + polish
- [ ] LLM drafts a short reply per message; Approve / Reject on glasses.
- [ ] Glanceable polish: one thing per screen, few words, 576×288 monochrome.

### H5 — Fri 27 Nov: Bot API fallback + end-to-end test
- [ ] Fallback Bot API behind the same UI (channel digest + triage bot).
- [ ] End-to-end test in simulator; scripted input + screenshots via the
      simulator automation API (`--automation-port 9898`).

### H6 — Sat 28 Nov: demo video + packaging
- [ ] Record demo video (max 3 min, simulator, real read → write flow).
- [ ] `npm run build && evenhub pack app.json dist -o tgheads.ehpk`.
- [ ] README + submission description finished; re-check catalog for dupes.

### H7 — Sun 29 Nov: buffer + submit
- [ ] Fix buffer; final verification (boots clean, exit dialog on
      double-tap, no keys in repo/`.ehpk`, demo uses own account only).
- [ ] **Submit before 23:59 Dubai / 30 Nov 02:59 WIB.**

## Simulator (headless VM) — ✅ VERIFIED WORKING 21 Nov 2026

**Status: WORKS.** The full loop was verified on this VM with a minimal
SDK smoke test (own code in `/tmp`, NOT the repo): Vite dev server +
simulator under Xvfb + automation API + a one-line `createStartUpPageContainer`
("HELLO SIM").

Evidence (21 Nov ~12:45 UTC):
- `GET /api/ping` → `pong`; `GET /api/console` shows
  `[EvenAppBridge] Bridge initialized` and `BRIDGE OK: 0` from
  `createStartUpPageContainer`.
- `GET /api/screenshot/glasses` → valid PNG **576×288 RGBA** containing the
  green `HELLO SIM` text (**577 pixels at green=255**).
- The simulator's X window also shows the text + companion WebView +
  control buttons (Up / Down / Click / Double Click / Long Press / Menu).
- `POST /api/input {"action":"click"}` → `{"ok":true}`.

### Setup (run once per VM — `/usr` and `/etc` are wiped on VM replacement)

```bash
npm install -g @evenrealities/evenhub-cli@0.2.0 @evenrealities/evenhub-simulator@0.9.3
apt-get update && apt-get install -y libwebkit2gtk-4.1-0
```

Notes:
- The simulator binary needs `libwebkit2gtk-4.1.so.0`,
  `libsoup-3.0.so.0`, `libjavascriptcoregtk-4.1.so.0` — all provided by
  `libwebkit2gtk-4.1-0`. Without it the binary fails to start.
- `apt-get update` is VERY slow through this VM's proxy (took ~14 min on
  21 Nov and was killed at ~435 MB of lists; the install then worked by
  copying `/var/lib/apt/lists/partial/*` into `/var/lib/apt/lists/`).
  Allow plenty of time, or reuse the copied-lists trick if it stalls.
- `xvfb-run` / `Xvfb` are already present. `libgl1-mesa-dri` (llvmpipe /
  swrast software rendering) is installed; the `libEGL warning: DRI3
  error` in the simulator log is harmless — rendering falls back to
  software GL and works.

### Run (exact working commands)

```bash
# Terminal 1 — dev server (from repo root)
npm run dev
# Terminal 2 — simulator under a virtual display, with automation API
xvfb-run -a evenhub-simulator --automation-port 9898 http://localhost:5173
# Smoke test (wait ~4 s after launch)
curl -s http://localhost:9898/api/ping
curl -s http://localhost:9898/api/screenshot/glasses -o glasses.png
```

Gotchas found during verification:
- A plain HTML page (no SDK) renders a **blank** glasses display — the
  canvas only shows SDK containers. Test with a real
  `createStartUpPageContainer` call, not a static page.
- The screenshot PNG uses row filters — decode it with a real PNG decoder
  (or unfilter in code). A naive raw-byte read reports "0 lit pixels"
  even when the text is there.
- If the window looks wrong after an edit, restart the simulator before
  assuming a bug (hot-reload is reported unreliable).

## Submit checklist

- [ ] Public repo + README explaining setup (incl. what pre-dated 23 Nov)
- [ ] Demo video ≤ 3 min (simulator, real Telegram read → write)
- [ ] Short description: problem, audience, how it works, track
      (Messaging & News)
- [ ] (Optional, recommended) `.ehpk` build
- [ ] Feature requests for APIs used (if relevant)
- [ ] No API keys / session strings in repo or `.ehpk`
- [ ] Demo uses only the user's own account + chats + groups

## Risks

1. Simulator won't run headless → de-risked H1 (see above); fallback: build
   on the user's own computer.
2. MTProto auth / session issues → Bot API fallback already planned (H5).
3. STT + LLM latency in the demo → rehearse the video scenario; tap quick
   replies always work as a shortcut.
4. Duplicate of an existing Even Hub catalog app → check catalog H1 and H6.
