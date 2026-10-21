# 0001. SysML v2 toolchain: opensysml

- **Date:** 21 October 2026
- **Status:** accepted

## Context

The pipeline generates SysML v2 textual notation. Generated notation that
nobody checks is worthless - the generator will be subtly wrong, and the only
question is whether that is discovered in CI or by a modeller in March. So a
validator is a MUST item, not a nice-to-have, and it has to be in CI from S2.

The obvious candidate was the OMG SysML v2 Pilot Implementation: Xtext-based,
JVM, usually run as a Docker container. That is a real operational cost for a
one-developer programme - a container to build, maintain and run in CI.

## Decision

Use **`opensysml`** (PyPI) as both validator and view renderer.

It downloads and manages its own `sysml-grpc` service, so there is nothing to
build and no Docker in the critical path. `OPENSYSML_BINARY` points at a local
binary where the download is blocked, and `OPENSYSML_SERVICE` points at an
externally managed service.

Pin the version for the programme.

## Consequences

Good:

- `parse_sources` loads all generated files together, so cross-file
  references resolve and each diagnostic names its own file, line and column.
  Structured diagnostics are what make a repair loop possible later.
- Validation runs in CI with no infrastructure.
- It also renders: a declared view or document can be turned into Markdown
  and a standalone HTML page, with diagrams laid out by the service. That
  covers "let a human look at the model" without a layout engine.
- `migrate` reads SysML v1 (`xmi`, `uml`, `mdzip`), which is the Cameo ->
  pipeline direction if grounding is ever wanted.
- A model exports as OMG API JSON, which is the first half of Cameo publish
  route (a).

Costs and limits:

- It is a young project. Pin the version and keep the hand-built-IR
  validation test as a canary. Risk R8.
- The API JSON and RDF mappings are marked **experimental**.
- There is **no SysML v1 output**, which is exactly why Cameo publish is an
  open question. See [../cameo-publish.md](../cameo-publish.md).
- A clean parse is not a correct model: at least one plausible construct
  parses with zero diagnostics and is semantically wrong. Where semantics
  matter, evaluate as well as parse.

## Alternatives

- **OMG Pilot Implementation in Docker.** Reference-grade, but a container to
  own. Keep as the fallback if `opensysml` proves unstable.
- **No validator, review by eye.** Rejected. The generator cannot be trusted
  without one, and a repair loop needs located diagnostics to feed back.
