# Sprint timeline

**21 Oct 2026 -> 16 Apr 2027.** 26 week-slots, one developer full time,
**~23.7 engineer-weeks** of real capacity (W1 is a part week, W10-11 are
Christmas, and Easter costs two days in W23-24).

This is the working sequence. [ROADMAP.md](ROADMAP.md) has the per-workstream
acceptance criteria and the scope ladder; where the two disagree on dates,
this file wins.

Cameo is deliberately one week, late. It is the least certain part and the
least valuable until the pipeline it feeds actually works.

```
W1  W2-3  W4-6        W7-9     W10-11   W12   W13-14   W15-16  W17-19    W20-21    W22  W23    W24-25  W26
|   IR    transcr+chk dataset  xmas+ML  eval  extract  repair  Polarion  registry  upd. Cameo  harden  buf
```

## The plan

| Weeks | Dates | Workstream | Done when |
|---|---|---|---|
| **W1** | 21 - 23 Oct 2026 | **Kickoff and scope** — pick the 20-50 requirements for the dataset; book the MBSE engineer | The requirement ids are written down and the MBSE time is in a calendar |
| **W2-3** | 26 Oct - 6 Nov | **Intermediate representation** — *define together with MBSE engineer*. Pydantic schema, cross-reference validation, V1/V2 scope boundary. **Same sessions: agree the architecture consistency rules** — "what makes a model inconsistent in your world?" costs nothing extra while they are already in the room | The MBSE engineer has signed off the schema and a written list of consistency rules. It is the contract between extraction and rendering; everything downstream breaks if it changes later |
| **W4-6** | 9 - 27 Nov | **SysML transcription and consistency checks** — IR -> `.sysml`, deterministic output, the validator wired into CI, **plus the architecture consistency rules** agreed in W2-3 | A hand-built IR renders and validates with **zero** diagnostics, in CI, no LLM involved. A deliberately inconsistent IR is rejected by the rules with a named reason for each problem |
| **W7-9** | 30 Nov - 18 Dec | **Dataset generation** — hand-written IR for the W1 requirements, covering limits, interfaces, ranges, negatives and non-modelables. Every entry rendered and validated as it is written | 30-50 requirements with reviewed expected IR, in the repo, all validating, usable as a test fixture |
| **W10-11** | 21 Dec - 1 Jan 2027 | *Christmas* — budget half a week. **Experiment tracking**: MLflow, file-backed, plus `mlflow.langchain.autolog()`. Self-contained, needs no LLM and no service, so it is the right kind of holiday work | A run logs its params (model, prompt hash, dataset version, git SHA), its metrics and its artifacts (the generated `.sysml` and report) to `./mlruns`, and `mlflow ui` shows it |
| **W12** | 4 - 8 Jan | **Metrics and evaluation** — subject accuracy, attribute and constraint precision/recall, `modelable` correctness, validation pass rate. Built against the dataset, before there is an extractor to flatter. Scores land as MLflow metrics | The harness scores the dataset against itself at 100%, and a deliberately corrupted copy below that. Re-scoring an earlier run from its stored traces works without calling the LLM |
| **W13-14** | 11 - 22 Jan | **Extraction** — one structured-output LLM call per requirement | The dataset runs end to end and produces IR without hand-editing. The W12 harness prints a baseline on the first run |
| **W15-16** | 25 Jan - 5 Feb | **Repair loop and its evaluation** — the one agentic element: a conditional edge back to extraction, fed the IR validation errors and consistency findings for *that one requirement*, retry-capped. The second week is evaluation: does it help, and does it ever harm? | Pass rate before and after repair, and repair iterations per requirement, both on the dataset. The cap is never silently exceeded. **No requirement scores worse after repair than before**, measured against the expected IR. Every repair decision logged to MLflow |
| **W17-19** | 8 - 26 Feb | **Polarion ingest** — auth, paging, caching, link roles, HTML stripping, work item -> IR mapping. **Populate `Trace.revision`**, which W22 needs as its change signal | Real work items produce a validated model with a hardcoded registry and no LLM. Every requirement carries its Polarion revision |
| **W20-21** | 1 - 12 Mar | **Entity registry and resolution** — seed from Polarion components or the product breakdown; merge "the pump" / "pump unit" / "P-101" | Subject resolution rate against an agreed target, no duplicate part def for an entity already known under another name, registry seeded from Polarion rather than hand-written, and two runs produce identical part-def names |
| **W22** | 15 - 19 Mar | **Model update** — stable element identity, change detection against the previous run, and a diff report. Not a full merge: see the scope note below | Running twice on unchanged input produces an **empty** diff. Changing one requirement in Polarion produces a diff naming exactly that requirement and the elements it touches |
| **W23** | 22 - 26 Mar | **Cameo export** — one-way, whichever route the W2 spike picked. Manual import is an acceptable outcome. *Karfreitag 26 Mar: four days* | Generated content opens in Cameo, correctly typed, with the Polarion id attached |
| **W24-25** | 29 Mar - 9 Apr | **Hardening and pilot** — full real project, failure modes, run it with an engineer watching. *Ostermontag 29 Mar: nine days* | A full run completes, the report is trustworthy, someone else has used it |
| **W26** | 12 - 16 Apr | **Buffer and handover** — documentation, known limitations | Deliberately empty. If it is already spent by February, use the scope ladder |

