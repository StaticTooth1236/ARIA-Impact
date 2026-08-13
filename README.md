# ARIA — Agentic Reasoning for Impact Analysis

**Live at [aria-impact.com](https://aria-impact.com)**

ARIA is a production multi-agent AI system that performs engineering change impact
analysis for a vertically integrated aerospace program. Given a free-text change
request — a supplier swap, a schedule slip, a design change — ARIA reads a
19-document, ~214,000-word controlled program baseline and drafts a complete
10-section Project Management Plan update: decision gates, scored risk register
entries, reserve impacts, and owner assignments, streamed live to the browser in
about ten minutes.

Built end to end in four weeks of nights and weekends by a program manager using
AI pair-programming — dataset, retrieval architecture, agents, backend, frontend,
brand, and deployment.

---

## What it does

```
change request (free text)
        │
        ▼
┌─────────────────────────────────────────────┐
│  5 specialist agents — run in parallel      │
│  Change · Cost/Margin · Manufacturing ·     │
│  Supply Chain · Risk/Schedule               │
└─────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────┐
│  Document impact layer                      │
│  10 baseline documents assessed             │
│  individually, every run                    │
└─────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────┐
│  Streamed synthesis (senior-PM voice)       │
│  10-section PMP update, token by token      │
└─────────────────────────────────────────────┘
```

Roughly 14 LLM calls per analysis. ~10 minutes and ~$1.40 per full live run in
production.

## The key architectural decision

Standard RAG lets semantic similarity decide what gets read — and semantically
"loud" documents (like a risk register) win every ranking, starving quiet but
critical documents (like the IMS or the BOM). ARIA replaces probabilistic
coverage with a **deterministic manifest**: core baseline documents are analyzed
on every run *by construction*, and each analysis retrieves via
**filename-filtered retrieval** from exactly the document it claims to be
reading. Retrieval only *discovers* extra documents; it never decides coverage.

That change is what makes ARIA behave like an auditor instead of a summarizer.
In testing it has, unprompted:

- caught that a battery supplier change request assumed lithium-ion chemistry
  while the Bill of Materials baseline specifies Nickel-Cadmium — and opened a
  Critical-scored risk with a 15-day action item gating CCB approval;
- caught the Program Overview and Integrated Master Schedule stating conflicting
  First Flight dates, flagged the baseline inconsistency, and directed formal
  correction;
- responded to a $55–174M weight-reduction proposal by finding a $12M
  extended-range fuel tank modification already in the program's mod pipeline
  and making "rule out the cheaper alternative first" the mandatory first gate.

## The testbed

Eurus Systems' **MAAP-1** — a fictional tandem-rotor heavy-lift autonomous
helicopter with three mission variants (cargo, firefighting, ISR). Its baseline
is a fully synthetic, interlinked controlled-document set (`data/`):
requirements baselines, IMS, risk register, BOM, FFP financials, manufacturing
ramp, supply chain, drawing trees, TEMP, quality, safety, and security
documents — deliberately seeded with the kinds of cross-document conflicts real
programs accumulate.

**Everything in this repository is fictional and synthetic. No real program
data, suppliers, or figures are used.**

## Product design

- **Impact Console** (React): live agent telemetry, documents lighting up as
  assessed, the PMP drafting itself with rendered tables via Server-Sent Events.
- **Record/replay demos**: real runs are recorded as timestamped event streams
  and replayed with authentic pacing at zero inference cost — honestly labeled
  as replays.
- **Gated live inference**: custom analyses require a server-enforced access
  code; combined with API spend caps and CORS restricted to production origins,
  worst-case cost is a number chosen in advance.

## Tech stack

Python · FastAPI · Server-Sent Events · LlamaIndex · sentence-transformer
embeddings · Claude Sonnet 4.6 (Anthropic API) · React (Vite) · react-markdown
· Render (API) · Vercel (frontend)

## Repository layout

```
api.py                     FastAPI server: SSE streaming, access-code gate
record_demo.py             Records live runs as replayable demo JSON
data/                      The 19-document synthetic MAAP-1 baseline
frontend/                  React site + Impact Console + recorded demos
scripts/data_generation/   One-time scripts that generated the baseline
src/                       Agents, RAG system, pipeline event generator
```

## Running locally

Requires Python 3.11, Node 18+, and your own Anthropic API key.

```bash
# backend
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
echo "ANTHROPIC_API_KEY=your-key-here" > .env   # never committed (.gitignore)
uvicorn api:app --port 8000

# frontend (second terminal)
cd frontend && npm install && npm run dev
```

If `LIVE_ACCESS_CODE` is unset, live runs are open (dev mode); set it to
require an access code, as production does. Secrets live only in `.env`
locally and in encrypted host environment variables in production — nothing
sensitive is committed to this repository.

## Roadmap

Inline citations to source passages · what-if comparison runs · native .docx
export · computed EVMS/Monte Carlo hooks · scored evaluation harness ·
multi-program support.
