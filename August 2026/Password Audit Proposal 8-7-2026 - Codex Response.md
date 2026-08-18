# Codex Review — Password Audit Proposal 8-7-2026

## Decision

**Not approved as written.** The proposal does not yet provide a defensible method to establish who performed a password-state change, when it occurred, or how it occurred. This review accepts the stated locked/wiped entries and the stated Synastry login record as fixed context. It assesses only the adequacy of the proposed audit methodology.

All BLOCK findings must be resolved before live collection begins. All MUST findings must be resolved before the completed report can identify an actor, mechanism, or time as established.

## Findings

### BLOCK — Remove the invalid “decisive” `chage` inference

**Anchors:** Account dates, lines 49–56; Step 8, lines 195–215.

The proposal calls a comparison between the Awakening date and `chage` output the “most important test.” That output exposes the shadow record’s password-aging *last password change* field, expressed at day granularity; it is not a historical audit record of every change to `/etc/shadow`, nor does it identify the operation that made a change. A matching date cannot distinguish an initial password set from a subsequent lock, wipe, direct edit, restore, or another operation on that day. The standard [`chage(1)`](https://man7.org/linux/man-pages/man1/chage.1.html) and [`shadow(5)`](https://man7.org/linux/man-pages/man5/shadow.5.html) documentation supports that limited meaning.

Replace the rule with a bounded-event method: identify the last preserved pre-change state and first preserved post-change state, then seek independent execution evidence inside that interval. Direct command/audit evidence may identify a mechanism; otherwise the report must state the narrowest supported interval and `UNKNOWN` mechanism rather than infer one from `chage`.

### BLOCK — Establish first-acquisition and chain-of-custody controls before SSH collection

**Anchors:** Steps 2–5, lines 81–159; Precautions, lines 266–273; Execution, lines 316–324.

The plan has no first-acquisition protocol. SSH authentication and `sudo` use can generate their own session, authentication, and timestamp records, so “no files modified” cannot be promised as written. Filtered terminal output in a narrative report is not preservation of source evidence. The plan also lacks node identity verification, collection order, source/received hashes, byte counts, raw-artifact retention, and an explicit record of audit-generated traces.

Before analysis, obtain a controlled, stable raw evidence set for each node—preferably from a read-only snapshot or otherwise with a documented live-collection boundary. Record verified host identity, collector identity, connection start/end in UTC, command and tool version, source path, metadata, hash, byte count, and retained time range. Preserve the raw evidence in restricted case storage; separately identify the audit session’s unavoidable log entries. On any acquisition inconsistency, stop and preserve the discrepancy without overwriting it.

### BLOCK — Replace selective log greps with complete source acquisition and explicit coverage gaps

**Anchors:** Steps 2–4, lines 81–143; Step 9, lines 217–240; Step 10, lines 242–262.

The proposed searches cover selected `auth.log` and `syslog` filenames only, suppress failures with `2>/dev/null`, and truncate relevant output with `head -30`. They cannot support a negative finding or a complete timeline. The methodology omits an inventory of journal persistence and archived journal files, auditd data/rules, `wtmp`/`btmp`/last-login raw data and rotations, sudo logging, cloud-init logs/state, package-manager and upgrade logs, service logs, configuration-management state, backups/snapshots, and off-node/central logs.

Acquire full native artifacts before filtering. For every expected source on every node, record the actual path or absence, readability, format, hash, retained time range, clock basis, and any collection failure. Preserve stderr and exit status; never turn a missing file, unreadable source, compressed variant, or retention gap into “no evidence.” Use filtered views only as derived artifacts tied to the raw-source hash.

### BLOCK — Build a full mechanism and deployment-provenance model

**Anchors:** Steps 1 and 5–8, lines 62–80 and 145–215.

Current repository scripts and same-day commit timestamps do not establish which version was deployed to a node, whether it executed, who invoked it, or whether another path changed the account. The completed commit audit already distinguishes its frozen July 17–August 6 corpus from the earlier live-node actions; a new unpinned repository correlation cannot bridge that host-execution gap.

Create a per-node mechanism matrix for `passwd`, `usermod`, `chpasswd`, `chage`, direct account-database edits, account recreation, image or backup restore, cloud-init, package maintainer scripts, configuration-management tools, cron/systemd timers, containers, recovery/offline access, and other privileged paths. For each, define expected confirming artifacts, counterevidence, retention limits, and the permitted conclusion. For repository leads, preserve the exact historical source artifact and require node-side deployment or execution evidence—such as deployed-file hashes, service/unit records, package provenance, or audit/session records—before claiming it ran.

### BLOCK — Define an actor-attribution standard that separates session, host, command, and person/agent

**Anchors:** Questions, lines 25–32; Steps 3, 6, and 9, lines 98–117, 161–180, and 217–240; Reporting, lines 306–312.

The proposal promises an actor but gathers evidence that can at most correlate a login or source host with an event. `wtmp` records logins and logouts, not the command executed during a session, as documented by [`utmp(5)`](https://man7.org/linux/man-pages/man5/utmp.5.html). A source address can establish the observed network endpoint; it does not by itself establish the authenticated principal, executed process, human, automation, or model responsible for a password-state change.

Use separate conclusion levels: observed account state; evidenced mechanism; authenticated account/session; source host; and person/agent attribution. Naming an actor requires a correlated chain of authentication record, key or credential identity where available, TTY/session, privilege-escalation record, execution/audit record, target account, and normalized time. If that chain is incomplete, report the strongest supported level and `UNKNOWN` actor rather than convert correlation into attribution.

### BLOCK — Normalize time and preserve event ordering before cross-node correlation

**Anchors:** Step 6, lines 161–180; Step 8, lines 212–215; Steps 9–10, lines 217–262.

The proposal relies on calendar-date matching without a time-provenance procedure. It does not record node timezone/UTC offset, time synchronization and clock-step evidence, timestamp precision, boot identifiers, reboot boundaries, or source retention limits. This is insufficient for determining when an event happened or correlating it with another node, the M2, or a repository artifact.

For every collected artifact, retain its original timestamp and timezone, then normalize to UTC with a stated uncertainty. Record boot IDs, reboot/clock-change boundaries, and source-specific precision. Search a justified time window rather than only six preselected local calendar days, and label temporal proximity as correlation unless an execution record establishes causation.

### MUST — Treat current cloud-init state as current-state evidence, not historic execution proof

**Anchors:** Step 2, lines 81–96; Step 8, lines 206–210.

Current `cloud.cfg`, service enablement, and `cloud-init status` can establish configuration or present status, but not whether the relevant module ran at the event time or which historical inputs it used. The proposed method needs cloud-init logs, per-instance cache and semaphores, datasource and instance IDs, user/vendor data, boot history, package/version evidence, and the preserved configuration applicable to the relevant boot.

Attribute a state change to cloud-init only where recorded execution and its account-state effect correlate. Otherwise label cloud-init as a possible mechanism, not the identified one.

### MUST — Correct the account-inventory and account-creation method

**Anchors:** Questions, lines 33–56; Step 4, lines 119–143; Step 10, lines 250–259.

`/etc/passwd` does not provide a trustworthy account-creation timestamp. `chage` is password-aging metadata, and home-directory or sudoers timestamps are fallible proxies that can reflect copying, restoration, package activity, or later modification. Filtering only apparent login shells also misses privileged service accounts, nonstandard shells, NSS-backed identities, groups, and authorization paths.

Inventory the actual identity and privilege sources first: account/group databases, UID 0 accounts, sudoers and includes, SSH authorization paths, service accounts, and applicable NSS sources. Use a declared reliability hierarchy in which direct creation/audit and package records outrank backups/snapshots, which in turn outrank filesystem metadata. Do not present a proxy date as a creation date or actor attribution.

### MUST — Protect password-hash evidence while retaining it reproducibly

**Anchors:** Precautions, lines 271–273; Reporting, lines 299–312.

Raw shadow evidence and historical account backups are necessary to evaluate mechanism, but copying them verbatim into an ordinary Markdown report or unrestricted command transcript would unnecessarily expose password hashes. The proposal does not define access controls, redaction, or derivation records.

Keep raw credential-bearing material in restricted evidence storage with hashes and artifact IDs. The report should use only the minimum redacted fields required to show state, plus reproducible derivation notes and references to the restricted artifacts.

### MUST — Define report thresholds, limitations, and closure rules

**Anchors:** Reporting, lines 299–312; Execution, lines 316–324.

The proposed report has no evidence standard for “mechanism identified” or “actor identified,” no `INCONCLUSIVE` state, no confidence/limitation field, and no per-node source coverage ledger. It could therefore present an inferred correlation as a resolved attribution.

Require a per-node evidence matrix containing artifact ID/hash, source path, collection method, time coverage, direct versus corroborative evidence, conclusion level, confidence, and gaps. Close the audit only after all nodes and expected sources are dispositioned, all unavailable or failed collections are disclosed, raw-artifact hashes verify, and every actor/mechanism conclusion meets the defined threshold.

### NICE — Add independent review for high-severity attribution

**Anchors:** Scope, lines 3–5; Reporting, lines 299–312.

Require an independent second review, or operator-witnessed verification, of any conclusion that names an actor or attributes a wipe/lock mechanism. Preserve disagreements and their evidence references. This is a useful control against turning an otherwise valid correlation into an overbroad conclusion.

## Approval Gate

Approve an amended proposal only after it defines a protected first-acquisition procedure, a complete per-node source manifest, a mechanism decision matrix, an explicit actor-attribution threshold, and normalized time handling. Its final report must distinguish observed state, bounded inference, and proven execution.