## Capacity

| | Weeks |
|---|---|
| W1 (3 days) | 0.6 |
| W2-W9 | 8.0 |
| W10-W11 (Christmas) | 0.5 |
| W12-W22 | 11.0 |
| W23 (Karfreitag) | 0.8 |
| W24-W25 (Ostermontag) | 1.8 |
| W26 | 1.0 |
| **Total** | **23.7** |

The programme was extended from 31 March to 16 April to pay for W15-16 rather
than cutting something for it. Easter eats most of what the third extra
calendar week would have given, so **+3 calendar weeks buys +2 engineer-weeks**.

## Dependencies worth knowing

- **W2-3 gates everything.** The IR is the contract. If the MBSE engineer is
  unavailable, this slips and the whole timeline slips with it — chase that
  booking in W1, not W2.
- **W4-6 before W7-9, and that is why the dataset moved.** Once transcription
  and the validator exist, every hand-written dataset entry can be rendered
  and validated as you write it. A misunderstanding of the schema shows up the
  same hour instead of in February, and the dataset doubles as the renderer's
  test suite.
- **W7-9, then W12, then W13-14.** Write the expected answers, then the thing
  that scores them, then the thing being scored. Build the harness while no
  extractor exists and you cannot quietly tune the metric to flatter the model
  — and the moment extraction produces its first output, the number is already
  there. No "we will measure it later".
- **Extraction and repair do not need Polarion, which is why they run ahead of
  it.** Both take requirement text and return IR; the dataset already supplies
  the text. Running them in January and early February answers the programme's
  riskiest question — *how hard are our requirements, actually* — while there
  is still time to react.
- **W1 blocks W7.** The requirement set is the only prerequisite not yet
  settled, and it is needed in late November.
- **W12 has nothing live to measure, on purpose.** The harness is tested
  against the dataset itself: perfect input must score perfectly, and a
  corrupted copy must score worse. That is how you find out the harness is
  right while the answer is still knowable.
- **W10-11 before W12-16.** Tracking has to exist before the first numbers do,
  or the first numbers are unattributable — and the repair evaluation in W16
  compares runs, which is exactly what tracking is for.
- **W20-21 before W22, and this one is not negotiable.** Model update diffs
  two runs against each other. If entity resolution is unstable — "the pump"
  resolving to `PumpUnit` one run and `Pump` the next — every diff reads as a
  delete plus a create and the feature is worthless.
- **W23 is late on purpose.** Cameo consumes the pipeline's output, so it can
  only be built once there is output worth consuming.

## The repair loop: what it repairs, and what it must not

This is the only agentic element in the programme. Without it the pipeline is
strictly linear — one call per requirement, no conditional edges, no feedback,
no retry.

### It targets the IR, not the SysML

[PROJECT_PLAN.md](../PROJECT_PLAN.md) draws the loop as
`Validate (SysML) -> Extract`. In the architecture this plan builds, that is
the wrong target:

```
PROJECT_PLAN:  Validate(SysML) --errors--> Extract        <- wrong
this plan:     IR validation + consistency rules --> Extract
```

The LLM never writes SysML — it fills the IR, and deterministic code renders.
So a SysML diagnostic is almost always a **renderer bug**, not a model error.
Feeding those back to the LLM would paper over renderer defects instead of
fixing them. SysML diagnostics must fail loudly in CI; they must never trigger
a retry.

The loop's real inputs are the layer-1 and layer-3 findings below: unknown
subject, attribute not on the subject, evidence not verbatim, constraint unit
not commensurable with the attribute, requirement with no satisfy.

