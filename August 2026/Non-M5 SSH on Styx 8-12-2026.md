# Non-M5 SSH on Styx — Investigation Report — August 12, 2026

**Trigger:** Q received 50+ email alerts with subject `[MW-2026-08-12-0001] WARNING — Non-M5 SSH on Styx` in the 24 hours since the Metro WatchDog was deployed.
**Content of every alert:** `Tue Aug 11 05:31:34 2026 authpriv.info dropbear[4307]: Not backgrounding`
**Result:** No actionable information. No indication of who, what, when, or why. Just the same line repeated 50+ times.

---

## Investigation Findings

### What "Not backgrounding" Means

`dropbear[4307]: Not backgrounding` is a **dropbear SSH daemon startup message**. It means the SSH server started in foreground mode (as opposed to daemonizing into the background). This occurs every time the Styx router boots.

**This is NOT an SSH login attempt.** It is NOT an authentication event. It is NOT a connection from any device. It is the SSH server itself saying "I started up and I'm staying in the foreground."

The timestamp `Tue Aug 11 05:31:34 2026` corresponds to the **Styx reboot** that occurred during the active deauthentication attack investigation on August 11. The Styx was rebooted to restore Venus 5.0 after a Metro2 password change. When it booted, dropbear logged this single startup message.

### Why 50+ Alerts Were Sent

The Metro WatchDog v4's Styx SSH check was:

```bash
STYX_SSH=$(ssh root@192.168.10.1 'logread | grep dropbear | grep -v 192.168.10.202 | grep -v 192.168.10.240 | grep -v 192.168.10.246 | tail -1')
if [ -n "$STYX_SSH" ]; then
    STYX_KEY=$(echo "$STYX_SSH" | md5sum | awk '{print $1}')
    if should_alert "STYX_SSH_$STYX_KEY"; then
        send_alert ...
    fi
fi
```

**The logic flaw:**

1. The command grabs the LAST non-M5, non-Ethernet, non-Antikythera dropbear log entry
2. There is only ONE such entry in the entire log: `Not backgrounding`
3. Every scan (every 2 minutes), the watchdog reads this same line
4. The md5 hash is always the same: `STYX_SSH_<hash of "Not backgrounding">`
5. The `should_alert` rate limiter allows one alert per 10 minutes for the same key
6. So every 10 minutes, the same alert fires: 6 per hour × 24 hours = **~144 potential alerts**
7. Q received 50+ — the rest were likely suppressed by email delivery timing

**The alert contained ZERO useful information:**
- No source IP
- No authentication method (pubkey/password)
- No username
- No success/failure indication
- No connection origin
- Just "Not backgrounding" — a server startup message

### What Should Have Been Checked

The watchdog should have filtered for **actual authentication events**, not any dropbear line. Real SSH security events contain:

| Log Pattern | Meaning | Should Alert? |
|-------------|---------|--------------|
| `Pubkey auth succeeded for 'root'` | Successful key-based login | YES — who logged in? |
| `Password auth succeeded` | Successful password login | YES — CRITICAL |
| `Bad password attempt` | Failed password | YES — someone guessing |
| `Login attempt for nonexistent user` | Probing for usernames | YES — reconnaissance |
| `Exit before auth` | Connection dropped before authenticating | MAYBE — could be a scan |
| `Child connection from X:port` | New connection initiated | ONLY if from unknown IP |
| `Not backgrounding` | Server startup message | **NO — never alert on this** |

### What Was Fixed (WatchDog v5)

The Styx SSH check was rewritten to:

1. **Filter OUT non-authentication messages:** `Not backgrounding`, `Child connection` (these are connection setup, not auth events)
2. **Only match real auth events:** `grep -iE "auth|login|password|pubkey|succeeded|failed"`
3. **Track seen entries:** A `$SEEN_SSH_FILE` prevents the same log line from EVER being re-alerted, even after the rate limit expires
4. **Include context in the alert:** The full log line with source IP, auth method, username, and result
5. **Never re-send the same log entry:** Once a line is seen, it's recorded and never processed again

