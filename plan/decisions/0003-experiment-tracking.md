# 0003. Experiment tracking: MLflow, file-backed

- **Date:** 21 October 2026
- **Status:** accepted

## Context

From W13 onwards the core loop is: change a prompt or the registry, re-score
the dataset, decide whether it helped. Done without tracking, every one of
those decisions rests on a number nobody can reproduce or attribute, and
W18-19 — the sprint where the quality problem actually gets solved — becomes a
sequence of slow, expensive, unrepeatable experiments.

[PROJECT_PLAN.md](../../PROJECT_PLAN.md) names LangSmith or Langfuse for
"tracing and evaluation from day one". Neither fits well:

- **LangSmith** has the best developer experience — it is first-party for
  LangChain and LangGraph. It is also SaaS. If requirement confidentiality
  drove the LLM provider decision, requirement text cannot go to LangSmith
  either; it is the same data leaving the building, to a second vendor.
- **Langfuse** self-hosts, but that means Docker plus Postgres plus
  Clickhouse to own and keep running. Real operational weight for one
  developer with 21.5 engineer-weeks.

There is also a shape mismatch. Langfuse and LangSmith are built around
**trace inspection** — look at one call and see what it did. This programme's
loop is **experiment comparison** — did run B score better than run A, and
because of what.

## Decision

Use **MLflow**, file-backed, with no server:

```python
mlflow.set_tracking_uri("file:./mlruns")
mlflow.langchain.autolog()        # also mlflow.anthropic.autolog()
```

Per run, log:

| | What |
|---|---|
| params | model id, prompt hash, dataset version, registry version, git SHA |
| metrics | subject accuracy, attribute and constraint precision/recall, `modelable` correctness, validation pass rate |
| artifacts | the generated `.sysml` files, `report.md`, the diagnostics |
| traces | every LLM exchange, captured by autolog |

Set up in **W10-11** (the Christmas fortnight), because it is self-contained,
needs no LLM and no service, and it has to exist before the first numbers do.
Tracking added after the fact means re-running everything.

Verified against MLflow **3.17.2**: `mlflow.langchain.autolog`,
`mlflow.anthropic.autolog`, `mlflow.trace`, `mlflow.search_traces` and
`mlflow.evaluate` are all present. Pin the version.

## Consequences

Good:

- **No cloud, no server, no database.** One install and a directory, so the
  confidentiality question does not arise.
- **The run manifest comes free.** An earlier draft of this plan had us
  hand-rolling git SHA plus prompt hash plus scores into a JSON file; that is
  precisely MLflow's params/metrics model. Same effort, plus a UI and run
  comparison.
- **One line of tracing.** `autolog` captures the LangChain and LangGraph
  invocations, so no instrumentation pass is needed.
- **Re-scoring is free.** `search_traces` reads the stored responses back, so
  changing the *metric* and re-scoring an old run costs no LLM calls, no money
  and introduces no non-determinism. This is the single biggest practical win
  and the reason it is worth a day.
- **A separate response cache is not needed.** Traces serve that purpose, so
  there is one mechanism rather than two.

Costs and limits:

- `mlflow` is a heavy dependency for what we use of it.
- Its LLM ergonomics are thinner than Langfuse's: no prompt registry
  workflow, no cost dashboard tuned for LLM apps.
- LangGraph tracing through MLflow is less mature than LangSmith's
  first-party support. If graph-level traces turn out to be unreadable, that
  is the trigger below.
- `./mlruns` grows. Keep it out of version control and prune deliberately.

## Revisit at G4 (22 Jan 2027)

G4 is the first extraction baseline, and the point where we know whether
extraction quality is the dominant problem.

- **If the baseline is poor** and the weeks ahead are prompt iteration, a
  Langfuse instance starts earning its keep — prompt management and
  side-by-side trace diffing are genuinely better there. Budget a week and
  take it from the scope ladder.
- **If the baseline is decent** and entity resolution is the real issue, stay
  on MLflow and never look back.

Record the answer as a new ADR either way.

## Note for whoever implements it

The `mlflow` install ships its own instrumentation guide as a skill, at
`mlflow/assistant/skills/instrumenting-with-mlflow-tracing/SKILL.md` inside
the installed package. Read it before writing tracing code — it covers what is
worth tracing and how to verify that traces are actually landing, which is the
part that is easy to get wrong and not notice.

## Alternatives

- **Hand-rolled JSONL plus a run manifest.** Roughly the same day of work and
  no UI, no comparison, no autolog. Rejected once MLflow was on the table.
- **Langfuse self-hosted.** Better LLM ergonomics, materially more operational
  weight. Deferred to the G4 trigger above.
- **LangSmith.** Best DX, SaaS only at our tier. Likely excluded by the
  confidentiality decision — confirm before anyone budgets for it.
- **Nothing.** Rejected. The W18-19 loop is not tractable without it, and
  "it got better" is not a finding.