### The canonical case

An unresolved subject. `"main pump"` against a hand-written registry yields
subject `None`, the constraint dropped, and three review flags. That is a
perfect repair trigger: *"you said 'main pump'; here are the known entities;
pick one or justify a new one."* It is also the error class grounding alone
will not fully close, because new phrasings keep arriving.

### The evaluation is half the work, and the harder half

A repair loop can **raise the pass rate while lowering quality**. Asked to fix
"unknown subject", a model will happily pick the nearest entity just to make
the error go away — and a confidently wrong subject is worse than a flagged
failure, because nobody reviews it.

So W16 measures against the **expected IR**, not against the validator:

- pass rate before and after repair,
- repair iterations per requirement, and that the cap is never silently
  exceeded,
- **and that no requirement scores worse after repair than before.**

That third one is the gate. If repair improves pass rate but degrades subject
accuracy, it is doing harm and the cap goes to zero — which is exactly the
pre-repair behaviour, so the fallback is free.

### Interaction with grounding (W20-21)

Repair runs before grounding, so some of what it repairs is damage a seeded
registry would have prevented. That is useful rather than wasteful: the
repair-iteration count becomes a second, independent measure of grounding's
effect. If W20-21 works, repairs per run should fall.

## Validation: five layers, and the one that was missing

The programme needs "Validierungs- und Konsistenzprüfungen" in five distinct
senses. Four were already in the plan; the third was not, and it is the one
the role actually names.

| Layer | Question it answers | Where |
|---|---|---|
| 1. IR schema and references | Is the IR internally coherent? | W2-3 |
| 2. SysML syntax and semantics | Does the generated notation load? | W4-6 |
| **3. Architecture consistency** | **Is the model a sane architecture?** | **W4-6** |
| 4. Extraction correctness | Did the LLM get it right? | W7-9, W12 |
| 5. Update consistency | Does re-running change nothing? | W22 |

Layers 1 and 3 are also what the repair loop consumes.

### Why layer 3 cannot be left to the tools

Checked against a deliberately broken IR, all four of these pass both
layer 1 and layer 2 today:

- `mass <= 25 [s]` — seconds compared against a mass. The IR validates the
  unit against the *attribute's* quantity, but never the **constraint's** unit
  against it. OpenSysML only raises a *warning*, and `ValidationResult.ok`
  counts errors, so the model passes.
- A part definition that is declared and never used in any system tree.
- A requirement with no `satisfy` at all.
- A derivation cycle, `r1 -> r2 -> r1`.

None of these is a syntax error. All of them are wrong.

### Initial rule set

Deterministic Python over the IR — no LLM, no service, no network. Agree the
list with the MBSE engineer in W2-3; this is the starting point:

1. **Coverage.** Every in-scope work item is either modelled or recorded as
   not-modelable with a reason. Nothing silently absent.
2. **Satisfaction.** Every modelled requirement has at least one `satisfy`.
3. **Commensurability.** A constraint's unit matches the quantity of the
   attribute it constrains. This is the hole demonstrated above.
4. **No orphan definitions.** Every part def appears in some system tree.
5. **Reachability.** Every part usage reaches a root.
6. **Acyclicity.** No derivation cycles, no package or usage parent cycles.
7. **Purposeful attributes.** Every declared attribute is either constrained
   or carries a stated value — otherwise, why is it in the model?
8. **Severity policy.** Decide which OpenSysML *warnings* count as errors for
   us. Incommensurable quantities is the obvious first one: a warning there
   means a wrong model, not a stylistic preference.

Findings land in the traceability report beside the diagnostics, attributed to
the work item that caused them.

### What this costs, honestly

W4-6 was three weeks for renderer, validator and the three known notation
traps. The rule module is roughly three more days, so the sprint is tight.

If W4-6 runs long, **the consistency rules are the part that slips to
W24-25**, not the SysML validator — layer 2 is a MUST, layer 3 a SHOULD. But
try hard not to let it slip: the rules pay for themselves immediately in
W7-9, where fifty hand-written IRs get checked as they are authored, and the
repair loop in W15-16 consumes their findings.

## Model update: what is in W22 and what is not

The role this programme serves asks for generation **and adaptation** of
models. Full adaptation — merging generated content into a model a modeller
has since edited by hand, without clobbering their work — is 3-4 weeks and
does not fit. W22 buys the foundation it rests on, in one week:

