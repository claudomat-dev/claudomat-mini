# Agentic outbound via MCP — Pancake benchmark

- **Researched:** 2026-10-23
- **Evidence quality:** hands-on, read-only. Pancake MCP (`https://app.pancake.ai/api/mcp`) connected as a Claude connector to workspace "Brawe AI" (Pro trial, ends 2026-10-28). Only read tools were called — no Play created, no search run, no message sent. Marketing-claim section is secondhand (search summaries of vendor pages/listings).
- **Re-verify by:** 2026-12-22 (60-day freshness rule)

## What Pancake is

"The Vercel for GTM" (Product Hunt, mid-Oct 2026). A hosted GTM backend exposed as ~100 MCP tools (OAuth/CIMD, works in Claude, Claude Code, ChatGPT, Codex; not Cursor). Plus a web app. Targets solo founders, small B2B teams, agencies.

## Architecture observed through the MCP

**1. Brain** (`brain_*`) — one structured GTM knowledge object per workspace, auto-built from the website at onboarding:
company summary + value prop + key features + social proof; ICP (industries, geos, size band, job positions split buyer / champion / user); voice (archetype, guidelines, preferred words, avoided words, **banned claims**, per-channel variants with samples); personas (pains, watering holes); message pillars (pain → value prop); objection responses; market references (competitors + influencers with LinkedIn URLs and positioning notes); watched keywords (product / competitor / problem). Every item has id + revision. An "improvement loop" writes **proposals** (create/revise/archive or questions) from lead feedback and outcomes; nothing changes until a human accepts.

**2. Signals** (`signal_settings_*`) — weighted catalog: keyword (1.0), competitor (0.9), influencer (0.8), own_brand (0.7, needs own LinkedIn), hiring, tech stack; `position_change` is **not implemented**. Signal sources are LinkedIn post engagement (people who engage with watched competitors/influencers/keywords), TheirStack (hiring, stack), Coresignal (people DB).

**3. Plays** (`plays_*`) — a saved audience + sourcing policy + one LinkedIn sequence + schedule. Sourcing is a **fallback waterfall**: watchlist post-engagers → keyword post-engagers → company-stack → company-hiring → persona sweep (plain title/industry/size/geo DB search). Each stage: discovery → pushdown filters → pivot (engager lottery / persona stage) → `icp_judge` (LLM) with hard constraints and tolerance → enrich (profile scrape). Target "10 new leads per run", stop at target, daily at a fixed local time.

**4. Credits** — research is charged at actual cost (a run holds 5,000, settles at real spend); work email 50 credits, phone 500. Trial = 30,000 credits / 7 days. Observed runs cost 1,599 and 210 credits.

**5. Sequences** (`campaign_*`) — **LinkedIn-only**, sent from the member's own LinkedIn account (several accounts can be pooled):
`visit_profile → like_post → if_connected:` msg1 (+6h) → msg2 (+72h) → msg3 (+72h); `else:` invite (note optional, Premium-only, ≤195 chars; withdraw after 7d) → on accept +6h → same 3 messages. Messages are AI-written per person from step guidelines (or fixed text with variables), 17 languages. Default guidelines: msg1 = first name + one specific detail from their activity + one-sentence problem + one real question, no link; msg2 = one new concrete point, never mention no-reply; msg3 = short, warm, easy to say no. Hard-coded generic fallbacks if AI can't write.

**6. Review & feedback** — leads land in "To review"; approve/reject/feedback (`lead_feedback_*`), promote, disqualify. Feedback feeds Brain proposals.

**7. Ops** — CRM export, Slack delivery, daily/weekly email digest, `activity_since` cursor feed for agents, guidance object on every tool result (next recommended action + risk level + whether user authorization is required).

## What actually happened in this workspace (reality check)

