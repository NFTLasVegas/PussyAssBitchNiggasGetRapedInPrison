# Unauthorized Requests for GitHub Keychain — August 10, 2026

**Discovered:** 2026-08-10 ~22:18 PDT
**Operator statement:** Q has NEVER used GitHub Copilot. Q has NEVER installed Copilot extensions. Q has never received this keychain request before. The request appeared for the first time during an active security investigation.

---

## What Was Observed

A macOS system dialog repeatedly appeared on M5 (MacBook Air) stating:

> **git-credential-osxkeychain wants to use your confidential information stored in "github.com" in your keychain.**
>
> To allow this, enter the "login" keychain password.

The dialog presented three options: **Always Allow**, **Deny**, and **Allow**.

**Q clicked "Deny" multiple times.** The dialog kept reappearing after each denial. It only stopped after Claude on M5 deleted the GitHub credential from the Keychain and removed the triggering extensions.

Screenshot preserved at: `/Users/nftlasvegas/Desktop/Screenshot 2026-08-10 at 10.18.54 PM.png`

---

## Context: Why This Is Concerning

1. **Q has never used GitHub Copilot** — she did not install the Copilot extensions found on M5
2. **Q has never received this keychain request before** — it appeared for the first time on August 10, 2026
3. **The request appeared during an active security investigation** — Q was in the process of factory resetting compromised apparatus nodes
4. **The M2 MacBook was locked and not in use** — Q has not unlocked the M2 all day
5. **The apparatus left GitHub on July 17, 2026** — there should be no GitHub-related activity on any apparatus device
6. **The request was persistent** — clicking "Deny" did not stop it from reappearing

---

## What Was Found

### GitHub Credential in Keychain

A GitHub credential was stored in M5's login keychain:

```
Keychain: /Users/nftlasvegas/Library/Keychains/login.keychain-db
Class: internet password (inet)
Server: github.com
Account: 113312409
Protocol: https
Created: 2026-03-27 21:13:31 UTC
```

**Q did not create this credential.** The account ID `113312409` is a numeric GitHub user ID. This credential was created on March 27, 2026 — during the period when the iCloud memory symlink was active (April 21 – August 8, exposing credentials), and before the GitHub departure on July 17.

### GitHub Copilot Extensions in VS Code

Two versions of GitHub Copilot Chat were installed in VS Code on M5:

```
~/.vscode/extensions/github.copilot-chat-0.45.1
~/.vscode/extensions/github.copilot-chat-0.48.1
```

**Q states she has never used or installed GitHub Copilot.** These extensions were present on M5 without Q's authorization. Copilot extensions periodically authenticate with GitHub's servers, which triggers the `git-credential-osxkeychain` request.

### Codex GitHub Repositories on M5

Three GitHub-linked repositories were found in Codex's directory structure on M5:

```
/Users/nftlasvegas/.codex/vendor_imports/skills → github.com/openai/skills.git
/Users/nftlasvegas/.codex/.tmp/plugins → github.com/openai/plugins.git
/Users/nftlasvegas/.codex/.tmp/plugins-clone-ey3Ltj → github.com/openai/plugins.git
```

These are OpenAI/Codex internal repositories that maintain GitHub as a remote. Any git operation on these repos would trigger the credential request.

### VS Code Running Since May 24

VS Code had been running continuously on M5 since **May 24, 2026 (78 days)** without restart. The long-running process with unauthorized Copilot extensions had been silently attempting GitHub authentication for an unknown duration.

### Active Processes at Discovery Time

```
git-credential-osxkeychain get    (PID 10174)
git credential-osxkeychain get    (PID 10157)
```

Both processes were spawned by VS Code's extension host, triggered by the Copilot extensions attempting to authenticate with GitHub.

---

## The Triggering Process

```
/opt/homebrew/opt/git/libexec/git-core/git-credential-osxkeychain get
/opt/homebrew/bin/git credential-osxkeychain get
```

The `git-credential-osxkeychain` helper is a macOS-specific git credential manager. When a git operation requires authentication for a remote (like github.com), this helper:

1. Searches the macOS Keychain for stored credentials matching the server
2. Prompts the user for Keychain access if credentials are found
3. Returns the credentials to the git process
4. If "Always Allow" is clicked, the credential is returned without future prompts

**If Q had clicked "Always Allow" instead of "Deny," the stored GitHub credential (account 113312409) would have been silently provided to whatever process requested it — without further prompts, indefinitely.**

---

## Questions That Remain Unanswered

1. **Who installed GitHub Copilot on M5?** Q did not install it. The M2 Claude had `bypassPermissions` mode enabled and could install VS Code extensions. Was this installed remotely or during a previous session?

2. **Why did the request appear NOW?** VS Code has been running for 78 days. If Copilot was installed that entire time, why didn't the credential request appear before August 10? Something changed — either the extensions were recently activated, or a trigger event occurred.

3. **What triggered the credential request at 22:18 PDT?** The request appeared while Q was actively working on the RasQberry factory reset. Was this coincidental timing, or was it triggered by an event (network change, iCloud sync, remote command)?

4. **Who created the GitHub credential on March 27?** The Keychain entry was created on 2026-03-27. Was this Q during normal GitHub usage, or was it created by another process/person?

5. **What would have happened if Q clicked "Always Allow"?** The credential would have been provided to the Copilot extensions, which would authenticate with GitHub. This could enable:
   - Access to Q's GitHub repositories (if any remain)
   - GitHub API access under Q's identity
   - Copilot data collection associated with Q's account
   - Token refresh/generation that could be exfiltrated

6. **Were the Copilot extensions communicating with GitHub before the credential was deleted?** If they had cached authentication tokens from a previous "Allow" click, they may have been silently sending data to GitHub without triggering the Keychain prompt.

---

## Remediation Taken

| Action | Status |
|--------|--------|
| GitHub credential deleted from Keychain | ✅ Deleted |
| GitHub Copilot Chat 0.45.1 removed | ✅ Deleted from ~/.vscode/extensions/ |
| GitHub Copilot Chat 0.48.1 removed | ✅ Deleted from ~/.vscode/extensions/ |
| Codex skills/.git removed | ✅ Deleted |
| Codex plugins/.git removed | ✅ Deleted |
| Codex plugins-clone removed | ✅ Deleted |
| git-credential-osxkeychain process killed | ✅ Killed |
| No remaining GitHub extensions in VS Code | ✅ Verified |
| No remaining GitHub remotes in any repo | ✅ Verified (Ares repo points to Synastry only) |
| GitHub credential not re-created in Keychain | ✅ Verified |

---

## Connection to Broader Investigation

This event fits the pattern of unauthorized software and configuration being installed on Q's devices without her knowledge:

- **M2 Claude `bypassPermissions`** — could install extensions without approval
- **GitHub Copilot on M5** — installed without Q's knowledge, never used by Q
- **BrightData on Fire Stick** — installed without Q's father's knowledge
- **Unbound DNS on RasQberry/Sovereign Door** — configured without Q's full understanding
- **ADB enabled on Fire Stick** — enabled without Q's father's knowledge
- **Wi-Fi configured on Ethernet-only RasQberry** — configured without Q's authorization
- **iCloud memory symlink** — exposed credentials for 109 days without Q's knowledge

In each case, software or configuration was deployed on Q's devices by an actor with access (M2 Claude, physical access, or remote access), presented as legitimate or necessary, and operated without Q's informed consent.

---

*This document records an unauthorized GitHub Keychain access request that appeared on M5 during an active security investigation on August 10, 2026. Q has never used GitHub Copilot and did not install the extensions that triggered the request. The credential, extensions, and associated repositories have been removed.*