**In scope (W22):**

- **Stable element identity.** Derived from the Polarion ID, which the IR
  already carries on every requirement and on traced attributes. The same work
  item must yield the same element name on every run, forever.
- **Change detection.** `ir.json` is already persisted per run, so the
  previous IR is the baseline. Diff old against new.
- **A diff report.** Added, changed and removed requirements and elements,
  with the Polarion ids that caused each change.
- **Idempotence, proven.** Re-running on unchanged input produces an empty
  diff. That one test is the whole acceptance criterion, and it is the thing
  that is quietly false in most generators.

**Out of scope (phase 2):**

- Merging into a hand-edited model while preserving the hand edits. This is
  the genuinely hard part and needs a decision about who owns which element.
- Writing updates back into Polarion or Cameo. One-way out only.
- Conflict resolution when a human and the generator changed the same element.

Two identity caveats found by inspecting a generated IR:

- `Trace.revision` is currently never populated. Ingest has to set it in
  W17-19, or change detection falls back to diffing full requirement text,
  which is noisier and slower.
- **Part definitions carry no Polarion trace** — they come from the registry,
  so their identity is their registry name. Rename a registry entry and the
  diff reads as delete-plus-create. Registry names therefore become an
  interface, not an implementation detail: changing one is a migration.

## Why experiment tracking is not optional

The core loop from W13 onwards is: change a prompt, a repair policy or the
registry, re-score the dataset, decide whether it helped. Without tracking,
every one of those decisions rests on a number nobody can reproduce or
attribute.

Concretely, it buys three things:

1. **Attribution.** Every score is tied to a model id, a prompt hash, a
   dataset version and a git SHA. "It got better" becomes "it got better
   because of this change".
2. **Free re-scoring.** Traces store the raw responses, so changing the
   *metric* and re-scoring last week's run costs nothing — no LLM calls, no
   money, no non-determinism. Without it, W15-16 and W20-21 are sequences of
   slow, expensive, unrepeatable experiments.
3. **Comparison.** "Did repair help?" and "did grounding help?" are two-run
   diffs rather than something you write yourself.

MLflow file-backed is roughly a day and needs no server. See
[decisions/0003-experiment-tracking.md](decisions/0003-experiment-tracking.md)
for why not Langfuse or LangSmith, and the trigger for revisiting that.

## The trap this ordering creates

Polarion ingest lands in **W17-19**, after the dataset (W7-9), extraction
(W13-14) and repair (W15-16). So three workstreams are built against
requirement text **exported by hand**, and extraction evidence has to be a
verbatim substring of the text the pipeline produces. If ingest later strips
HTML even slightly differently, every evidence string in the dataset stops
matching *and* the extractor's evidence check starts failing on real data.

That makes the first mitigation below non-optional rather than merely sensible:

1. **Write the text normalisation in W7, day one.** The HTML-stripping
   function comes before the dataset, and the hand-exported texts are produced
   *through it*. A day's work, and it locks the evidence contract before three
   workstreams depend on it.
2. **Re-verify in W17-19.** When real ingest lands, assert that every dataset
   evidence string is still a verbatim substring of the ingested text, and
   re-run the extractor against ingested rather than exported text. Either it
   passes or it tells you immediately.

Skip step 1 and the failure is silent: the dataset and the extractor quietly
disagree with the pipeline, and the metrics keep reporting numbers.

## Parallel where possible

One developer means little true parallelism, but three things can be
progressed in the gaps while waiting on other people:

- The **Cameo route spike** — a few hours in W2, so W23 is execution rather
  than research. See [cameo-publish.md](cameo-publish.md).
- The **Polarion field survey** — dump one work item's full field set early
  and read it with an engineer. Requirement text hiding in a custom field is
  the classic silent data loss.
- **Link role configuration** — read it from the installation rather than
  guessing which roles mean "derive".

## What is not in here

Human-review interrupt, prompt and few-shot tuning depth, full model
adaptation (merging into hand-edited models), and anything in the V2 IR scope
(behaviour, verification, interfaces). They are real work and they do not fit.
See the scope ladder in [ROADMAP.md](ROADMAP.md) for the order things get cut
if these weeks turn out to be optimistic.

Also still outside the programme, from the wider role description: agentic
generation of **architecture alternatives**, PowerPoint and Word document
generation, SharePoint integration, cloud deployment with SSO, and UI/UX.
Roughly 14-25 further weeks — a phase 2, not a squeeze.
