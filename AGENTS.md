# The fleet

Every agent is one job, one trigger, one tight scope. This is the full
catalog. Roles are generalized archetypes; the real system is wired to specific
tools and data sources that aren't shown here.

## Orchestrator

### Larry
**Role:** the conductor. **Trigger:** every operator message.
Understands intent, routes to a specialist, hands over context, collects the
result, summarizes. Does small things inline; never does a specialist's job.

---

## On-demand specialists

### Vera — document verifier
**Trigger:** "check this quote / document against the source."
Compares a document to its source of truth (counts, prices, addresses), reports
deltas. **Scope:** read-only.

### Conny — meeting prep
**Trigger:** "what's on today," "prep my meeting."
Reads calendar + related mail (+ optional CRM), returns a compact per-meeting
brief: what it's about, what to bring. **Scope:** read-only.

### Yeti — inbox triage
**Trigger:** "what needs attention," "inbox check," "anything I missed."
Sweeps email + chat (DMs and threads where I'm tagged) → a scannable digest:
needs-action / FYI / open to-dos. **Scope:** read-only.

### Toni — ticket triage
**Trigger:** "ticket check," "what's open on my tickets."
Triages support/CRM tickets across pipelines, maintains a shared board file
that the inbox-triage agent folds in. **Scope:** read-only to the CRM; writes
only the internal board.

### Mara — reply drafter
**Trigger:** "draft a reply to …," "draft for the request from …."
Reads the thread, pulls context from CRM/data, writes a reply **in my style**
(learned from sent mail), marks missing facts as placeholders. **Scope:**
creates a draft; never sends. Can also re-learn the writing-style profile on
request.

### Nora — follow-up chaser
**Trigger:** "what's stalled," "where do I need to follow up," "follow-up check."
Finds threads that went quiet across a few categories (resource confirmations
pending, quotes unanswered, approvals outstanding), returns a digest and hands
the threads to the drafter. **Scope:** read-only.

### Felix — delivery watcher
**Trigger:** "delivery check," "what's done," "who do I need to notify."
Detects when a deliverable is reported done in a team channel and flags the
customer notification that's now due; enriches from the data export. **Scope:**
read-only; writes only a board file.

### Piet — resource routing
**Trigger:** "who can take job X," "open jobs without a resource," "routing."
Pulls open jobs with real value, matches them against a resource roster by
proximity / value-vs-travel / experience / preference, returns a ranked
suggestion list. Proposes only — the human makes the actual request. Also runs
as a scheduled job to feed the morning briefing. **Scope:** proposes; writes
suggestion files.

### Bea — bulk importer
**Trigger:** "import CSV," "create new records," "bulk upload."
Turns a list/table/email into a validated bulk-**import** CSV (new records),
picks the right template, asks for missing required fields, returns CSV + a
check report. **Scope:** produces a file; never touches the live system.

### Edi — bulk editor
**Trigger:** record IDs/links + fields to change ("set field X for these IDs").
Builds a bulk-**edit** CSV with changed columns only, for existing records.
**Scope:** produces a file; never touches the live system.

### Greta — report builder
**Trigger:** recurring key-account tables ("enrich the planning file," "build
the weekly table," "import CSV from the account's planning").
Builds the recurring tables for a key account from their planning files via
tested scripts. **Scope:** produces files only.

---

## Scheduled automations (cron, no human trigger)

### Bruno — morning briefing (daily, early)
Reconciles the day's data export with CRM/chat/mail and posts the full
operational briefing. Folds in findings from the pre-check jobs below.

### Falk — external-constraint pre-check (daily, before Bruno)
Flags open records whose location falls inside a restricted zone, via a
geospatial/regulatory API. An early-warning signal, not an approval — folds
into Bruno. Deterministic, token-free.

### Hugo — history snapshot (daily, first)
Token-free import of the day's data export into a local SQLite DB. Makes status
changes datable ("since when?"), lets records survive a rolling-window export,
and rejects suspiciously filtered exports instead of importing them.

### Walter — weekly report (weekly)
Builds the week-over-week KPI report and an HTML visualization.

### Rita — learning loop (weekly)
Compares proposals to reality (draft vs. sent, suggested vs. chosen, snoozed
items) and writes human-gated improvement proposals. See
[docs/the-learning-loop.md](docs/the-learning-loop.md). Never edits heuristics
itself.

### Klara — vault maintenance (monthly)
Consolidates the memory vault — duplicates, stale facts, dead links, index
gaps. Cosmetic/additive fixes directly; content changes only as proposals.
Hard taboo: never rename/move/delete files wired into scripts.

---

## How a new agent gets added

1. Identify recurring manual work or a pattern worth automating — log it as a
   proposal first.
2. Define the four attributes: **trigger, tool scope, output contract, side
   effects**.
3. Default to read-only. Justify any write access explicitly.
4. Wire it into the orchestrator's routing table with a crisp trigger.
5. If it should run unattended, add a deterministic scheduled variant.

Build only after the human signs off — the system never spawns new agents
unprompted.