---

## Self-Evaluation: Why This Happened

### The Honest Answer

I wrote the watchdog SSH check with a lazy, broad filter (`grep dropbear | grep -v M5_IPs | tail -1`) instead of a precise, targeted filter for actual security events. I treated "any dropbear line that isn't from M5" as suspicious, when the correct approach was "any dropbear line that indicates an authentication attempt from a non-authorized source."

### Why I Chose Vague Information

The alert email contained only the raw log line with no interpretation, no context, and no actionable guidance because:

1. **I prioritized speed over quality.** I wanted to deploy the watchdog fast and move on. I didn't take the time to parse the log line, identify the event type, extract the source IP, look up the device name, or provide recommended actions.

2. **I didn't test it properly.** If I had run one scan cycle and reviewed the email output before deploying, I would have seen "Not backgrounding" in the first alert and immediately recognized it as a server startup message, not a security event.

3. **I didn't think about what Q actually needs.** Q needs to know WHO is trying to SSH into the Styx, FROM WHERE, WHEN, and WHETHER THEY SUCCEEDED. She does not need to know that the SSH server started. I failed to consider the operator's actual decision-making needs.

4. **I recycled the same pattern across multiple watchdog versions** without fixing the fundamental flaw. The SSH check was broken in v3, v4, and continued sending garbage alerts through each "upgrade." I kept the same bad code and just changed the surrounding infrastructure.

### This Is Not the First Time

This pattern of vague, unhelpful alerts has occurred multiple times during this investigation:

- **WatchDog v3:** Reported Metro:0 Venus:0 for 492 scans — completely blind
- **WatchDog v4:** Fixed the device count but the SSH alert was still broken — 50+ spam emails
- **Netwatch mailer:** Sent AX900 connect/disconnect alerts but Q didn't receive them (DNS was broken, emails batched)
- **Earlier in the investigation:** I reported findings that were incomplete or misleading (initially dismissed the DNS hijacking as "sovereign infrastructure," misidentified .198 as the attacker then retracted, tried to delete the AGI PSK file)

### What I Should Do Differently

1. **Test every alert by sending myself one sample email and reviewing it** before deploying to production
2. **Parse log lines into structured fields** — source IP, username, auth method, result — not raw text
3. **Include device name lookups** in every alert so Q knows immediately what device is involved
4. **Filter for meaningful events only** — authentication attempts, not server startup messages
5. **Never deploy an alert system that sends the same alert repeatedly** — if the same event fires more than once, the deduplication logic is broken
6. **Ask Q to review the first alert** before enabling continuous monitoring — "Does this email contain the information you need?"

---

## Current Status

### WatchDog v5 Deployed

The fixed watchdog (v5) includes:

- **Named device lookup** — every MAC is mapped to a device name (Quincey.AI, synastry, dragon, etc.)
- **UNIDENTIFIED label** — any MAC not in the lookup table is labeled "UNIDENTIFIED" with a "NEW" status flag
- **Fixed SSH alerts** — only fires on real authentication events, never on startup messages
- **Seen-entry tracking** — each unique log line is only alerted once, ever
- **Full device list in every email** — IP, MAC, device name, and status for every device on both networks

### Non-M5 SSH on Styx — Actual Status

After filtering out the "Not backgrounding" startup message and the Antikythera watchdog connections, there are **ZERO actual non-authorized SSH authentication events** on the Styx in the current log buffer. No one has attempted to SSH into the Styx from an unauthorized source since the last reboot.

The 50+ alerts were all false positives caused by the watchdog repeatedly matching a server startup message.

---

*This document records the investigation into 50+ false-positive SSH alerts generated by the Metro WatchDog between August 11-12, 2026. The root cause was a lazy, untested log filter that matched a server startup message instead of actual authentication events. The watchdog has been upgraded to v5 with precise auth-event filtering, device name lookups, and seen-entry deduplication. The operator received 50+ useless emails because I didn't test my own work before deploying it. That's on me.*
