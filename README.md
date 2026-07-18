# Home Network Security Assessment

A full-lifecycle grey-box vulnerability assessment of a private residential network — scoped and
authorized under a signed Rules of Engagement, executed with Nmap, inSSIDer, and an authenticated
Qualys VMDR scan, and reported with prioritized, actionable remediation guidance.

**44 confirmed vulnerabilities across 15 responding hosts on a 22-device network.** The perimeter
held; the risk was inside.

---

## Overview

This project documents a security assessment performed against a home network protected by a
pfSense firewall, Suricata IDS, and pfBlockerNG filtering. It was completed as the capstone
deliverable for CSIA 440: Cyber Testing and Penetration at Columbia Basin College.

The engagement produced two findings worth leading with:

1. **External reconnaissance found nothing.** No services indexed on the public interface, no PTR
   record, no unsolicited inbound traffic passing the firewall. The perimeter was sound.
2. **Internal scanning found plenty.** Both Critical findings and the largest concentration of
   Medium findings sat on a single aging network printer — and the next tier of risk was on the
   wireless management interfaces I had configured myself.

That contrast is the point of the project. Perimeter security and interior security are different
problems, and confirming the first tells you very little about the second.

> ### Scope and authenticity
>
> This was an **academic engagement on a residence I had explicit written authorization to test**,
> under a signed Rules of Engagement executed with the property owner before any activity began.
> It is not a commercial engagement and is not presented as one.
>
> The ROE prohibited exploit frameworks and destructive testing, so **no exploitation was
> performed**. Accurately described, this is a grey-box vulnerability assessment and
> reconnaissance exercise conducted under a penetration-test ROE. Findings represent confirmed
> vulnerability *presence*, not demonstrated exploitability.
>
> All materials here have been **de-identified for public release** — see
> [De-identification](#de-identification) below.

---

## Repository Contents

| File | Description |
|---|---|
| [`reports/security-assessment-report.md`](reports/security-assessment-report.md) | The full assessment report — inventory, OSINT, physical security, wireless, Nmap, vulnerability findings, methodology, lessons learned. |
| [`reports/rules-of-engagement.md`](reports/rules-of-engagement.md) | The signed ROE — scope, boundaries, authorized tooling, constraints, authorization statement. |
| [`reports/vulnerability-scan-summary.md`](reports/vulnerability-scan-summary.md) | De-identified summary of the authenticated Qualys VMDR scan: configuration, severity distribution, confirmed findings by host, analysis. |

**Start with the ROE if you want to see how the engagement was scoped; start with the report if
you want to see what it found.**

---

## Methodology

A structured four-phase approach:

**1 — Pre-Engagement.** Scope, constraints, and authorization defined in a signed ROE with the
network owner. Administrative access to the firewall and AP interfaces established for grey-box
testing.

**2 — Reconnaissance.** Passive OSINT across five sources (Shodan, Whois/ARIN, Have I Been Pwned,
reverse DNS, search-engine reconnaissance). Wireless enumeration with inSSIDer. Initial asset
inventory built from DHCP leases and AP device lists. Physical security controls documented.

**3 — Active Scanning.** Host discovery and service/OS detection with Nmap across the `/24`. A
Qualys VMDR Virtual Scanner Appliance stood up on VirtualBox and deployed on the LAN for an
authenticated vulnerability scan. No exploitation, per the ROE.

**4 — Documentation & Analysis.** Findings correlated across tools, risk-rated, prioritized, and
written up. Executive summary written last.

**Tools:** Nmap · Qualys VMDR · inSSIDer · pfSense · pfBlockerNG · Suricata · VirtualBox

---

## Key Findings

| ID | Finding | Risk | Summary |
|---|---|---|---|
| **F-1** | Network printer with writable SNMP and excessive exposed services | **Critical** | Default SNMP community strings accepted with WRITE access (QID 78031) plus end-of-life SNMP v1/v2c (QID 105459), on a host exposing 11 open TCP ports including raw print (9100), SMB (445), and NetBIOS (139). An ideal LAN foothold, credential store, and pivot point. |
| **F-2** | Wireless management interfaces accept cleartext credentials | **High** | All three mesh nodes accept plain-text HTTP Basic Auth (QID 86763, 45242) and support obsolete TLS 1.0/1.1. Compromise of these interfaces yields control of the entire wireless infrastructure. One architectural weakness, replicated three times. |
| **F-3** | Outdated firmware and incomplete asset inventory | **Medium** | Firewall two versions behind with an EOL DHCP backend and a self-signed GUI certificate; two devices unidentifiable beyond their manufacturer OUI. |
| **P-1** | Core network hardware in an unlocked interior closet | **Medium** | Strong perimeter controls, but anyone reaching the interior — intruder or invited guest — has unrestricted physical access to the firewall and primary AP. |
| **W-1 / W-2** | Open casting SSID; WPA2 rather than WPA3 | **Low** | Minor internal attack surface and a modernization gap on Wi-Fi 5-era hardware. |

**Severity distribution (confirmed):** 2 Critical · 0 High · 18 Medium · 24 Low = **44 confirmed**

*Note: the raw scan reports 310 total detections, but 257 of those are information-gathered items
and 9 are unconfirmed potentials. Reporting the raw total as "vulnerabilities" would overstate the
result by roughly 7×. See the [scan summary](reports/vulnerability-scan-summary.md) for the full
breakdown.*

---

<!-- ═══════════════════════════════════════════════════════════════════════════
     ⚠️  DRAFT SECTION — DO NOT PUBLISH AS-IS  ⚠️
     Every bracketed value below is a placeholder. Fill in only remediation you
     have ACTUALLY completed, with real dates and real verification results.
     Delete any row you have not done. If you have not started remediation yet,
     delete this entire section rather than publishing unverified claims.
     ═══════════════════════════════════════════════════════════════════════════ -->

## Remediation *(draft — complete before publishing)*

An assessment that stops at findings is only half the work. This section tracks what was actually
fixed and how it was verified.

| Finding | Action Taken | Date | Verification |
|---|---|---|---|
| F-1 — Printer | *[e.g. device decommissioned and removed from network / SNMP disabled, unused print + file protocols disabled, admin password set, firmware updated]* | *[date]* | *[e.g. re-scan confirmed 0 open ports / Nmap re-scan shows N ports remaining]* |
| F-2 — AP management | *[e.g. HTTPS-only management enforced; TLS 1.0/1.1 disabled; admin credentials rotated]* | *[date]* | *[e.g. re-scan confirms QID 86763 no longer detected]* |
| F-3 — Firmware / inventory | *[e.g. pfSense updated 25.07.1 → 26.03.1; DHCP migrated ISC → Kea; GUI certificate replaced; both unknown devices identified and given static mappings]* | *[date]* | *[e.g. re-scan; asset inventory now complete at N/N devices]* |
| P-1 — Physical | *[e.g. network hardware relocated to lockable enclosure]* | *[date]* | *[e.g. visual verification]* |
| W-1 / W-2 — Wireless | *[e.g. IoT VLAN created; WPA3 transition mode enabled / hardware replacement scheduled]* | *[date]* | *[date]* |

**Post-remediation re-scan:** *[Summarize: confirmed findings reduced from 44 to N. Attach or
summarize the delta.]*

<!-- END DRAFT SECTION -->

---

## Skills Demonstrated

- **Engagement scoping and authorization** — drafting and executing a Rules of Engagement with
  defined in-scope and out-of-scope systems, authorized tooling, client constraints, and a written
  authorization statement.
- **Reconnaissance and OSINT** — multi-source passive intelligence gathering, including assessment
  of social-engineering exposure, with correct interpretation of negative results.
- **Network enumeration** — host discovery, service and OS fingerprinting, MAC OUI and DHCP
  correlation for asset identification.
- **Vulnerability management** — deploying and operating a Qualys VMDR virtual scanner appliance
  for authenticated internal scanning; triaging confirmed vs. potential vs. informational
  detections.
- **Wireless assessment** — RF environment analysis, mesh topology inference from BSSID and
  channel data, encryption posture evaluation.
- **Risk analysis and communication** — impact/likelihood matrix, prioritized findings, and
  remediation guidance written for the person who has to act on it.
- **Technical writing** — a structured, professional deliverable with an executive summary that
  leads with what matters.

---

## De-identification

These materials have been de-identified for public release. Removed or generalized: the public WAN
address and derived geolocation, the residence location, names of the property owner and household
members, personal email addresses and OSINT results, all photographs, raw wireless captures
containing neighboring third-party networks, and vendor account identifiers.

**Deliberately preserved:** all RFC 1918 private addressing, vulnerability findings, QIDs, CVE
references, port lists, and technical analysis. Private addresses are non-routable and reveal
nothing about a reachable target; stripping them would make the analysis unreadable without
improving anyone's security.

The goal is to remove what would let a reader locate and target the assessed network — not to
obscure authorship or dilute the technical content. Full notes are in the
[report appendix](reports/security-assessment-report.md#appendix-de-identification-notes).

The complete 256-page Qualys vendor export is retained privately and is available on request; a
de-identified summary is published in its place.

---

## Contact

**Michael Keuhlen** — m.keuhlen@proton.me

*CompTIA A+ · CompTIA Security+ · Qualys VMDR Specialist*
