# Polarion ingest

Built in **S3 (23 Nov - 4 Dec 2026)**, gated by **G3**. API access is
confirmed to exist, which removes the longest lead time on this side.

## Decide first

| Question | Why it matters | When |
|---|---|---|
| Polarion version, and REST or SOAP? | REST (Polarion 21+) is pleasant; older installations mean SOAP and roughly a week more. | S0 |
| Auth: personal access token or service account? | A token tied to one person breaks when they leave. Prefer a service account with read-only scope. | S0 |
| Which project and space are in scope? | Bounds the ingest and names the golden set. Still open - first item in [sprint-0.md](sprint-0.md). | S0 |
| Which link roles mean "derive"? | Polarion link roles are configured per installation. `derived_from`, `refines`, `parent` are guesses until someone reads the configuration. | S3 |
| Which custom fields carry real content? | Installations put requirement text, rationale or verification method in custom fields. Missing one silently loses content. | S3 |

## What to pull

- **Work items**: id, type, title, description, status, priority, revision,
  and the custom fields identified above.
- **Links**: role plus target id, both directions. The role vocabulary is
  installation-specific; read it from the configuration rather than hardcoding.
- **Document and space structure**: becomes the package tree, so chapter and
  section ordering must be preserved.
- **Component work items, product breakdown, glossary** if they exist. These
  seed the entity registry in S8 and are the most reliable grounding
  available - worth asking for even if nothing uses them until February.

## Design notes

**Cache aggressively.** Every run during development should hit a local cache,
not Polarion. A full re-pull is an explicit command. This keeps the test loop
fast and keeps you off the server's rate limiter.

**Strip HTML carefully, and keep the original.** Polarion descriptions are
HTML. Extraction evidence has to be a verbatim substring of the text the LLM
saw, so the stripped text is the canonical one - but keep the raw HTML in the
cache, because the first time a table or an embedded image matters you will
need it.

**One mapper module, and nothing else knows about Polarion.** The pipeline
downstream should consume a plain internal requirement type. When the real
client replaces a JSON stand-in, or a second ALM tool appears, that is a
mapper change and nothing more. Keep the boundary honest with a test.

**Paginate and expect partial failures.** A space with 2,000 work items will
page, and one malformed item must not abort the run - it gets reported and
skipped, like every other degradation in this pipeline.

## Acceptance criteria (G3, 4 Dec 2026)

1. Real work items from the agreed space produce a validated `.sysml` model,
   with a hardcoded registry and **no LLM in the loop**.
2. The package tree matches the document and chapter structure.
3. `derive` relations appear for the configured link roles, and ignored roles
   are *reported*, not silently dropped.
4. Re-running against the cache needs no network.
5. A work item that cannot be mapped is named in the report with a reason.

## Risks specific to this side

| | Risk | Mitigation |
|---|---|---|
| P1 | Requirement text lives in a custom field nobody mentioned | Dump one work item's full field set in S3 day 1 and read it with an engineer |
| P2 | Link roles differ from the guesses | Read the installation's link-role configuration; make the mapping a config file, not code |
| P3 | Descriptions contain tables and images that carry requirements | Report them as not-modelable with a reason rather than flattening them to nonsense |
| P4 | Access is revoked or the token expires mid-programme | Service account, not a personal token; fail with a clear message |
