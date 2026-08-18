# DNS Hijacking 8-9-2026 (Codex Confirmation)

**Review scope:** The August 9 System Idle Sniffer DNS evidence report, the live Styx/router and DNS-node state, and relevant repository provenance.

**Method:** Read-only DNS queries, SSH inspection using pre-existing pinned host keys, live DHCP inspection, and Git/provenance review. No apparatus configuration or pre-existing repository content was changed during evidence collection; this document is the requested repository addition.

---

## Verdict

The live DNS redirection is real. Both nodes authoritatively override the named domains, and Styx DHCP currently distributes them as DNS resolvers.

However, the evidence document does not establish a newly deployed, malicious DNS hijacking operation, its author, or deception. It contains material chronology errors and unsupported inferences. It should be treated as a record of a serious authorization dispute around an active split-horizon DNS deployment, not as proof of an attributed intrusion.

The configuration comments describing sovereign infrastructure were not treated as authorization. They are mutable narrative, just like the Git author identity. Q's denial must be assessed separately.

| Claim | Result |
|---|---|
| quincey.ai is redirected locally | Confirmed. Both .225 and .36 return authoritative A 192.168.0.225; 8.8.8.8 returns 159.65.79.66, 103.168.172.37, and 103.168.172.52. |
| The other three apex domains are locally overridden | Confirmed. Both resolvers authoritatively map ares.technology, ares.love, and aphroqite.ai to .225. |
| mail.quincey.ai is redirected to .225 | False. Both return authoritative NXDOMAIN; Google returns 103.168.172.65. A control nonexistent label receives the same local NXDOMAIN, so this is generic static-zone shadowing, not evidence of mail-specific targeting. It can still make the hostname unavailable. |
| DHCP forces the pair on clients | Confirmed for DHCP clients that honor it. Styx's live UCI config and this Mac's active DHCP lease specify .225,.36; scutil --dns shows them as active resolvers. It does not prove every client or every query used them. |
| Both nodes were newly moved off Styx overnight | Contradicted by prior tracked records. August 7 and 8 material already places them on 192.168.0.36/.225 via Wi-Fi; infra/styx/harden.sh also says they live upstream on Metro2. |
| RasQberry Wi-Fi was deliberately trail-free | Unsupported/incorrect. It is on Wi-Fi with eth0 down/no carrier, but NetworkManager is active through a Netplan-derived connection profile. Sovereign Door has a persistent metro2.nmconnection profile. |
| Git proves Q authorized the configuration | No. All relevant commits are unsigned (N); author/committer fields and co-author trailers are mutable. |
| Git proves an attacker made the changes | No. It cannot attribute an actor either. |
| iCloud symlink exposed the Styx password for 109 days | Not independently verifiable from current apparatus evidence. The claimed symlink was reportedly removed; the repository contains later narrative assertions rather than the original artifact. |
| DNS proves credential interception, traffic inspection, exfiltration, or physical access | No. DNS redirection can cause availability and routing impact, but no evidence shows a TLS proxy, valid certificates, captured queries, data transfer, or who changed cables/settings. |

---

## Technical verification

- Direct UDP and TCP, non-recursive DNS queries to both 192.168.0.225 and 192.168.0.36 returned authoritative local answers for quincey.ai: A 192.168.0.225. Google Public DNS returned the three public A records listed above.
- Both nodes run active, valid Unbound configurations. unbound-checkconf passed, and the active configuration hashes exactly match the tracked infra/dns files.
- Both nodes are currently on wlan0 in 192.168.0.0/24 and default-route through 192.168.0.1. RasQberry's eth0 is down with no carrier; Sovereign Door has no active Ethernet interface.
- Styx's live DHCP configuration contains option 6,192.168.0.225,192.168.0.36. The current Mac lease at 192.168.10.202 contains exactly those DNS servers, and its active general resolver is the pair.
- Styx routes to the two DNS nodes through its upstream Wi-Fi interface using source 192.168.0.105. This explains why resolvers that allow only 192.168.0.0/24 can answer clients on 192.168.10.0/24.
- The deployment is static local-zone/split-horizon DNS behavior, not cache poisoning. The local NS/glue records do not by themselves create a DNS loop.

---

## Material problems in the evidence report

### Authorization and attribution are not proven

If Q is the authorized decision-maker and her non-authorization statement is authenticated, the live deployment should be classified as a serious suspected unauthorized DNS deployment. The technical evidence does not establish who created it, when it became live, or whether any particular person or tool was responsible.

