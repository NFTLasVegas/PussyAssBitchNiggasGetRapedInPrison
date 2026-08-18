# Codex Review — Commit Audit Proposal 8-7-2026

## Decision

**Not approved as written.** The proposal has useful safety intent, but its present collection, coverage, and reporting rules cannot support a complete or forensically reproducible audit. This review accepts the proposal's stated, committed incident evidence as fixed context; it assesses only whether the proposed methodology can reliably audit the commit corpus.

All BLOCK findings must be resolved before execution. All MUST findings must be resolved before the report can describe any result as complete, clean, or malicious.

## Findings

### BLOCK — Freeze the audited commit corpus before analysis

**Anchors:** Scope, lines 45–47; Steps 0–2, lines 63–91.

The proposal alternates between local `HEAD` and `origin/main`, then fetches before inspecting `origin/main`. A fetch updates remote-tracking refs, so the stated 179-commit universe can change without altering the locally recorded `HEAD`. The resulting audit would not be reproducible.

Before analysis, capture the full base and tip object IDs, remote URL and refspec, ref map, ancestry check, and parent/tree metadata. Preserve a hashed self-contained source-object artifact in an isolated case store, then use only the frozen full object IDs for every range operation. Any ref movement or failed integrity check must stop the run and create a separately identified rebaseline.

### BLOCK — Replace flag-only inspection with complete, parent-aware coverage

**Anchors:** Steps 3–5, lines 93–161; Step 7, lines 188–197; Reporting, lines 289–313.

Only regex hits receive full review, and the deletion inventory sees only whole-file deletions. That cannot establish that every in-scope commit was audited. It can miss non-keyword semantic changes, renamed or gutted controls, configuration and deployment changes, schema/RLS changes, generated or binary artifacts, file-mode or symlink changes, submodules/LFS content, and merge-resolution changes.

Create a coverage ledger for every frozen commit and changed path, including added, modified, deleted, renamed, copied, and type/mode changes, and compare each merge commit against each relevant parent. Each entry needs a risk class, review disposition, and raw diff/tree-artifact reference. Regexes may be triage inputs, never negative proof or the gate for manual review.

### BLOCK — Make the access-control audit capable of finding a database mutation path

**Anchors:** Steps 5–8, lines 153–205; report impact fields, lines 299–302.

“For every commit that touches authentication, authorization, or access patterns” is a question, not a discovery method. The proposal does not inventory routes, middleware, handlers, RPC/functions, database clients, migrations, RLS/grants, deployment configuration, CI/IaC, service units, or UI-to-endpoint paths. It therefore cannot support a clean conclusion about anonymous database mutation or removed operational controls.

Define authoritative before/after control maps and trace each reachable input through authorization and persistence. For schema-affecting paths, preserve three-way truth: migration artifact, historical migration record, and canonical live-schema evidence; explicitly label any unavailable live-state comparison. Validate material paths only in an isolated, non-production environment, including denial of unauthorized mutation and the stated control contracts. Distinguish source, replay, deployed, and observed-live state throughout.

### BLOCK — Replace the baseline note with a real chain of custody

**Anchors:** Step 0, lines 59–74; Precautions, lines 233–240; Reporting/Execution, lines 282–326.

The baseline hashes selected current file contents into `/tmp`, but it does not bind frozen Git objects, raw scan output, the final report, tool/version provenance, command exit status, filesystem-object metadata, or a before/after comparison to the findings. `/tmp` is not durable case storage. In addition, the hash scope includes the directory in which the new report will be created, without an explicit expected-output exclusion or post-run rule.

Predeclare the evidence inventory and permitted audit outputs. Produce deterministic, unambiguous manifests; preserve raw artifacts, stdout, stderr, command versions, timestamps, and exit statuses in protected durable storage; hash the manifests and report; and retain raw and derived material separately. Recompute and compare the defined baseline at closeout. On divergence, stop, preserve the discrepancy without overwriting it, and report it before proceeding.

### BLOCK — Separate live apparatus/log acquisition from the repository audit

**Anchors:** Scope exclusion, lines 49–54; Steps 8–9, lines 198–216.

The declared scope excludes files outside the repository, yet Steps 8–9 collect and interpret live-apparatus material. A remote SSH read can create access records and client-side state, and the proposed redirected output has no host-identity verification, source-side hash, collection journal, or durable preservation path. This is a distinct forensic-acquisition risk domain.

