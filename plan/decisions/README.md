# Decision records

One file per decision, numbered, newest last. Append; never rewrite history -
if a decision is reversed, add a new record that supersedes the old one and
say why.

```
NNNN-short-slug.md
```

Each record:

```markdown
# NNNN. Title

- **Date:**
- **Status:** proposed | accepted | superseded by NNNN
- **Context:** what forced a decision
- **Decision:** what we are doing
- **Consequences:** what this costs, and what it rules out
- **Alternatives:** what was rejected, and why
```

Write one at **every gate** in [../ROADMAP.md](../ROADMAP.md). The gates are
where the plan is allowed to change, and a changed plan with no written
reason is indistinguishable from drift.

| | Decision | Status |
|---|---|---|
| 0000 | Programme constraints: deadline, staffing, scope ladder | to write in S0 |
| 0001 | SysML v2 toolchain: opensysml | accepted |
| 0002 | Cameo publish route | **due at G1, 6 Nov 2026** |
| 0003 | Experiment tracking: MLflow, file-backed | accepted; revisit at G4 |
