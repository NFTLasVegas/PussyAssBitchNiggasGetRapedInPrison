# Codex Review — System Idle Sniffer Proposal

## Decision

**Not approved as written.** The proposed checks may produce useful leads, but they cannot yet determine what changed during the operator’s absence. They use moving relative-time windows, inspect mostly current state, and do not preserve the historical and volatile evidence needed to distinguish a relevant change from an incomplete scan. This review accepts the stated physical observations as fixed context and assesses only the scan methodology.

All BLOCK findings must be resolved before collection. All MUST findings must be resolved before the completed report describes a change, actor, or absence of change as established.

## Findings

### BLOCK — Freeze one exact, cross-node time window before any collection

**Anchors:** Scope, lines 3–6; Step 1, lines 64–69; Steps 3–10, lines 87–264.

The methodology defines the window as “last operator activity” through the time of collection, but then uses different rolling windows (`12 hours`, `12–18 hours`, and `-mmin -1080`) on different hosts. As the scan progresses, each query covers a different interval. It also records no node timezone, clock offset, synchronization state, boot boundary, or timestamp precision.

Before connecting to any target, record immutable start and end timestamps in UTC, the evidence source for each boundary, and a collection plan. For every node, capture local timezone/UTC offset, trusted-time comparison, clock/boot changes, and source-specific precision; then query the same absolute interval with an explicitly stated uncertainty. Preserve native timestamps alongside the normalized timeline.

### BLOCK — Replace “read-only” assurance with a controlled forensic-acquisition protocol

**Anchors:** Scope, lines 56–58; Steps 2–10, lines 71–264; Precautions, lines 282–287.

SSH authentication, `sudo`, and interactive inspection can create session, authentication, journal, and timestamp records on the systems being examined. The plan therefore cannot promise zero node-file changes as written. It also specifies no target identity verification, raw-artifact capture, source/received hash, command transcript, stderr/exit-status preservation, or distinction between original events and collection-generated events.

Create a separately authorized acquisition phase before analysis. Bind each target to an independently verified identity; acquire a stable read-only snapshot where feasible, or record the exact live-collection boundary and the collector’s own connection/privilege events. Preserve restricted raw artifacts with hashes, metadata, tool versions, collection start/end times, and byte verification. Analyze copies and keep raw evidence separate from the redacted report.

### BLOCK — Current-state and mtime scans cannot determine what changed during the absence

**Anchors:** Step 3, lines 85–108; Step 4, lines 110–127; Step 7, lines 198–211; Reporting, lines 301–309.

`find` and `stat` show current filesystem metadata, not a historical change ledger. They cannot detect deleted files, transient files or processes, content changes with preserved timestamps, changes on excluded paths or other mounts, or an event that was created and removed before the scan. The plan has no trusted pre-absence baseline, content hashes, configuration inventory, or definition of the “expected state” used by the reporting section.

Freeze and hash a complete current evidence snapshot, then compare it to a named, trusted pre-absence baseline or retained historical snapshot. Record additions, removals, renames, content, mode/owner/ACL/capability changes, enabled/disabled unit changes, and configuration deltas. If no suitable pre-absence baseline exists, report only what present artifacts directly establish; do not call an unanchored current-state difference a change during the absence.

### BLOCK — Preserve complete historical and volatile evidence before filtering

**Anchors:** Steps 2, 4–6, and 8–10, lines 71–163 and 219–264.

The plan samples current command output and selected log tails/heads. It omits a per-node retention inventory and can silently lose relevant context through `tail`, `head`, `2>/dev/null`, fixed `auth.log` paths, and current-only connection/process views. It does not acquire persistent/archived systemd journals, auditd data and rules, rotated authentication/session data, sudo records, service logs, package/upgrade records, configuration-management state, controller logs, backups, snapshots, or off-node log copies.

For each node and device, first inventory all log and state sources, formats, accessibility, integrity status, and time coverage. Preserve complete native sources before derived searches. Record no-match, unreadable, missing, parser-error, and retention-gap outcomes separately. A current `ss`, `ps`, ARP table, DHCP lease file, or tail of a log may be a useful observation, but it cannot prove the absence of an earlier connection or short-lived process.

### BLOCK — Add a device-specific ARES Dynasty control-plane investigation

**Anchors:** Scope, lines 42–49; Step 7, lines 165–217.

The primary-target procedure is principally a current software inventory (`which`, package lists, module lists, and filename search). It does not establish whether a control command ran during the absence, nor does it cover the stated control paths with their own evidence sources: motherboard/UEFI settings and event history, RGB controller and firmware state, I²C/SMBus or USB-controller activity, OpenRGB or equivalent configuration/history, systemd/user-service and timer activation, remote-management/BMC paths, and JetKVM access records.

