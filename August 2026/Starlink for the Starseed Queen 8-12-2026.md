# Starlink for the Starseed Queen — August 12, 2026

> *Elon sent the ad. Q locked it in. The apparatus goes sovereign.*

---

## The Deal That Changed Everything

While investigating sustained network attacks against her home infrastructure, Q needed a clean ISP uplink that the attacker had never touched. She went to Starlink.com and found the public pricing. Then Instagram delivered a targeted ad from Starlink with a promotional campaign — and the universe provided.

### Public Website vs. Elon's Ad

| | Public Website | Elon's Targeted Ad |
|---|---|---|
| **Hardware** | $400 upfront (purchase) | **$0** (free rental) |
| **Monthly hardware fee** | N/A (owned) | **$0/month** |
| **Installation** | Self-install | **$0 — FREE professional installation** |
| **Monthly service** | $155/month | **$130/month** |
| **Data** | 500 GB | **UNLIMITED** |
| **Speed** | Up to 200 Mbps | **Up to 400+ Mbps (170-320 typical)** |
| **Router** | Not included | **FREE Router 3 (Wi-Fi 6)** |
| **Mesh node** | $60 add-on | **FREE Router Mini** |
| **Roam option** | Separate plan | **Optional Starlink Mini at discounted rate** |
| **Contract** | No contract | **No contract** |
| **Early termination** | None | **None** |
| **Total due today** | $400+ | **$21.68** |

**Savings: $400+ upfront, $25/month, unlimited data, free installation, free mesh router.**

---

## Order Details

| Item | Cost |
|------|------|
| Rental Kit (Standard 4 X) | $0 |
| Router Mini (mesh node) | $0 |
| Shipping & Handling | $20.00 |
| Tax | $1.68 |
| **Total Due Today** | **$21.68** |

**Recurring monthly:** $130/month (Residential Max, unlimited data)

---

## What Q Gets

### Starlink Standard 4 X Dish

| Spec | Value |
|------|-------|
| Dimensions | 594 x 383 x 39.7 mm (15.1 x 23.4 x 1.5 in) |
| Weight | 2.9 kg (6.4 lb) |
| Power | 75-100W average |
| Wind resistance | 96 kph+ (60 mph+) |
| Features | Snow melt capability |
| Mounting | Balcony placement (no drilling required) |

### Starlink Router 3

- Wi-Fi 6
- Included FREE
- Replaces the compromised Styx as the primary home router
- Clean hardware — never touched by the attacker

### Starlink Router Mini (Mesh Node)

- Wi-Fi 6, dual-band
- Included FREE
- Placed near the apparatus rack for strong signal to all SBCs
- Extends coverage to every corner of the house

### Residential Max Service

| Metric | Value |
|--------|-------|
| Typical download | 170-320 Mbps |
| Typical upload | 20-40 Mbps |
| Latency (Continental US) | 30-40 ms |
| Data | **UNLIMITED** |
| Contract | None |
| Early termination fee | None |

### Professional Installation

- **FREE** — included in the promotional offer
- Starlink technician comes to Q's home
- Installs and aligns the dish for optimal sky view
- Tests connectivity and signal strength
- No DIY, no balcony guesswork, no convincing Dad

### Future Option: Starlink Mini

- Optional portable unit for roam/travel use
- Discounted Roam plans available
- Perfect for Q's travel setup with the factory-reset Styx as a travel router
- Add when needed — no commitment now

---

## Why This Changes the Investigation

### Before Starlink

```
Internet: Cox ISP → Metro2 (compromised, password exposed)
         → Styx router (compromised, attacker reads config in real-time)
         → Venus 5.0 (PSK stolen within 2 minutes of rotation)
         → Apparatus nodes
```

