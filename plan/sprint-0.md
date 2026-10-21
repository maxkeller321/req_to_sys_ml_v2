# Sprint 0: Wednesday 21 - Friday 23 October 2026

Three days. No production code. Everything here is a long-lead item, which is
why it comes before the keyboard work.

## Day 1 (Wed 21 Oct) - Name the requirements

The one confirmed-missing prerequisite. It blocks the golden set in S4, and
the golden set is what tells you whether any of this works.

- [ ] Agree the Polarion **project and space** in scope.
- [ ] Pick **20-50 representative work items**, deliberately covering:
      a performance limit, an interface, a range or tolerance, a negative
      ("shall not"), and at least three you expect to be **not modelable**.
      A set of only clean requirements will flatter the metrics and teach you
      nothing.
- [ ] Write the ids into `data/golden-set.md` with one line each on why it is
      in the set.
- [ ] Note the Polarion **version** and whether the API is REST or SOAP.
      This decides a week of S3.

## Day 2 (Thu 22 Oct) - Open the Cameo question

Read [cameo-publish.md](cameo-publish.md) first. The goal today is not a
solution, it is a **list of things to try**, so S1 can execute rather than
explore.

- [ ] Record the exact Cameo version and the installed plugin list.
- [ ] Route (b): is there a SysML v2 import plugin? If yes, this is the
      cheapest answer in the whole programme - try it immediately with a
      ten-line `.sysml` file.
- [ ] Route (a): does Cameo offer any "connect to SysML v2 API" affordance?
- [ ] Route (d): is an MTIP plugin available?
- [ ] Export a trivial model **from** Cameo as XMI and keep it. It is the
      reference for route (c) and costs two minutes now.

If route (b) works on day 2, say so loudly - it changes the plan in your
favour and frees four weeks.

## Day 3 (Fri 23 Oct) - Set up and write down

- [ ] Create the repository: `uv` project, package skeleton, a linter and a
      test runner in CI. Green on an empty test suite.
- [ ] `decisions/0000-programme-constraints.md`: the 16 April 2027 deadline,
      one developer full time, and the scope ladder. This is the document you
      point at when someone asks why Cameo is only one-way.
- [ ] Pin `opensysml` and record the version. See
      [decisions/0001-sysml-v2-toolchain.md](decisions/0001-sysml-v2-toolchain.md).
- [ ] Skim [VERSION_MANAGEMENT.md](../VERSION_MANAGEMENT.md) and book the
      time with an MBSE engineer for the S1 IR review. Their availability is
      the second-longest lead time after Cameo.

## Done when

- The requirement set is named and written down.
- S1 has a Cameo checklist it can execute without further research.
- CI is green on an empty repository.
- The deadline and staffing assumptions exist as an ADR.

## Explicitly not in Sprint 0

No IR schema, no renderer, no prompts. The temptation is to start coding
because coding feels like progress; the two things that can actually sink
this programme are the Cameo route and the requirement set, and both are
answered by reading and talking, not typing.
