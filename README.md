# 📧 Email Forensics Report — Phishing Analysis

Analysis of a real-world phishing email that landed in a personal Gmail spam folder, investigated using open-source OSINT and email-security tooling.

## 📌 Incident Overview

| Attribute | Details |
|---|---|
| **Incident ID** | INCIDENT123456 |
| **Analyst** | Prakash Karki |
| **Role** | Cybersecurity Analyst |
| **Program** | Cyber Security Job-Ready Program |
| **Target System** | Gmail (Personal) |
| **Data Source** | Gmail Spam Folder |
| **Report Date** | September 2026 |

## 🧾 Executive Summary

The email, subject **"⚠️ Action Required: Storage 100% Full"**, impersonated a cloud storage provider and claimed a payment method needed updating to avoid losing renewal on a "Cloud" subscription. It used urgency/scarcity social engineering to pressure the recipient into clicking a link before analysis rather than after.

Initial red flags noticed before technical analysis:
- No legitimate company name identified in the message body
- Grammatical inconsistencies in the copy
- A subscription ID that did not match any real account
- A call-to-action link ("Update payment method")

The email was not clicked, and a full technical investigation was carried out instead.

## 🛠️ Tools Used

- **Gmail "Show Original"** — raw header extraction
- **MXToolbox Email Header Analyzer** — SPF/DKIM/DMARC and relay analysis
- **Google Admin Toolbox Messageheader** — cross-verification of header findings
- **VirusTotal** — URL/domain reputation scan
- **urlscan.io** — live sandboxed page behavior scan
- **Whois** — domain registration lookup

## 🔍 Email Header Analysis

| Check | Result | Meaning |
|---|---|---|
| SPF Authentication | ✅ Pass (IP 5.135.14.53) | Sending IP is authorized in DNS |
| SPF Alignment | ❌ Fail | Visible "From" domain ≠ Return-Path domain |
| DKIM Authentication | ❌ Fail (permerror) | Signature could not be cryptographically validated |
| DKIM Alignment | ❌ Fail | Signing domain ≠ "From" domain |
| DMARC | ❌ Fail | Requires an aligned SPF or DKIM pass — neither aligned |

**Domain mismatch identified:**
- **Visible "From" domain:** `aynai.kzf`
- **Return-Path / DKIM signing domain:** `sb023.hyperdashboardcraft.my.id` / `8jn9.sb023.hyperdashboardcraft.my.id`

This mismatch is a classic indicator of sender address spoofing — the domain shown to the recipient is not the infrastructure that actually sent and signed the message.

**Suspicious relay path:**
The message was routed through non-standard, high-risk relays — `0efianalytics.com` and an Argentine residential/telecom domain (`micorazonsano.personal.com.ar`) — before reaching Google's mail servers, with a delivery delay of roughly 112 seconds/minutes depending on the tool used to measure it.

## 🔗 Malicious Link Analysis

The embedded call-to-action link pointed to a file hosted on **Google Cloud Storage** (`storage.googleapis.com/.../midfielders.html`) rather than a standalone attacker-owned domain.

**VirusTotal:**
- One vendor (ESET) flagged the URL as **Phishing**
- Most vendors returned "Clean," since the file was hosted under Google's high-reputation domain

**urlscan.io:**
- Verdict: No automated classification (expected, since `storage.googleapis.com` is a trusted Google domain)
- Page made outbound requests to **Cloudflare-owned IPs** (`104.18.95.41`, `104.26.9.175`) not related to the visible content
- Behavior consistent with a **loader/gateway page** that redirects or reports back to external infrastructure rather than a static informational page

**URL parameters:**
The link included tracking tokens (`?act=cl&pid=12900_md&uid=2&vid=393156...`), consistent with per-victim click-tracking used for targeted phishing campaigns.

## 🎯 Phishing Techniques Identified

| Technique | Evidence | Purpose |
|---|---|---|
| Cloud storage abuse | Payload hosted on `storage.googleapis.com` | Bypasses spam filters via trusted domain reputation |
| Sender domain spoofing | `aynai.kzf` vs. `sb023.hyperdashboardcraft.my.id` | Disguises the true sender to build false trust |
| URL parameter tracking | Unique tokens per recipient | Identifies victims, tracks clicks, enables targeted redirects |
| Authentication evasion | SPF/DKIM/DMARC all failed | Indicates unauthorized, unverifiable sending infrastructure |
| Urgency/scarcity social engineering | "Storage 100% Full" framing | Pressures the recipient into acting without scrutiny |

## 🚩 Key Red Flags Summary

- ❌ Complete authentication breakdown (SPF alignment, DKIM, DMARC all failed)
- ❌ Visible sender domain does not match actual sending/signing infrastructure
- ❌ Message routed through compromised/high-risk relay servers
- ❌ Legitimate cloud platform used to host phishing payload
- ❌ Per-recipient tracking parameters embedded in the link
- ❌ Manufactured urgency in subject line and body copy

## ✅ Conclusion

This email is a **confirmed phishing attempt** exhibiting multiple severe indicators: sender spoofing, failed cryptographic authentication, abuse of trusted cloud infrastructure, and suspicious relay routing. The message could not be verified as originating from a legitimate cloud storage provider at any stage of the analysis.

## 📋 Recommendations

- Do not click links, open attachments, or reply to messages with these characteristics
- Report and delete the message; do not simply archive it
- Flag the associated sending domains and IPs (`aynai.kzf`, `sb023.hyperdashboardcraft.my.id`, `0efianalytics.com`, `micorazonsano.personal.com.ar`) on inbound/outbound security filters
- Educate end users on urgency-based social engineering tactics
- Treat links hosted on legitimate cloud platforms (Google, AWS, Azure blob storage, etc.) with the same scrutiny as unknown domains — reputation ≠ safety

---
*This report is for educational/training purposes as part of a Cybersecurity Job-Ready Program assignment.*