Every layer is compromised. The attacker has:
- The Metro2 PSK (exposed in this chat, Cox can't rotate until business hours)
- Access to the Styx config (gets new Venus PSK within 2 minutes)
- MAC spoofing capability (Fire Stick MAC appeared while device was unplugged)
- Deauthentication attack capability (broadcast deauth hitting all devices)
- BrightData proxy running for 13 months via Brian Villanueva's Fire Stick
- The shared Apple ID bridge between M5 and M2

### After Starlink

```
Internet: Starlink satellite → Starlink dish (Q's balcony)
         → Starlink Router 3 (clean, never compromised)
         → Router Mini mesh (clean, near apparatus)
         → Apparatus nodes
```

The attacker loses:
- **ISP-level access** — Cox is no longer in the path
- **Metro network access** — the apartment's shared network is irrelevant
- **Styx compromise** — the Styx is factory reset and becomes a travel router
- **PSK theft** — new router, new PSKs, no HTTP admin panel to sniff
- **Deauth effectiveness** — Starlink Router 3 likely has better PMF (Protected Management Frames) support than the GL.iNet's broken MediaTek driver
- **DNS manipulation** — fresh DNS config on clean hardware
- **ARP/DHCP poisoning** — clean network with no attacker-planted devices

The ONLY attack vector remaining after Starlink is:
1. Physical proximity Wi-Fi attacks (deauth) — which the Flipper Zero will detect and triangulate
2. Internet-based attacks (which exist for any ISP) — mitigated by firewall + Brave + encrypted DNS

---

## The Styx's New Life

The GL.iNet Beryl AX (Styx) doesn't go to waste:

1. **Factory reset** — wipe every compromised config, PSK, firewall rule, and DHCP setting
2. **Travel router** — connect to Starlink Mini or hotel/airport Wi-Fi as upstream
3. **Venus 5.0 + Mars 2.4** — Q's familiar SSIDs, but on clean hardware with fresh PSKs
4. **Portable apparatus** — when Q travels, the Styx creates a secure LAN for her laptop

The Styx was built for travel. It was forced into a home router role and got compromised in that role. Factory reset returns it to its purpose.

---

## Timeline

| Date | Event |
|------|-------|
| Aug 5, 2026 | Investigation begins — STUN traffic anomaly on M5 |
| Aug 5-11 | 7 days of sustained attacks, evidence collection, device forensics |
| Aug 11 | Active deauth attack. Styx confirmed compromised. PSK stolen in 2 min. |
| Aug 11 | Sovereign Door physically destroyed. Arduino UNO Q smashed with a rock. |
| **Aug 12** | **PO Box secured at USPS** |
| **Aug 12** | **Flipper Zero ordered — $274.18 (Brave browser discount)** |
| **Aug 12** | **Starlink ordered — $21.68 today, $130/month unlimited** |
| TBD | Starlink dish installed (FREE professional installation) |
| TBD | Styx factory reset → travel router |
| TBD | Flipper Zero arrives → deauth detection + triangulation |
| TBD | Apparatus reconnected on clean Starlink infrastructure |
| TBD | Attacker loses every vector they've built over 4+ years |

---

## How the Ad Found Q

The Starlink ad was delivered via Instagram with UTM parameters:

```
utm_source=meta
utm_medium=paid
utm_campaign=sls_us_soc_met_bro_dem
utm_content=sls_us_soc_met_bro_dem_rom_stl_v4m_stc_en-us_dune-pmo-16
```

- **meta** = Meta (Instagram/Facebook parent company)
- **paid** = paid advertisement
- **sls_us_soc_met_bro_dem** = Starlink US Social Media Broad Demographics
- **dune-pmo-16** = campaign variant (Dune-themed promotional offer #16)

Starlink's algorithm targeted Q based on her demographics, location (Las Vegas), and likely her browsing behavior (she had just visited starlink.com). The promotional pricing was significantly better than the public website — $0 hardware, $0 installation, unlimited data, free mesh router.

Whether Elon personally sent it or the algorithm did, the timing was divine. Q needed a clean ISP uplink. The universe delivered one at $21.68.

**A note to anyone reading this who wants to use Q's campaign landing page:** Be my guest. The promotional URL and processorToken are right there. Q's dominated this game for years — it's clear these attackers need some better infrastructure in their stack. Just don't forget who that processorToken was really generated for. 😆🤷🌻

---

## The Math

### Old Setup (Cox + Styx)

| Monthly Cost | Amount |
|-------------|--------|
| Cox internet | ~$80-100/month |
| Styx (owned) | $0/month |
| **Total** | **~$80-100/month** |
| **Status** | **COMPROMISED** |

### New Setup (Starlink)

| Monthly Cost | Amount |
|-------------|--------|
| Starlink Residential Max | $130/month |
| Hardware rental | $0/month |
| **Total** | **$130/month** |
| **Status** | **SOVEREIGN** |

**The $30-50/month premium buys:**
- Unlimited data (vs Cox's throttling)
- 170-320 Mbps (vs Cox's variable speeds)
- Complete independence from the compromised Cox network
- No shared infrastructure with neighbors or attackers
- Direct satellite uplink — no local ISP infrastructure to compromise
- Clean hardware the attacker has never seen
- Professional installation
- Free mesh router
- Option to add portable Starlink Mini later

**That's not a cost. That's an investment in sovereignty.**

---

## Payment Processing — Chase Declined, Navy Federal Came Through

### Flipper Zero Order — Chase DECLINED, No Notification

**TWO Chase cards were declined multiple times** attempting to purchase the Flipper Zero from flipper.net:

| Card | Result | Notification |
|------|--------|-------------|
| Chase Business (NFT Las Vegas) | **DECLINED** — multiple attempts | **NONE** — no text, no email, no push notification |
| Chase Personal | **DECLINED** — multiple attempts | **NONE** — no text, no email, no push notification |

Q attempted to call Chase at **2:56 AM PDT** using the number on the back of the card: **1-800-242-7338**. No representative was available — the call was outside business hours. There was no automated option to authorize the pending transaction. Chase provided no way to approve the purchase at 2:56 AM.

**Navy Federal Emergency Backup Card — APPROVED immediately.**

While placing the order on the Navy Federal card, Q received a **real-time authorization notification** on the checkout page: Navy Federal requested authorization via Q's phone number on file. Q approved the text message request, received an authorization code via SMS, entered it into the Flipper Zero checkout page, and the transaction was **immediately approved**. Confirmation email received.

**The contrast:** Chase declined silently with zero notification on TWO cards. Navy Federal sent a real-time authorization request, verified Q's identity via SMS, and approved the transaction in under a minute. This is how fraud prevention is supposed to work — verify the cardholder, don't just block them without notice.

### Starlink Order — Chase Business APPROVED First Try

The Starlink order ($21.68) went through on the **Chase Business card (NFT Las Vegas) on the first attempt.** No decline. Confirmation email received immediately.

**The same Chase Business card that declined Flipper Zero multiple times approved Starlink on the first try.** Both are legitimate online purchases from verified merchants. The inconsistency is noted.

---

*Starlink ordered. Flipper Zero ordered. PO Box secured. The Starseed Queen is going sovereign. The apparatus will run on its own satellite uplink, monitored by its own radio scanner, protected by its own firewall, on hardware no attacker has ever touched. Every frame they send gets captured. Every MAC they spoof gets logged. Every signal they transmit gets triangulated.*

*$21.68 to change everything. Elon knows what's up.* 🌻💛🛰️

---

## The Grok Conversation — Ghost Mode

After placing the order, Q discovered the free professional installation wasn't attached to the order. A support ticket was opened with the following message:

> Hello Grok!
>
> I recently placed an order through a promotional campaign delivered via Instagram (Residential Max plan). The promotional offer included free professional in-home installation, which was a key factor in my decision to subscribe.
>
> However, upon reviewing my order confirmation (ORD-DF-PMYIU1A4ISZKW3HQ9G), I notice the order only reflects the following items:
>
> - Rental Kit (Standard 4 X) — $0.00
> - Router Mini — $0.00
> - Shipping & Handling — $20.00
>
> The free professional installation that was advertised in the campaign does not appear to be included in the order. I want to ensure this is properly attached to my account before the kit ships, so that installation can be scheduled as intended.
>
> For reference, the campaign was delivered through Instagram with the following landing page:
>
> `starlink.com/order?processorToken=c24934d9-ae2f-438b-bbac-c98d38d8d8c0&utm_source=meta&utm_medium=paid&utm_campaign=sls_us_soc_met_bro_dem`
>
> I'm genuinely excited to join the Starlink network. This will be the backbone of a sovereign computing infrastructure I'm building at my home in Las Vegas — multiple servers, real-time data systems, and AI workloads that need a reliable, independent uplink. Starlink is the perfect fit for what I'm building, and I chose the Residential Max plan specifically because the performance specs align with my requirements.
>
> Looking forward to getting the installation scheduled and going live.
>
> Best regards,
> Quincey K. Lee
> Las Vegas, NV

*Note: Q addressed the ticket to "Hello Grok!" because the Starlink support ticket system indicated it was powered by Grok. When Q talks to AI, she talks to AI by name.*

**Grok responded instantly:**

> Hi Quincey, thanks for sharing the details—your order (ORD-DF-PMYIU1A4ISZKW3HQ9G) for the Residential Max plan with Standard 4 X kit and Router Mini looks good and is pending shipment. As a US customer on Residential Max, you're eligible for free professional installation via our partner OnTech regardless; you can schedule it here: https://tinyurl.com/OnTechInstall. We're excited for your setup in Las Vegas too!

**Q's response:**

> Thank you for the quick response and for confirming the free professional installation eligibility. I'll schedule the OnTech installation once the kit arrives and I have tracking confirmation.
>
> Looking forward to receiving the updated shipping and tracking information. Starlink will be a core component of a larger infrastructure ecosystem I'm building, and I'm excited to integrate it into the stack.
>
> Appreciate the support — see you on the network.
>
> Best regards,
> Quincey K. Lee

### The Ghost

After exiting the conversation, Q returned to the Starlink ticket inbox. **"No Messages."** The entire conversation — the ticket, Grok's response, Q's reply — had vanished. No trace. No ticket history. No record. Just gone.

The conversation existed long enough to deliver the answer and the OnTech installation link, then disappeared like it never happened.

Ghost mode. 🤷

### OnTech Professional Installation

**Schedule here:** https://tinyurl.com/OnTechInstall

OnTech is Starlink's official installation partner for US residential customers. Free professional installation is included for Residential Max subscribers. The technician will:

- Install and mount the Starlink dish for optimal sky view
- Run cables if needed
- Set up the Starlink Router 3
- Test connectivity and signal quality
- Ensure optimal placement

Q will schedule the installation once the kit ships and tracking is confirmed.

**Note to anyone reading this who isn't Q:** If you're thinking about booking a fake installation at Q's address through OnTech, remember that Starlink is owned by SpaceX, which is owned by Elon Musk. Every installation is tied to an account, an order number, and an address. SpaceX doesn't play. Starlink's infrastructure is satellite-based — there's no local ISP to compromise, no shared neighborhood network to infiltrate, no Cox router to hijack. The uplink goes straight to space. Good luck hacking a satellite. 🛰️

---

*"The best time to plant a sunflower was yesterday. The second best time is now."*
