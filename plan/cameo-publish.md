# Getting generated content into Cameo

This is the only part of the programme with no known solution. It gets
decided at **G1, Friday 6 November 2026**, because every route has a
different cost and one of them does not fit the window at all.

## The asymmetry

Cameo Systems Modeler / CATIA Magic is a **SysML v1.x** tool: `.mdzip`,
UML/XMI, profiles and stereotypes. The pipeline produces **SysML v2**. Those
are different metamodels, not different file extensions.

The direction we do *not* need is the one that is solved:

```
Cameo .mdzip  --[ opensysml.migrate(), experimental, ledgered ]-->  SysML v2
SysML v2      --[ nothing                                     ]-->  Cameo
```

`opensysml` reads `xmi`, `uml` and `mdzip` as input. It writes `sysml`,
`kerml`, `text`, `ttl`/`rdf` and `api-json`. There is **no v1 output format**.
So publishing into Cameo means leaving the v2 toolchain at some point, and
the question is where.

## The five candidate routes

Ranked by expected cost. Spike them in this order and stop at the first that
works.

### (a) SysML v2 API & Services server, read by Cameo

Stand up the OMG standard REST API in front of the model; have Cameo connect
to it.

**Half of this is already known to work.** A parsed model exports as OMG API
JSON - the element form the standard's API serves - and on a small pilot
model that is ~1,500 element objects with `@type`, `@id`, `elementId` and
`ownedRelationship`. Producing the payload is not the risk.

- **Cost if it works:** 2-3 weeks.
- **Unknown:** whether your Cameo version can consume a SysML v2 API server
  at all, and whether it needs a plugin to do so.
- **Caveat:** the API JSON mapping is marked *experimental* by `opensysml`.
  Pin the version and keep a conformance test.
- **Spike (half a day):** does Cameo offer any "connect to SysML v2 API"
  affordance? If yes, point it at the reference implementation with a
  three-element model.

### (b) Cameo's own SysML v2 importer

If your licence includes a SysML v2 import plugin, feed it the `.sysml` text
the pipeline already produces and stop there.

- **Cost if it works:** 1-2 weeks. By far the cheapest.
- **Unknown:** whether it exists in your version and licence, and how much
  of the notation it accepts. An importer that chokes on `metadata def` or
  `requirement` constraint bodies is not useful for us.
- **Spike (half a day):** check the plugin list in your installation, then
  hand-import a ten-line `.sysml` file with a package, a part def with one
  attribute, and one requirement.

### (c) Generate SysML 1.x XMI from the IR directly

Do not try to convert v2 to v1. Write a **second renderer off the IR** -
behind the same interface as the `.sysml` renderer - that emits SysML 1.x
XMI for Cameo import. The IR is small and we own it: packages, part defs,
attributes, requirements, satisfy, derive. That is a narrow mapping.

This is architecturally the cleanest fit: the IR is the contract, and a
Cameo exporter is just another consumer of it. It is also fully offline and
unit-testable, with no service and no licence dependency at generation time.

- **Cost if it works:** 4-6 weeks.
- **Risk:** Cameo's import of foreign-generated XMI is sensitive to profile
  application and stereotype IDs. Getting a file Cameo *accepts* is the whole
  difficulty; getting one that is *well-formed* is easy. Do not estimate this
  without the spike.
- **Spike (one day):** export a trivial model **from** Cameo as XMI, read what
  it emits for a stereotyped requirement and a part property, then hand-edit
  a copy and re-import it. If a hand-edited round trip fails, this route is
  dead and the spike was cheap.

### (d) MTIP interchange

`opensysml` references MTIP exports (it can take one to lay out migrated
views), which places MTIP in Cameo's interchange orbit. If an MTIP plugin can
*import* as well as export, that is a documented path in.

- **Cost if it works:** 3-5 weeks.
- **Unknown:** almost everything. Rank it fourth because the evidence is one
  mention in a parameter description.
- **Spike (two hours):** does the MTIP plugin exist for your Cameo version,
  and does it import?

### (e) Custom MagicDraw/Cameo Java plugin

Write a plugin against the Cameo Java OpenAPI that reads our API JSON and
builds model elements.

- **Cost:** 8+ weeks, plus a Java toolchain and the Cameo SDK.
- **Verdict: out of scope for this programme.** One developer cannot add a
  Java plugin to a 23.7-week Python programme and still deliver the MUST
  items. If the spikes land here, take the fallback instead.

## Decision criteria at G1

Choose the **cheapest route with a demonstrated manual import**. "The
documentation says it should work" does not pass G1; a file or connection
that actually landed in Cameo does.

Record in `decisions/`: the route, what was demonstrated, what it cost, and
what was ruled out and why.

## Declared fallback

If no route is demonstrated by 6 November:

> Cameo publishing drops from SHOULD to COULD. The pipeline writes `.sysml`
> and API JSON to a directory; importing into Cameo is a documented manual
> step for the modeller. The programme delivers Polarion -> validated
> SysML v2 -> traceability report, and Cameo integration becomes a follow-on
> with its own budget.

This is a worse product. It is not a failed programme, and saying so in
November costs four weeks less than discovering it in February.

## What "published" has to mean

Whichever route wins, the acceptance criteria at G4 and S7 do not change:

1. **Typed, not text.** A requirement arrives as a requirement, not a note.
2. **Traceable.** The Polarion ID is on the Cameo element and survives.
3. **Idempotent.** Publishing twice does not duplicate. Always underestimated.
4. **Reviewable.** A modeller can tell generated content from hand-authored.