- 2 Plays, 57 leads total (35 + 22), **0 reviewed, 0 enrolled, 0 contacted**. The LinkedIn sender is **disconnected**, so the sequence is paused — no outreach has ever run.
- **All sampled leads came from the last-resort persona sweep** (Coresignal title search). `originSignal: null`, `warmness: 0`, `brainEvidence: []` on every one. The signal-first stages yielded nothing; the run stopped with `sources_dry`.
- "Why they fit" is just the title restated ("Founder at X, 1 employee, London"). No intent, no trigger.
- Fit is loose: executive coaches and "Senior Consultant & Dashboard Manager" pass because "Consultant" is in the title list. Onboarding left junk personas in the Brain ("housewife", typo "serial enterpreneur") and they flow straight into the ICP judge's title list.
- No work emails fetched (costs extra), so only LinkedIn is reachable.
- **Replies are not handled**: the tool docs state "Pancake never answers one itself" — contradicting marketing copy about auto-sending a calendar link on "yes".

## Marketing claims vs. observed

| Claim (secondhand) | Observed |
|---|---|
| Signal-driven leads with reasons and sources | Signals exist, but here every lead came from a plain title search with no signal |
| Sends from your own account after warm-up | True, but LinkedIn-only; "warm-up" = profile visit + post like |
| Up to 3 messages | Confirmed: 3 messages at +6h / +72h / +72h |
| Yes → calendar link auto-sent | Not in the product surface; replies go to the human |
| 40+ data providers | Seen in use: Coresignal, TheirStack, LinkedIn post search, profile scrape |

## Where we beat it

| Pancake gap | Our design |
|---|---|
| LinkedIn-only, one account bottleneck; nothing runs when it disconnects | Multi-channel: email (own warmed domains) + LinkedIn + X/Reddit/community touchpoints; channel failover |
| Signal stages dry up → silently falls back to cold title search | Never ship a signal-less lead as "signal-driven"; label tier explicitly (hot/warm/cold) and ask before burning credits on cold sweep |
| "Why they fit" restates the title | Evidence card: the actual post/comment/job/stack/trigger, quoted + dated + linked |
| No reply handling | Reply classifier (interested / objection / referral / OOO / not-now / unsubscribe) + Brain-grounded draft for each; calendar link with approval |
| Onboarding garbage reaches the ICP judge | Brain linting: reject placeholder personas, typos, over-broad titles; show expected precision before saving |
| Loose title matching (coaches pass as founders) | Negative criteria + precision sampling: judge on 20 leads, user thumbs up/down, tune before scaling |
| No position-change signal | Champion/job-change + new-founder (company created <90d) signals |
| Feedback → proposals only | Closed loop on outcomes (accept rate, reply rate, meetings) per signal/persona/message pillar; auto-reweight with audit |
| Credits opaque per lead | Cost per lead and per tier shown before approval; budget cap per Play |
| Fixed 3-step cadence | Adaptive cadence by engagement (profile view back, like, open) |

## Patterns worth copying

- Structured Brain with ids/revisions, **banned claims**, per-channel voice — and every write tool requires grounding in it.
- Waterfall sourcing with per-stage budget and a machine-readable run `report` (cost, rejections by reason, stop reason, advice).
- `guidance` on every tool response (next action, risk, needs-user-authorization) — makes agents safe by default.
- `activity_since` cursor feed instead of polling.
- Human approval before any outreach; proposals instead of silent self-edits.

## Sources

- Hands-on: Pancake MCP read tools, workspace "Brawe AI", 2026-10-23
- https://pancake.ai/install.md (v2.1.0) · https://pancake.ai/ · https://pancake.ai/claude
- https://zipoapps.net/pancake-review-gtm-plays-claude-codex-mcp/
- https://dynamicbusiness.com/ai-tools/pancake-signal-driven-gtm-automation-platform.html
- https://slashdot.org/software/p/Pancake/ · https://topai.tools/t/pancake-ai · https://digg.com/ai/e49vqbe1
