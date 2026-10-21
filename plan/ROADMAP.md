# Roadmap: 21 Oct 2026 -> 16 Apr 2027

One developer, full time, from a blank repository. **23.7 engineer-weeks** of
real capacity.

> **Sequence revised 21 Oct 2026.** Cameo was cut from two sprints to a single
> week late in the programme, and dataset generation, the metrics harness and
> extraction were all moved ahead of Polarion ingest. Extraction needs only
> requirement text, which the dataset already supplies, so the programme's
> riskiest question is answered in January rather than February. **Model
> update** was added, because generation without adaptation only solves half
> the problem. The **repair loop** was then restored with two weeks including
> its own evaluation, and the end date moved from 31 March to **16 April**
> to pay for it rather than cutting something else. [sprint-timeline.md](sprint-timeline.md) is the
> authoritative week-by-week plan; the sprint list below is kept for its
> acceptance criteria, which are unchanged. Gate dates below are realigned to
> the new sequence.

## Scope ladder

One person cannot build the pipeline, both integrations and extraction
quality in 23.7 weeks. This is the order things go, decided now:

| | Scope | Why here |
|---|---|---|
| **MUST** | IR schema; render to `.sysml`; validate (syntax + semantics); Polarion ingest; LLM extraction; golden set + metrics; traceability report | Without these there is no product. Nothing in this row may be dropped without moving the deadline. |
| **SHOULD** | Architecture consistency rules; repair loop + its evaluation; entity registry grounded in Polarion; model update (stable identity + change diff) | Grounding makes the model right at scale. Model update is the "adaptation" half the role asks for. The repair loop is the only agentic element and the only consumer of the diagnostics the rest of the plan produces. |
| **COULD** | Cameo publish (one-way); human-review interrupt; prompt/few-shot tuning depth | Cameo's route is unknown until the W2 spike, and a manual import is acceptable. |

**Dropped first, in this order:**

1. **Cameo publish** -> write the files, import into Cameo by hand. Saves 1 week.
2. **Repair evaluation, not the loop** -> ship the loop with a retry cap of 1
   and no before/after study. Saves 1 week, and you no longer know whether
   repair helps or harms. Uncomfortable, which is why it is not first.
3. **Model update** -> regenerate every time, no diff report. Saves 1 week,
   and gives up the "adaptation" half of what the role asks for.
4. **Dataset size** -> 20 requirements instead of 50. Saves 1 week, and costs
   confidence in every metric.
5. **Registry automation** -> registry stays hand-written. Saves 2 weeks, but
   drop this *only* after item 3: unstable entity naming makes diffs
   worthless, so cutting grounding while keeping model update is incoherent.

Drop from the top. If all five go and it still does not fit, the deadline
moves again - that is the honest answer, and it is better said in November than in
March.

## Gates

A gate is a date where the plan is allowed to change. Write the answer into
[decisions/](decisions/) on the day.

| Gate | Date | Week | Question | If the answer is bad |
|---|---|---|---|---|
| **G1** | Fri 6 Nov 2026 | W3 | Which Cameo route, proven by a minimal manual import? | Cameo drops to a documented manual import. Decided in November, not March. |
| **G2** | Fri 27 Nov 2026 | W6 | Does a hand-built IR render to `.sysml` and validate with zero diagnostics, **and do the consistency rules reject a deliberately broken one?** | The IR schema is wrong. Stop and fix it before the dataset is written against it. If only the rules are missing, they slip to W24-25 - layer 2 is a MUST, layer 3 a SHOULD. |
| **G3** | Fri 18 Dec 2026 | W9 | Is the dataset complete, reviewed, and does every entry validate? | Cut the dataset to 20 requirements rather than shipping unreviewed expected answers. |
| **G4** | Fri 22 Jan 2027 | W14 | **How hard are our requirements actually?** The first extraction baseline, measured by the W12 harness. | The most important gate in the plan. A poor baseline against a good registry means the problem is prompting or schema; a good one means the effort belongs in entity resolution. Re-plan W15-21 on the answer. |
| **G5** | Fri 5 Feb 2027 | W16 | **Does repair help, and does it ever harm?** Pass rate and subject accuracy, before and after, against the expected IR. | If repair raises pass rate but degrades accuracy, set the retry cap to zero. That is the pre-repair behaviour, so the fallback costs nothing and the evaluation still earned its week. |
| **G6** | Fri 26 Feb 2027 | W19 | Do real Polarion work items reach a validated model, with no LLM in the loop? | Cut the Polarion field scope, not the schedule. Re-verify dataset and extractor evidence against ingested text here. |
| **G7** | Fri 12 Mar 2027 | W21 | Does grounding beat the W14 baseline, **and is entity naming stable between runs?** | Stability matters as much as the score: W22 diffs two runs against each other, so unstable naming makes it worthless. If naming is unstable, spend W22 fixing that instead of building the diff. |
| **G8** | Fri 26 Mar 2027 | W23 | Feature complete, including whatever Cameo export survived G1? | Ship what works; the pilot shrinks to a demo. |

## Per-workstream acceptance criteria

Dates and sequence live in [sprint-timeline.md](sprint-timeline.md); this
section is only the bar each workstream has to clear. Status column: fill in
as you go.

| Workstream | Weeks | Done when | Status |
|---|---|---|---|
| Kickoff and scope | W1 | Requirement set named in writing; MBSE engineer booked; Cameo checklist ready for W2 | |
| Intermediate representation | W2-3 | MBSE engineer has signed off the schema **and** the consistency rule list | |
| SysML transcription + consistency | W4-6 | Hand-built IR: zero diagnostics in CI. A broken IR is rejected with a named reason per problem | |
| Dataset generation | W7-9 | 30-50 requirements with reviewed expected IR, all validating, usable as a fixture | |
| Experiment tracking | W10-11 | A run logs params, metrics and artifacts to `./mlruns`; `mlflow ui` shows it | |
| Metrics and evaluation | W12 | Scores the dataset against itself at 100%, a corrupted copy lower; re-scores from stored traces without calling the LLM | |
| Extraction | W13-14 | Dataset runs end to end without hand-editing; the harness prints a baseline | |
| Repair loop + evaluation | W15-16 | Pass rate and iterations measured before/after; cap never silently exceeded; **no requirement worse after repair than before** | |
| Polarion ingest | W17-19 | Real work items reach a validated model, no LLM; `Trace.revision` populated; dataset evidence re-verified | |
| Grounding | W20-21 | Resolution rate at target; no duplicate part def for a known entity; identical names across two runs | |
| Model update | W22 | Twice on unchanged input -> empty diff; one changed requirement -> a diff naming exactly it | |
| Cameo export | W23 | Generated content opens in Cameo, correctly typed, Polarion id attached | |
| Hardening and pilot | W24-25 | A full run on the real project completes; someone else has used it; failure modes handled | |
| Buffer and handover | W26 | Documentation and known limitations written down | |

## Capacity arithmetic

See [sprint-timeline.md](sprint-timeline.md) for the week-by-week table.
**23.7 engineer-weeks** across 26 calendar weeks.

Against that: MUST items are roughly 13-15 weeks from zero, SHOULD another
6-8, COULD another 3-5. The ladder is not decoration.
