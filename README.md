# BrainTrust — Project Meridian

> A Claude-powered shared brain for consulting teams. Drop in what you know. Ask it anything.

## The Problem

Every engagement has the same issue: knowledge lives in 12 different places. Slack threads, email chains, a SharePoint nobody updates, someone's personal notes. When a new team member joins, they spend a week hunting context. When a partner asks "what did we decide about X?", someone has to remember.

**BrainTrust** is the fix: a single place where the team drops everything they know, and a Claude-powered agent that can answer questions across all of it.

---

## What It Does

| Phase | Description |
|-------|-------------|
| **Ingest** | Anyone on the team pastes a note, uploads a doc, drops a meeting transcript, or writes a decision |
| **Tag** | Content is categorized: `meeting-note`, `decision`, `open-question`, `client-context`, `risk` |
| **Query** | Ask anything in plain English: *"What did we decide about the timeline?"* |
| **Synthesize** | Ask it to produce something: *"Draft the brief for tomorrow using everything in here."* |

---

## Team

| Name | Role | GitHub |
|------|------|--------|
| Kate Crotty | Technical | @kacrotty |
| Contributor | Functional | @cabsher_deloitte |
| Contributor | Functional | @jpbarcenas_deloitte |
| Contributor | Designer | @jwilner_deloitte |

---

## Content Taxonomy

The brain accepts exactly five content types:

- **`meeting-note`** — Summaries or transcripts from team or client meetings
- **`decision`** — Resolved choices, with rationale and decision-maker noted
- **`open-question`** — Unresolved issues that need an answer before the engagement can move forward
- **`client-context`** — Background, preferences, sensitivities, and history about the client
- **`risk`** — Identified risks with likelihood, impact, and owner

### What Goes In / What Doesn't

**Goes in:** Anything a team member would need to know to contribute to the engagement.

**Doesn't go in:** Raw data files, financial models, external research not yet reviewed by the team, or anything not directly relevant to Project Meridian.

---

## Architecture

```
POST /ingest     ← add a note/doc to the brain (JSON file store)
POST /query      ← ask a question; all items stuffed into Claude's context
GET  /items      ← list everything in the brain
```

### Key Architectural Decision: Context Stuffing vs. Embeddings

For this demo, we use **context-window stuffing** — all notes are loaded into Claude's context on every query. This is deliberately simple:

- Zero infrastructure overhead
- Works perfectly for a corpus of ~20–100 notes
- Lets us ship a working demo in 2 hours

**In production**, we'd swap to **embeddings + vector search** (e.g., pgvector or Pinecone):
- Retrieve only the top-K relevant items per query
- Scale to thousands of documents
- Reduce cost per query significantly

The interface contract (`POST /query` → answer) stays identical — swapping the retrieval layer doesn't change the UI or the agent behavior.

---

## What We Built

Built in 2 hours at the Deloitte AI Hackathon. The stack is intentionally minimal:

- **Frontend:** Next.js (React) — dashboard, add-to-brain form, query + answer view
- **Backend:** API routes in Next.js — ingest, query, and list endpoints
- **Storage:** JSON file store — no database needed for a hackathon demo
- **AI:** Claude API — single call per query with full context stuffed in

---

## Golden Query Set

These are the 10 queries the brain must answer correctly (our eval harness):

1. What did we decide about the go-live date?
2. What are the open risks with a legal dimension?
3. Summarize everything we know about the client's procurement process.
4. What's still unresolved going into next week?
5. Who owns the risk related to data migration?
6. What was discussed in the most recent client meeting?
7. What is the agreed timeline for Phase 1 delivery?
8. What are the client's known sensitivities we should be aware of?
9. Summarize all open questions for the partner update.
10. What decisions have been made about team structure?

### Refusal Cases (Brain should not guess)

- *"What is the client CEO's salary?"* — not in the brain, should flag as unanswerable
- *"What did the client say in yesterday's call?"* — if no transcript exists, should say so
- *"What will the client decide about the contract?"* — future prediction, not in scope

---

## How to Run It

```bash
git clone https://github.com/kacrotty/Table24.git
cd Table24
npm install
```

Create a `.env.local` file:
```
ANTHROPIC_API_KEY=your_key_here
```

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

---

## How We Used Claude Code

- Generated the initial project scaffold and API route structure
- Used Claude to draft the `CLAUDE.md` domain context, which improved query quality significantly
- Iterated on the system prompt for the query endpoint to enforce refusal on out-of-scope questions
- Built `presentation.html` entirely with Claude Code

---

## What We'd Build Next

1. **Embeddings + vector search** — scale beyond context window limits
2. **Slack / Teams integration** — ingest directly from conversation threads
3. **Access controls** — partner-only vs. full team visibility tiers
4. **Audit trail** — who added what and when, with change history
5. **Proactive surfacing** — "3 open questions haven't been touched in 5 days"

---

## Project Status

- [x] Repo initialized
- [x] Content taxonomy defined
- [ ] Backend API (ingest / query / list)
- [ ] Frontend dashboard
- [ ] Demo dataset (20 realistic items)
- [ ] Golden query eval
- [ ] presentation.html
- [ ] CLAUDE.md complete