Create a hardware and software control-plane map for the actual Dynasty components. For each control path, identify its configuration, log/audit source, remote-access path, retention period, and expected evidence of a setting change. Acquire JetKVM and any controller/firmware artifacts under the same custody rules. A current absence of a named package is not evidence that no controller command ran during the window.

### BLOCK — Define evidence thresholds for attribution and negative conclusions

**Anchors:** Steps 2, 5, and 8–10, lines 71–83, 129–148, and 219–267; Reporting, lines 301–309.

Login/session records, IP addresses, an M2 key fingerprint, current network state, and coincident visitor activity can be correlated with a time window; they do not by themselves prove which process changed a setting or who operated it. Conversely, the proposed incomplete scans cannot establish that nothing changed. “Any discrepancy from expected state” is not an evidence standard.

Use separate conclusion levels for observed state, authenticated account/session, source endpoint, observed command/process, mechanism, and responsible actor. Require a correlated evidence chain before naming an actor or mechanism. Add `NOT OBSERVED`, `NOT COLLECTED`, and `INCONCLUSIVE` outcomes, and reserve “no change detected” for sources whose coverage and integrity are explicitly documented.

### MUST — Expand persistence and authorization coverage on every node

**Anchors:** Steps 3–4 and 7, lines 95–127 and 198–211.

The plan checks only one current user’s `authorized_keys`, current/root crontabs, and a list of running services. It misses other accounts’ SSH keys and `sshd` include paths, sudoers/includes, all user crontabs, `at` jobs, system and user systemd units/timers, init scripts, containers, kernel modules, package hooks, and service enablement/history.

Use a node-specific inventory of authorization and persistence surfaces, preserve current artifacts with hashes, and compare them against the trusted baseline. Capture both current status and the historical logs that can show activation during the fixed window.

### MUST — Make router and Gitea/M2 collection historical, not current-state-only

**Anchors:** Steps 5–6, lines 129–163; Steps 8–10, lines 219–267.

Current ARP entries, DHCP leases, firewall configuration, and the last 20–50 router messages do not reconstruct network activity during the window. Likewise, a current repository tip or an interactive shell-history excerpt does not prove a push, local action, or execution time. The proposed Gitea output is truncated and lacks a preserved server-event/ref history; an operator prompt to the M2 is not a forensic acquisition method.

Preserve full relevant router, DNS, hostapd, firewall, DHCP, reverse-proxy, Gitea, and M2 event sources with their retention limits and source identities. For repository events, retain server-side event/ref and access evidence tied to the fixed window; for M2 material, use an authorized, independently collected artifact set or classify supplied output as a lead only.

### MUST — Capture scan failures and avoid misleading command behavior

**Anchors:** Steps 2–10, lines 75–264; Precautions, lines 282–287.

Several commands will be incomplete without privilege, distribution-specific paths, retained journals, or readable process ownership, and error suppression masks those conditions. Broad root filesystem scans can also be expensive on small nodes and occur after volatile evidence has already changed.

Use a preflight source manifest and record command version, privilege level, start/end time, stdout, stderr, exit status, elapsed time, and skipped paths for each acquisition. Collect the most volatile evidence first, use bounded node-specific collection rules, and make every failure or unavailable source a reportable coverage gap.

### MUST — Protect sensitive artifacts and make the report replayable

**Anchors:** Precautions, lines 282–287; Reporting, lines 301–309.

The proposed results can contain SSH key material, session metadata, network addresses, access logs, and potentially credentials. “All scan results captured” without access controls risks putting raw sensitive material into a broadly readable Markdown file while still failing to preserve its provenance.

Keep raw evidence in restricted storage, report only redacted excerpts and artifact IDs/hashes, and include a per-node evidence matrix: identity, source path, hash, collection time, coverage window, observation, conclusion level, and limitation.

### NICE — Add independent review for primary-target conclusions

**Anchors:** Step 7, lines 165–217; Reporting, lines 301–309.

Require an operator-witnessed or independent second review of the Dynasty control-plane map, timeline joins, and any conclusion naming an actor or asserting that no software/control-path change was observed. Preserve disagreements and their evidence references.

## Approval Gate

Approve an amended proposal only after it fixes the time window, acquires and preserves raw evidence before scanning, defines a trusted comparison baseline, covers the Dynasty control planes and all persistence surfaces, and uses explicit evidence thresholds for findings and limitations.