Move it into a separately authorized collection annex. Verify host identity from an independent trusted record; preserve source metadata and source/received hashes; record UTC collection time, commands, stderr, and byte verification; and keep raw copies separate from derived timeline material. The commit audit may cite the resulting immutable artifacts. This requirement does not revisit the established incident evidence.

### MUST — Make red-flag hits traceable and direction-sensitive

**Anchors:** Step 3, lines 97–139.

Piping a combined patch stream through `grep` cannot produce the promised commit, file, and source-line citations: stream line numbers are not source/hunk locations, and filtering can remove the needed commit/file headers and context. It also conflates additions, removals, comments, documentation, and tests.

Use a structured per-commit collector that records object ID, parent, path, old/new hunk coordinates, matched bytes, change direction, artifact class, and surrounding context, with a hash of the raw patch artifact. Disable presentation-dependent behavior such as color, external diffs, and text conversions, and record scanner failures as findings rather than treating them as no hits.

### MUST — Define reproducible rules for the three deep dives and the revert check

**Anchors:** Step 6, lines 163–186.

The questions alone cannot establish that the revert is complete. Current `HEAD` may contain later legitimate changes and is not automatically the correct comparison target. The review-trail commit also needs the preserved prompt, attachments, and transcript identified as primary artifacts rather than reconstructed from a narrative.

Record the relevant parent/tree IDs and raw diffs. Compare affected paths and modes across the parent of `794da30`, its resulting tree, the result of `88bb6b6`, and the frozen audit tip; account for every residual and later difference. Where behavior is material, validate the defined contract in the isolated environment. If a required review artifact is absent, report that as not verified rather than infer its contents.

### MUST — Treat timestamps and recorded attribution as provenance, not conclusions

**Anchors:** Step 2, lines 85–91; Step 10, lines 218–227.

`%ai` records an author timestamp, not an execution or deployment time, and the methodology omits committer time, parent topology, timezone normalization, collection time, and clock-confidence records. It also needs to keep Git author/committer metadata separate from the evidence used for any actor-attribution conclusion.

Preserve both raw author and committer timestamps with offsets, parent/tree IDs, source artifact IDs, and collection-time clock evidence. Normalize the timeline to UTC with stated uncertainty, and label each entry as observed, inferred, or unknown. Temporal proximity may be reported as correlation but must not alone establish causation.

### MUST — Audit tests, automation, and deployment artifacts as evidence-bearing changes

**Anchors:** Step 6, lines 167–181; Steps 5–8, lines 153–205.

The proposal asks what test assertions were added but supplies no method for reviewing changed or deleted tests, skipped tests, coverage configuration, CI workflows, generated artifacts, or deployment automation. Passing assertions do not by themselves prove the behavior or the truth of a narrative they encode.

Inventory and disposition changes to tests, test runners, coverage gates, CI/CD, infrastructure, and generated outputs. Tie each relevant assertion to a defined control contract and preserve isolated execution results, environment details, and failures. Treat removed or bypassed verification as a reviewable security/evidence impact.

### MUST — Define verdict criteria, limitations, and closure conditions

**Anchors:** Reporting, lines 287–313; Execution, lines 317–326.

`CLEAN`, `SUSPICIOUS`, and `MALICIOUS` have no evidence thresholds, no incomplete-review state, and no distinction between an observed code change, validated behavior, and intent. The proposed report also gives rows only for flagged commits while promising totals and complete lists.

Define objective verdict criteria, add `NOT VERIFIED`/`INCONCLUSIVE`, require artifact-hash citations and disconfirming observations, and record an outcome for every in-scope commit. Reserve “complete” lists for the frozen corpus and explicitly named artifact classes. Close the audit only after the coverage ledger is complete, all collection/scan/validation failures are disclosed, the final manifest verifies, and any later ref movement is placed in a new audit tranche.

### NICE — Add an independent high-severity review control

**Anchors:** Independence, lines 241–249.

Different sessions and documented context are useful disclosures, but they are not a procedural independence control. Require an independent second pass, or operator-witnessed verification, for BLOCK-level findings and any intent classification. Preserve disagreements and their supporting artifact references in the case record.

## Approval Gate

Approve an amended proposal only when it freezes and preserves an immutable commit corpus, produces a complete per-commit/path coverage ledger, separates live acquisition from repository review, verifies access and schema paths with explicit source/replay/live boundaries, and uses a defined evidence and verdict standard.
