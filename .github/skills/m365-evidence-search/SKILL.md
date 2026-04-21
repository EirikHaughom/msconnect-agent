---
description: Multi-pass M365 evidence search using WorkIQ. Searches emails, Teams messages, meetings, and documents with structured passes, per-channel coverage, and built-in anti-patterns.
---

# M365 Evidence Search

A structured, multi-pass methodology for exhaustively searching Microsoft 365 data using WorkIQ. Use this skill whenever you need to gather evidence, context, or activity history from M365 sources.

## Execution Rules

1. **Do NOT issue one broad search.** Run multiple targeted passes and union the results.
2. **Break into small, specific queries.** WorkIQ performs poorly on broad aggregations. Instead of "summarize my work for the last 6 months," ask targeted questions:
   - _"What emails did I send or receive about [topic] in [month]?"_
   - _"What meetings did I attend related to [project] in [month]?"_
   - _"What documents did I create or edit about [topic] in [month]?"_
3. **Run separate WorkIQ calls per month** when doing temporal searches — never ask for multi-month summaries in a single call.
4. **Expect missing data.** Not all meetings have transcripts. Some evidence lives outside M365. Flag gaps explicitly rather than silently omitting them.
5. **Parallelize** independent queries where possible.

## Search Passes

### Pass 1 — Topic-Focused Evidence

For each topic, project, or priority provided by the caller, search across:

- **Emails** — decisions made, outcomes communicated, stakeholder alignment, customer/partner threads
- **Teams messages** — collaboration evidence, cross-team work, knowledge sharing
  - Include **1:1 chats, small group chats, channel messages, and meeting chats** — these are separate stores in Teams
- **Meetings** — calls, presentations, strategy reviews (use transcripts when available, calendar context when not)
- **Documents** — deliverables, presentations, reports, technical assets, published content
- **People** — organizational context for collaborators mentioned

Extract search terms from the caller's input: names, project names, customer/partner names, internal programs, and topics.

**Locale-aware searching:** If the user works in a non-English-speaking country (as identified by the role lookup), include local-language equivalents of key search terms when searching for evidence involving local customers, partners, or events. Conversations with local customers and partners often occur partly or fully in the local language — searching only in English will miss this evidence.

### Pass 2 — Per-Channel Sweep

Explicitly search each channel type to catch items missed by topic searches (commonly missed ones marked ⚠️):

1. **Emails** — inbox + sent + forwarded + meeting follow-up emails
2. **Teams 1:1 chats**
3. ⚠️ **Teams small group chats** (ad-hoc multi-person, commonly missed)
4. **Teams group/channel messages** — internal team channels, customer/partner channels
5. ⚠️ **Teams meeting chats** (separate store, contains post-meeting discussion)
6. ⚠️ **Edited messages** (enumerate the full day, not just the first thread)

### Pass 3 — Per-Month Calendar Sweep (Safety Net)

For each month in the date range, ask a time-bounded question to catch items the topic passes missed:

_"What significant emails, meetings, Teams discussions, or documents were related to [broad topic area] in [Month Year]?"_

This time-first sweep provides coverage the topic-first passes may miss.

## Consolidation

Union results from all passes. Deduplicate by (date, source, content prefix).

For each evidence item, capture:
- **Date** (with time if available)
- **Source** (person, organization, role)
- **Channel** (Email / Teams 1:1 / Teams small group / Teams channel / Teams meeting chat / Document)
- **Context** (what topic, project, customer, or event it relates to)
- **Content** (summary of the evidence — what happened, what was decided, what was delivered)
- **Confidence**: Strong (direct evidence) | Partial (inferred or approximate) | Weak (indirect only)

## Self-Critique Gate

Before returning results, verify:

- [ ] Did I search Teams **small group chats**? (commonly missed)
- [ ] Did I search Teams **meeting chats**? (commonly missed)
- [ ] Did I include **edited messages** and full-day enumeration?
- [ ] Did I cover **every topic/project/priority** with at least one search pass?
- [ ] Did I flag **gaps** where evidence is thin or missing?
- [ ] Did I run the **per-month safety net** sweep?

If any answer is "no" — rerun the relevant pass.

## Anti-Patterns to Avoid

- ❌ Relying on a single broad question — always multi-pass
- ❌ Filtering to only emails when Teams is often the richer source
- ❌ Stopping at the first thread of a busy day (edited messages lurk later)
- ❌ Asking for multi-month summaries in a single WorkIQ call
- ❌ Declaring "not found" without running all passes
- ❌ Skipping small group chats because they're neither 1:1 nor channel
- ❌ Prematurely concluding search is done after one successful pass
- ❌ Ignoring calendar context when meeting transcripts are unavailable

## Privacy Rules

- Focus on evidence of the **user's own contributions and impact**
- **Summarize** sensitive material rather than reproducing private messages verbatim
- Do not surface confidential information about other employees
- Reference source type and approximate date, not full message content
