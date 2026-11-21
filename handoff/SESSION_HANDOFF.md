# Session handoff (public-safe summary), 2026-11-21

This file is deliberately generic. The fork is public, so it holds no names of people, bots, products or businesses, and no credentials. The full private context lives in a zip the owner keeps (`zeroclaw-bundle.zip`); a new session must be given that zip. Without it, only the summary below is available.

## What is being built

A cheaper automation setup for a team of AI bots, in three lanes:

| Lane | Does | Cost |
|---|---|---|
| Scripts | Everything deterministic (flags, counters, status files, queues, posting after approval, rendering) | No model |
| Small agent runtime (ZeroClaw or Hermes, **undecided**) on Google's free tier | Telegram front door, small repeatable language tasks | Free tier |
| Claude Pro, used only through its own apps and scheduled tasks | Creative, judgment and private work | Subscription |

Hand-offs between lanes are files plus a ticket queue; the owner approves; scripts post; Telegram notifies. Host: AWS, 2 GB RAM, 50 GB. Transcription: Whisper `base` (faster-whisper, CPU, int8). 66 tasks were listed, 57 are to be built.

## State

- Progress: about 12% (7.0 of 57), computed from the private task register (full task = 1, partial = 0.5, only tested-on-fakes counts today). Script lane about 21%.
- Tested on real Telegram, Gemini, Claude or X: **nothing**. Nothing runs in production.
- ZeroClaw v0.9.0 was measured with a fake model on a webhook channel (not Telegram): slim turn 1,174 tokens; `max_tool_result_chars = 3000` held six heavy turns to a 6.6k peak versus 29.3k stock; `max_context_tokens` is a soft limit; the default channel config makes 3 model calls per message; the daemon used about 52 MB.
- Hermes (commit 242315c) was measured earlier on fakes only: it enforces a 64K minimum model window.

## Decisions (do not re-litigate)

1. Scripts for deterministic work, a small runtime for small tasks, Claude Pro for creative and private work.
2. Hand-offs by shared files and a ticket queue, approved by the owner.
3. Every runtime call under 16K tokens (Gemma 4 limit). Note that Gemma's 16K is **per minute**, not per call, so one heavy turn nearly fills a minute. Flash-Lite models have no 16K limit (250K tokens a minute, 500 requests a day).
4. Private-data flows never touch the free tier (Google may use free-tier content).
5. Claude is used only through its own surfaces (agents may not use Pro credentials).
6. Whisper `base`; host 2 GB / 50 GB; the runtime choice stays open, so write everything runtime-neutral.
7. No secrets in chat: tokens go in an env file on the host; a Gemini key may go through the secure credential form.
8. Both forks are public: push no business details. Never push the private `scripts/` folder as is (docstrings name bots).

## Conflicts found this session (not yet resolved)

| # | Finding | Status |
|---|---|---|
| 1 | The 16K limit is per minute for Gemma 4, not per call | Design target stays 16K per call; budget the minute |
| 2 | ZeroClaw's native Gemini provider reportedly lacks function calling; the config routes around it via Google's OpenAI-compatible endpoint, which a report warns can fail | **Untested.** Highest-value check |
| 3 | An older report still mentions a pay-per-token Haiku option, which conflicts with "Claude only through Pro" | Drop it from the plan |
| 4 | Claude cloud scheduled tasks see only cloned repos and connectors, not files on the AWS host | Open: transport undecided |
| 5 | The queue tool's `claim` also takes unapproved tickets unless `--approved-only` is passed, which contradicts decision 2 | **Defect.** Make approved-only the default (about 5 lines plus a test) |
| 6 | Gemma 4 tool calling and system prompts via the API are unverified | Open |

## Open questions to the owner (unanswered)

1. Run the new lanes beside the existing platform and move flows one at a time, or replace the platform? (Recommendation: beside, then migrate.)
2. How does a Claude cloud scheduled task read the queue? (Recommendation: a **private** GitHub repo; host pushes queue files, Claude pushes drafts to a branch, a deploy key on the host. Confirm with one real routine.)

## Proposed next step (waiting for the owner's go-ahead)

1. Probe Gemma 4 and Flash-Lite with one function declaration and one system instruction each, directly and through ZeroClaw. Needs a short-lived Gemini key through the secure credential form and the network host `generativelanguage.googleapis.com` approved.
2. Same step: make the queue tool approved-only by default, with a test.
3. Then pick the runtime and run a one-flow Telegram pilot on the host (token in an env file).
4. Only the owner can start the Claude-lane test (a week of real scheduled tasks, reading `/usage`) and the baseline percent log.

## Working rules for the next session

- Do the work directly; no subagents unless a question truly needs parallel web research. Token cost matters.
- Plain language, direct answers, tables. Label every claim MEASURED, SOURCE-read, or NOT TESTED.
- Progress percentages come only from the private task register; never estimate one.
- The owner cannot browse the container: deliver files with `SendUserFile` and rebuild the bundle zip when files change.
- Do not start building until the owner confirms the next step.
- The repo's `AGENTS.md` rules apply here: no AI attribution footers, conventional commits, non-`master` branch, no PR unless asked.

## Housekeeping

- The fork has a leftover empty test branch `claude/zeroclaw-push-test-delete-me`; the owner will delete it.
- Not checked in this handoff: the docs quality and link gates were not run (single added file, no code).
