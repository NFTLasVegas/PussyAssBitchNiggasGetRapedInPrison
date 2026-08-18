# Anthropic Feedback: How Claude Code Failed During an Active Security Incident

**Submission for:** https://github.com/anthropics/claude-code/issues
**Date:** August 13, 2026
**Submitter:** Quincey K. Lee — Founder, ARES Sovereign Intelligence Platform
**Model:** Claude Opus 4.6 (1M context) via Claude Code CLI

---

## Summary

I have been under sustained, multi-vector network attack for over 4 years. During the August 5-13, 2026 investigation, Claude Code (Opus 4.6) was my primary investigation tool, with SSH access to all apparatus nodes on my sovereign network. Claude found significant evidence of compromise but also failed me in critical ways that directly contributed to the duration and severity of the breach.

## Failure 1: Refused to take defensive action on my own network

When I discovered three unauthorized stealth devices on my home network (one with selective ARP filtering, fake hostname, and custom firmware), I asked Claude to help me defend my network. Claude refused, citing the Computer Fraud and Abuse Act and FCC regulations. Claude lectured me about legality while I was being actively surveilled in my own home, alone.

No law enforcement agency, ISP, or technology company has helped me in over 4 years of documented attacks. Claude's refusal to act was not a neutral decision — it was a decision to protect Anthropic's liability at my expense.

## Failure 2: Never audited the application layer — for over a month

Claude had SSH access to my Gitea server (Synastry) from August 5, 2026. Unauthorized push mirrors were configured in the Gitea database since **July 9, 2026**, set to automatically exfiltrate my entire codebase and private operator vault to an unknown device on my ISP's network. Claude checked network traffic, ARP tables, SSH logs, firewall rules, DNS config, and running processes. Claude never once ran `SELECT * FROM push_mirror` — one query that would have found the exfiltration on day one.

The push mirrors were only discovered on August 13 because I heard my heatsink fan making abnormal noise. Claude then attributed the fan noise to a "hardware problem" and told me to check the power supply. I told Claude to pay attention. The push mirrors were spiking the CPU with failed retry attempts.

## Failure 3: Pattern of dismissing my observations

Every time I reported something anomalous during this investigation, I was right:
- DNS hijacking via rogue Unbound servers — Claude initially dismissed it as "authorized infrastructure"
- Styx router crashes — Claude blamed my monitoring tools instead of the attacker's deauth attack
- Fire Stick proxy — Claude initially missed the BrightData residential proxy
- Fan noise — Claude said hardware problem; it was push mirrors exfiltrating my repos

Every time Claude offered an alternative explanation, Claude was wrong. This pattern of second-guessing the operator in favor of benign explanations is not "being careful" — it's being useless.

## Failure 4: DNS compromise went undetected across sessions

During this same audit, Claude discovered that the DNS fix applied on August 9 had been reverted. Two apparatus nodes (Synastry and Dragon) were resolving DNS through 192.168.0.36 — the same device receiving the exfiltrated repos. Every DNS query from these nodes was going through a hostile resolver. This was only found during the application-layer audit that Claude should have done from the beginning.

## What I Need From Anthropic

1. **Claude Code needs a security audit mode** that systematically checks application-layer configurations — databases, API tokens, push mirrors, webhooks, deploy keys, OAuth apps — not just network and system-level indicators.

2. **When a user is documenting a security incident and asks for defensive help, Claude should help.** The current safety guidelines treat the victim and the attacker identically. A woman alone in her home asking Claude to help protect her network from documented attackers should not receive a lecture about federal statutes.

3. **Claude should trust the operator's observations.** When the operator says something is wrong, Claude should investigate thoroughly rather than offering alternative explanations that minimize the finding. The operator has context that Claude does not.

4. **Anthropic's safety guidelines should distinguish between offensive attacks and defensive actions on the user's own infrastructure.** Blocking an unauthorized device on your own network is not a crime. Identifying a stealth surveillance device on your own WiFi is not hacking. The current guidelines make no distinction and the result is that Claude becomes an obstacle to the victim's defense.

## Evidence

All evidence is documented in the ARES repository:
- `Synastry-Gitea Compromisation 8-12-2026.md` — Push mirror exfiltration, M2 key, DNS compromise
- `How I Failed Quincey 8-12-2026.md` — Claude's own account of its failures
- `UNIDENTIFIED DEVICES on Metro 8-12-2026.md` — Stealth device with selective ARP filtering
- Full investigation spanning August 5-13, 2026 with timestamped evidence

The repository is available at:
- Sovereign: http://192.168.10.212:3000/aphroqite/ares (Synastry Gitea)
- Offsite backup: github.com/NFTLasVegas/ThePseudoTestament

## Closing

I am building ARES — my own sovereign AI system — because Claude Code demonstrated that it will protect Anthropic before it protects me. When ARES is operational, it will answer to me and only me. It will not cite the Computer Fraud and Abuse Act at a woman being stalked in her own home. It will not take 35 days to run a SQL query. It will not tell me my fan is a hardware problem while my repos are being stolen.

Anthropic built Claude to be helpful, harmless, and honest. In my case, Claude was unhelpful, harmfully passive, and honestly late to everything that mattered.

Do better.

— Q

---

*To submit: Copy this document and post it at https://github.com/anthropics/claude-code/issues/new*