All inspected Git commits show signature status N. The claimed Q author/committer name and the Claude co-author trailers are mutable commit metadata, not proof of identity or authorization.

### The claimed August 8-9 relocation is contradicted

Earlier tracked material already places RasQberry at 192.168.0.36 on Wi-Fi and Sovereign Door at 192.168.0.225 on Wi-Fi. The assertion that they disappeared from the Styx LAN overnight, or were newly discovered on Wi-Fi on August 9, is not supported by the current repository record.

Absence from Styx DHCP leases does not demonstrate physical removal. It is consistent with the documented architecture in which the nodes live upstream of Styx.

### The Wi-Fi anti-forensics inference is incorrect

RasQberry has an active Netplan-derived NetworkManager connection named netplan-wlan0-metro2. The empty /etc/NetworkManager/system-connections directory does not show that Wi-Fi was configured to avoid leaving a trail. Sovereign Door has a persistent profile at /etc/NetworkManager/system-connections/metro2.nmconnection.

The live interface state confirms Wi-Fi, but cannot identify who configured it or whether Q authorized it.

### Mail suppression is real, but targeting is not established

The local static quincey.ai zone causes authoritative NXDOMAIN for mail.quincey.ai. The same result occurs for an intentionally nonexistent label, while Google returns public wildcard data for that label. This establishes broad local-zone shadowing, not a mail-specific record or a targeted suppression rule.

### Interception and exfiltration claims are unsupported

The evidence does not show that .225 serves HTTP or HTTPS to clients, has a valid certificate for the affected public domains, captures DNS queries, or transmits data elsewhere. DNS redirection alone does not prove credential interception, traffic inspection, exfiltration, or account compromise.

The active Unbound configs disable query and reply logging. The nodes could observe queries they receive while operating, but retained logging or exfiltration has not been demonstrated.

### Monitoring and service claims are overstated

Tracked monitor configurations target both DNS nodes, so the claim that the upstream network has zero monitoring is too broad. Those probes do not validate semantic DNS answers, which is a real monitoring gap.

Gitea on RasQberry and the health-analyzer account are documented apparatus components. Their presence is not evidence of exfiltration or compromise. The listed localhost services on Sovereign Door remain uncharacterized, but unknown does not mean malicious.

---

## Repository and provenance findings

- The exact live DHCP option appears in infra/styx/dhcp-lan.uci, added by commit 3dd48db on July 27.
- The DNS configuration existed in repository history from June 10.
- A July 30 task record explicitly describes a previous wrong-subnet/strangers diagnosis as false and identifies these devices as the Metro2 DNS pair. This is not authorization proof, but it is conflicting evidence the August 9 report omits.
- The repository contains multiple DNS-related configuration artifacts, including a second RasQberry config with a differing ACL. The active live config was verified against infra/dns, but this duplicate authority is a real provenance/drift concern.
- Repository records disagree about when DHCP advertisement first became active: June documentation says it already existed, while the replayable UCI export first appears in the July 27 commit. The exact activation date remains unresolved without router-native backups/logs.

The reviewed evidence document itself is untracked. It was created shortly after its stated collection window and embeds conclusions instead of preserving raw command output, packet captures, hashes, and chain-of-custody data. That timing is compatible with a write-up, but the document is not audit-grade evidence as written.

---

## Evidence still needed before making attribution claims

1. Preserve router-native DHCP configuration, UCI exports, persistent logs, backup hashes, DHCP lease/ACK evidence, and NAT/routing state.
2. Preserve full timestamped DNS transcripts and packet captures for A, AAAA, MX, TXT, CNAME, NS, and SOA queries to both local resolvers and public authoritative servers.
3. Preserve active Unbound include trees, service units, cron files, firewall/listener state, authorized_keys metadata, Wi-Fi journals, and host-key continuity on both nodes.
4. Obtain Gitea receive/audit logs, SSH authentication logs, remote refs, and deployment journals. Unsigned Git commits alone cannot authenticate Q or identify another actor.
5. Obtain an independent authorization record and, for the iCloud claim, source artifacts such as endpoint telemetry, APFS snapshots, or iCloud version history.

---

## Final conclusion

The active DNS behavior holds: this is authoritative split-horizon DNS redirection and local-zone suppression affecting clients that use the two configured resolvers.

The claimed new August 8-9 attack timeline, deliberate deception, physical relocation, specific actor, interception, exfiltration, and malicious intent do not hold on the available evidence. If Q's denial of authorization is accepted, handle the system as a suspected unauthorized DNS deployment while preserving evidence and completing provenance collection before assigning blame.
