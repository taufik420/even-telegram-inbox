# TG Heads — Telegram on Even G2 Smart Glasses

**Heads Up: Even Realities Hackathon** (TVM Ventures × Daos Hub × Even Realities) ·
Track **Messaging & News** · Solo · Remote · No G2 hardware — simulator only.

> **Status: PREP ONLY (21 Nov 2026).** App code starts **23 Nov 2026**, per the
> hackathon rule *"Build during the hackathon"*. What exists before 23 Nov:
> this repo, this README, and [`PLAN.md`](PLAN.md). No app features have been
> written yet. Anything pre-existing (libraries, APIs, starter template) is
> listed in PLAN.md as required by the rules.

## Vision

Telegram, truly on your glasses. One unread message per screen, glanceable in
seconds — and you can answer without taking your phone out: voice reply,
approve an AI-drafted reply, or tap a quick reply. **Inbox in 30 seconds.**

## The one app: three ideas, unified

All three track ideas live in **one glasses UI** (one card per screen, read AND
write via voice / taps / R1 ring), with **two Telegram backends** behind it:

| # | Feature | Backend | What it does |
|---|---------|---------|--------------|
| 1 | **Inbox in 30 Seconds** ⭐ (core) | MTProto (GramJS, your own account) | One unread message per screen (sender + 1-line preview + time). Up/Down = next message. Tap = act: 🎙️ voice reply (mic → STT → send as you), ✅ approve AI-drafted reply, 👍 quick reply. |
| 2 | **Channel Digest** | Bot API (fallback) | AI summary (1–2 lines) of new posts from channels you follow. One digest card per screen. Tap = 👍 react / save to Saved Messages. |
| 3 | **Triage + AI Drafts** | Bot API (fallback) | Companion bot forwards important messages; backend drafts a short reply with an LLM. Glasses show message + draft. Tap = send draft, voice = dictate your own, swipe = skip. |

**Why two backends:** MTProto is the deepest real Telegram integration
(full inbox — the strongest candidate for the **$1,000 Best Telegram
Integration** prize) but carries auth/session risk. Bot API is the safe
fallback: if MTProto auth misbehaves mid-week, the demo still runs on the
same UI. No mock data in the demo — real Telegram, end to end.

## Stack

- Glasses app: **Vite + TypeScript + `@evenrealities/even_hub_sdk`** (0.0.14),
  Even Hub CLI 0.2.0, Even Hub simulator 0.9.3 (free, no G2 needed).
  Scaffold: official `evenhub-templates/asr` (mic → STT pipeline stub).
- Backend (own server): Node + GramJS (`telegram`) for MTProto; Bot API via
  grammy/telegraf for the fallback; Whisper (or Deepgram/AssemblyAI) for STT;
  LLM for reply drafts and channel digests.
- Glasses ↔ backend: SSE / WebSocket from our own server.
- Packaging: `.ehpk` via `evenhub pack` (optional for submission, required
  for Even Hub catalog distribution).

## Prizes in reach

- Best Telegram Integration **$1,000** · Track 1st Messaging & News **$1,250**
  + G2 · Grand Prize $2,000. (One project can win only one cash prize.)
- Submit by **29 Nov 23:59 Dubai = 30 Nov 2026 02:59 WIB**. Demo Day 3 Dec
  (remote demo allowed).

## Security & privacy

- **No secrets in this repo, ever.** No API keys, no session strings, no
  tokens — `.ehpk` files can be extracted by anyone.
- Telegram `api_id` / `api_hash` are created by the user at
  [my.telegram.org](https://my.telegram.org/apps) and live only on the
  backend server. The MTProto session string is the user's own, stored
  mode `0600`, never committed, never pasted in chat.
- Demo uses only the user's own account, chats, and groups. Messages from
  other people are never shown publicly.
- No payments, no custody, no financial advice — read-only / paper by
  design; the app never holds user funds.

## Links

- Hackathon (DoraHacks): https://dorahacks.io/hackathon/even-realities/detail
- Even Hub docs: https://hub.evenrealities.com
- Templates: https://github.com/even-realities/evenhub-templates
- Research notes: `~/workspace/research_notes/even-realities-telegram-track.md`

## Build

See [`PLAN.md`](PLAN.md) for the 7-day plan (23–29 Nov), the submit
checklist, and the exact simulator command verified on this machine.
