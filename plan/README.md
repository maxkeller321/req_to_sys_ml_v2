# Programme plan: Polarion -> SysML v2 -> Cameo

A plan to build this from nothing, between 21 October 2026 and 16 April 2027,
with one developer full time.

**Starting point: a blank repository.** No code, no pipeline, no connections.
Every sprint below budgets for building the thing, not for extending
something that already exists.

| File | What it is for |
|---|---|
| [sprint-timeline.md](sprint-timeline.md) | **The working sequence**: week-by-week workstreams and dates. Read this first. |
| [sprint-0.md](sprint-0.md) | What to do in the next three days. |
| [ROADMAP.md](ROADMAP.md) | Acceptance criteria per workstream, the eight gates and the scope ladder |
| [cameo-publish.md](cameo-publish.md) | The one genuinely unsolved problem, five candidate routes, and how we choose |
| [polarion-ingest.md](polarion-ingest.md) | The ingest side: auth, paging, caching, link roles, mapping |
| [risks.md](risks.md) | Risk register: trigger, mitigation, declared fallback |
| [question-1-architecture.html](question-1-architecture.html) | **Open decision**: typed IR in the middle, or retrieval instead? Interactive — drag the slider |
| [question-2-performance-vs-integration.html](question-2-performance-vs-integration.html) | **Open decision**: how the 23.7 weeks split between extraction quality and the six interfaces |
| [decisions/](decisions/) | Architecture decision records, newest last |

## What this plan is not

Two files cover the schedule and they have different jobs:
**sprint-timeline.md** owns the sequence and the dates;
**ROADMAP.md** owns the acceptance criteria, the gates and the scope ladder.
Where they disagree on a date, the timeline wins.

It is not a design document. The architecture it builds towards is in
[PROJECT_PLAN.md](../PROJECT_PLAN.md), the LLM step in
[EXTRACT_PLAN.md](../EXTRACT_PLAN.md), and the IR scope boundary in
[VERSION_MANAGEMENT.md](../VERSION_MANAGEMENT.md). Read those for *what*;
read this folder for *when, in what order, and what we drop if we run out of
weeks*.

## Where the estimates come from

Several technical choices in here are already de-risked by prior
investigation rather than assumed, which is why some sprints look
unusually specific. The ones that matter:

- **The validator works and is free.** `opensysml` (PyPI) downloads and runs
  its own `sysml-grpc` service; a generated model can be parsed with
  structured, located diagnostics. No Docker, no Xtext build. See
  [decisions/0001-sysml-v2-toolchain.md](decisions/0001-sysml-v2-toolchain.md).
- **Six units in the obvious ISQ/SI whitelist do not exist** in the SysML v2
  standard library (`ms`, `kPa`, `MPa`, `mV`, `kV`, `mA`), and a plausible
  workaround parses cleanly while being semantically wrong. Budgeted in S2
  rather than discovered in S9.
- **A model can be exported as OMG API JSON today** - the element form a
  SysML v2 API server ingests. That is the first half of one Cameo route and
  it is known to work, which is why [cameo-publish.md](cameo-publish.md)
  ranks that route the way it does.
- **Wildcard library imports shadow short root-package names**, so qualified
  references need the global-scope prefix. A day's debugging, budgeted in S2.

Treat these as spike results that have already been paid for. Everything else
is estimated from scratch.

## The assumptions this plan is built on

Change any of these and the plan needs re-cutting, not patching.

| Assumption | Status | If it breaks |
|---|---|---|
| Deadline **16 April 2027** | Confirmed. The original "March 2026" was five months in the past. | Re-cut sprint boundaries |
| **One developer, full time** | Confirmed | Scope ladder in ROADMAP.md decides what goes |
| Blank repository, nothing built | Confirmed | - |
| Polarion API access exists today | Confirmed | S3 becomes an access sprint; everything after slips two weeks |
| Cameo licence and a real `.mdzip` available | Confirmed | The Cameo half cannot start; risk R1 |
| LLM provider decided | Confirmed | Extraction work blocks; risk R6 |
| Requirements for the golden set **not yet picked** | Confirmed *not* done | First item in [sprint-0.md](sprint-0.md) |

## The hard truth about capacity

26 calendar weeks, minus the Christmas fortnight and two Easter days, is
**about 23.7 engineer-weeks**. Building the pipeline from zero, ingesting Polarion,
publishing into Cameo, and getting extraction quality good enough to trust
does not fit in 23.7 weeks for one person.

So the plan commits to a scope *ladder*, not a scope. ROADMAP.md names what
is MUST, what is SHOULD, and the exact order things get dropped. Making that
call now is the difference between a plan and a wish.

## Two decisions that are still open

Both are in this folder as interactive pages — open them in a browser and drag
the slider; each position spells out what it buys and what it costs.

1. **Architecture** — `Requirement -> IR -> SysML v2` against
   `Requirement -> RAG -> SysML v2`. Not actually either/or: the plan already
   uses retrieval for few-shot selection and for entity grounding, so the real
   question is *where the IR's authority ends*.
2. **Performance against integration** — the task is an open research
   question and nothing here reaches 98 % correct, so optimising toward it has
   no stopping rule. The reachable bar is **never silently wrong**, and once
   that floor holds the remaining weeks are better spent on reach.

Decide 1 before 2: architecture determines where effort can usefully go. Both
feed the scope ladder in [ROADMAP.md](ROADMAP.md).
