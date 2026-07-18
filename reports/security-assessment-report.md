# Network Security Assessment Report

**Grey-Box Vulnerability Assessment — Private Residence Home Network**

| Field | Value |
|---|---|
| Author | Michael Keuhlen |
| Institution | Columbia Basin College |
| Course | CSIA 440: Cyber Testing and Penetration |
| Report Date | June 14, 2026 |
| Original Classification | Confidential — Academic Use Only |
| Public Release | De-identified copy |

*Prepared in fulfillment of the CSIA 440 limited network assessment requirement, conducted
under the authorization of a signed Rules of Engagement.*

> **Public release note.** This is a de-identified copy of an academic report. Private
> individuals' names, device hostnames, the public WAN address, the residence location,
> third-party (neighboring) wireless networks, personal OSINT results, and physical
> security photographs have been removed or generalized. Private RFC 1918 addressing,
> vulnerability findings, QIDs, CVE references, and all technical analysis are unmodified.
> See [Appendix: De-identification Notes](#appendix-de-identification-notes).

---

## 1. Introduction

### 1.1 Engagement Overview

This report documents the findings of a limited network security assessment conducted against a
private residence's home network. The assessment was performed as a required deliverable for
CSIA 440: Cyber Testing and Penetration at Columbia Basin College. All activities were conducted
within the scope and constraints defined in the Rules of Engagement (ROE) signed by both the
client (the network owner) and the tester prior to the start of testing.

The objective was to assess the security posture of the home network by enumerating connected
devices, identifying open services and vulnerabilities, evaluating physical and wireless security
controls, and producing prioritized, actionable recommendations for remediation.

**No active exploitation was performed.** All vulnerabilities were identified and documented for
remediation purposes only.

### 1.2 Dates of Testing

| Phase | Dates |
|---|---|
| Engagement Start | May 27, 2026 |
| Reconnaissance / OSINT | May 27 – May 31, 2026 |
| Active Scanning (Nmap) | June 1 – June 5, 2026 |
| Vulnerability Scanning | June 1 – June 7, 2026 |
| Report Assembly | June 7 – June 10, 2026 |
| Engagement End | June 10, 2026 |
| All Work Complete | NLT June 14, 2026 |

### 1.3 Testing Team

| Name | Role | Relevant Experience |
|---|---|---|
| Michael Keuhlen | Lead Tester / Student | BAS coursework in Cybersecurity. CompTIA A+, CompTIA Security+, and Qualys VMDR Specialist certifications. |

### 1.4 Purpose of the Test

The purpose of this engagement was to identify security vulnerabilities in the home network
infrastructure, assess the effectiveness of existing security controls (firewall, IDS, physical
controls), and produce findings and recommendations to improve the overall security posture. The
test was authorized by the network owner in accordance with the signed ROE.

### 1.5 Type of Test

| Attribute | Value | Notes |
|---|---|---|
| Assessment Type | Vulnerability assessment under a penetration-test ROE | Reconnaissance, enumeration, and authenticated vulnerability scanning. |
| Knowledge Level | Grey-box | Tester held administrative access to the pfSense and Orbi interfaces; internal layout was known. |
| Exploitation | None (scan only) | Vulnerabilities identified and documented; no active exploitation, per ROE. |

> **Scoping note.** This engagement is accurately characterized as a grey-box vulnerability
> assessment and reconnaissance exercise conducted under a penetration-test ROE. Because the
> ROE prohibited exploit frameworks and any destructive testing, no exploitation or
> post-exploitation activity was performed. Findings represent confirmed vulnerability
> *presence*, not demonstrated exploitability.

---

## 2. Network Details

### 2.1 Network Topology

The network is a single flat `/24` LAN behind a pfSense firewall.

```
                          [ Internet ]
                                |
                             Fiber
                                |
                      [ ISP Fiber ONT ]          (out of scope — ISP owned)
                                |
                            Ethernet
                                |
              WAN: [REDACTED]  [ Netgate 1100 — pfSense ]  LAN: 192.168.1.1
                                |
                            Ethernet
                                |
                  [ Orbi RBR50 — Access Point Mode ]  192.168.1.110
                       /            |            \
                 Wireless        Wireless       Wireless
                    /               |               \
       [ RBS50 Satellite ]     [ Main floor ]    [ RBS50 Satellite ]
          192.168.1.115          clients           192.168.1.112
                |                                        |
          Upstairs clients                        Basement clients

  LAN subnet:  192.168.1.0/24
  DHCP pool:   192.168.1.100 – 192.168.1.199
```

The pfSense firewall (Netgate 1100) serves as the network boundary. The Netgear Orbi RBR50
operates in Access Point mode with two satellite units extending wireless coverage.

### 2.2 System Inventory

The following table inventories all in-scope systems identified during the engagement. Device
identities were determined from pfSense DHCP leases, MAC address manufacturer lookups, and Nmap
fingerprinting. The network comprises 22 devices, including network infrastructure, personal
computers, mobile devices, and a range of smart-home/IoT devices.

> Hostnames in this public copy have been replaced with functional labels. See the
> [de-identification notes](#appendix-de-identification-notes).

| Label | IP Address | OS / Description | Purpose |
|---|---|---|---|
| `fw-01` | 192.168.1.1 | FreeBSD 15.0 (Netgate 1100, arm64) | Gateway / Firewall |
| `ap-01` | 192.168.1.110 | Netgear Orbi firmware | Wireless Router (AP) |
| `ap-02` | 192.168.1.112 | Netgear Orbi firmware | Mesh Satellite (Basement) |
| `ap-03` | 192.168.1.115 | Netgear Orbi firmware | Mesh Satellite (Upstairs) |
| `WORKSTATION-01` | 192.168.1.116 | Windows 11 | Personal Laptop |
| `WORKSTATION-02` | 192.168.1.124 | Windows 11 | Personal Laptop |
| `WORKSTATION-03` | 192.168.1.127 | Windows 11 | Personal Laptop |
| `WORKSTATION-04` | 192.168.1.126 | Windows 11 | Personal PC |
| `PRINTER-01` | 192.168.1.117 | VxWorks (HP Officejet Pro) | Network Printer |
| `MOBILE-01` | 192.168.1.136 | Android | Smartphone |
| `MOBILE-02` | 192.168.1.135 | Android | Smartphone |
| `MOBILE-03` | 192.168.1.119 | Apple iOS | Tablet |
| `WEARABLE-01` | 192.168.1.122 | Wear OS | Smartwatch |
| `IOT-CAM-01` | 192.168.1.109 | Embedded Linux (Ring) | Security Camera |
| `IOT-CAM-02` | 192.168.1.114 | Embedded Linux (Ring) | Video Doorbell / Camera |
| `IOT-LOCK-01` | 192.168.1.129 | Embedded firmware | Smart Door Lock |
| `IOT-MEDIA-01` | 192.168.1.120 | Google Cast (Chromecast) | Streaming Device |
| `IOT-IRRIG-01` | 192.168.1.106 | Orbit Irrigation (lwIP embedded) | Smart Irrigation Controller |
| `IOT-HVAC-01` | 192.168.1.103 | Resideo (Honeywell Home) | Smart Thermostat |
| `IOT-HVAC-02` | 192.168.1.113 | Resideo (Honeywell Home) | Smart Thermostat |
| `UNKNOWN-01` | 192.168.1.111 | Universal Global Scientific Ind. (OUI only) | Unidentified IoT device (likely a second streaming device) |
| `UNKNOWN-02` | 192.168.1.123 | Chongqing Fugui Electronics (OUI only) | Unidentified IoT device (likely a second smartwatch) |

**Note:** Two devices (`UNKNOWN-01` and `UNKNOWN-02`) could not be conclusively identified
beyond their manufacturer OUI. This gap is documented as an asset-management finding.

---

## 3. Executive Summary

This assessment evaluated a home network protected by a pfSense firewall, a Suricata intrusion
detection system (IDS), and pfBlockerNG content filtering. External reconnaissance confirmed a
strong perimeter: the network exposes no services to the public internet, and the firewall
effectively blocks unsolicited inbound traffic.

**The most significant risks identified are therefore internal**, concentrated in an aging
networked printer, the wireless management interfaces, and device lifecycle and inventory
management.

A total of **44 confirmed vulnerabilities** were identified by a Qualys VMDR authenticated scan
across 15 responding hosts: **2 Critical (Severity 5), 0 High (Severity 4), 18 Medium
(Severity 3), and 24 Low (Severity 2)**. The three highest-priority risks are presented below.

### 3.1 Risk Analysis Matrix

| Impact ↓ / Likelihood → | 1 – Rare | 2 – Unlikely | 3 – Possible | 4 – Likely | 5 – Almost Certain |
|---|---|---|---|---|---|
| **5 – Critical** | | | | **F-1** | |
| **4 – High** | | | | | |
| **3 – Medium** | | | **F-2** | | |
| **2 – Low** | | **F-3** | | | |
| **1 – Negligible** | | | | | |

- **F-1** Network Printer — Critical / Likely
- **F-2** Cleartext Wireless Management — High / Possible
- **F-3** Outdated Firmware & Inventory Gaps — Medium / Unlikely

### 3.2 Top Three Findings

#### Finding F-1: Network Printer with Writable SNMP and Excessive Exposed Services

| Field | Detail |
|---|---|
| **Affected Host** | `PRINTER-01` — HP Officejet Pro, 192.168.1.117 |
| **Risk Level** | **Critical (5)** |

**Description.** The HP networked printer is the single highest-risk device on the network.
Qualys identified 14 vulnerabilities on this host, including two Critical (Severity 5) findings:
the SNMP service accepts the default community strings `public` and `private` and permits WRITE
access (**QID 78031**), and it runs end-of-life SNMP v1/v2c, which transmits community strings in
cleartext (**QID 105459**). Nmap separately confirmed an unusually broad attack surface of 11 open
ports, including unauthenticated raw printing (9100), SMB (445), NetBIOS (139), and web
administration (80/443).

The combination of writable SNMP, file-sharing services, and a web admin interface makes this
device an ideal foothold for an attacker on the LAN: it can be reconfigured, used to harvest
stored scan-to-email/SMB credentials, or leveraged as a persistent pivot point. Printers are also
among the least frequently patched devices on any network, making them a primary initial target.

**Recommendation.** Treat the printer as the top remediation priority.

1. Disable SNMP entirely if unused; if required, disable v1/v2c and configure SNMPv3 with
   authentication.
2. Disable unused print/file protocols — raw 9100, LPD (515), and SMB (139/445).
3. Set a strong administrator password on the embedded web server and force HTTPS.
4. Apply the latest vendor firmware.
5. Isolate the printer on a dedicated IoT/print VLAN so a compromise cannot reach personal
   computers or infrastructure.

---

#### Finding F-2: Wireless Management Interfaces Accept Cleartext Credentials

| Field | Detail |
|---|---|
| **Affected Hosts** | `ap-01`, `ap-02`, `ap-03` — 192.168.1.110, .112, .115 |
| **Risk Level** | **High (4)** |

**Description.** All three Orbi mesh nodes expose web-based management interfaces that accept
credentials over unencrypted channels. Qualys flagged plain-text HTTP Basic Authentication
(**QID 86763**) and a remote management service accepting unencrypted credentials (**QID 45242**)
on each node, along with support for obsolete TLS 1.0 and TLS 1.1 (**QID 38628**, **QID 38794**).

Because these are the administrative interfaces for the entire wireless network, an attacker
positioned on the LAN could intercept administrator credentials in transit and gain full control
of the wireless infrastructure, enabling traffic redirection, rogue configuration, or
network-wide compromise. The exposure is multiplied across three devices.

**Recommendation.**

1. Access AP administration only over HTTPS; disable plain HTTP management where the firmware
   allows.
2. Disable legacy TLS 1.0/1.1 in favor of TLS 1.2+.
3. Confirm the administrator password is strong, unique, and changed from any default.
4. Because the RBR50 is a Wi-Fi 5-era platform that may not support modern TLS or WPA3, evaluate
   the mesh system for replacement as part of a hardware modernization plan.

---

#### Finding F-3: Outdated Firmware and Incomplete Asset Inventory

| Field | Detail |
|---|---|
| **Affected Host** | `fw-01` — 192.168.1.1; network-wide |
| **Risk Level** | **Medium (3)** |

**Description.** Two related lifecycle issues were observed.

First, the pfSense firewall is running version 25.07.1, while version 26.03.1 is available. Its
bundled ISC DHCP server has reached end of life and requires migration to the Kea DHCP backend.
Qualys additionally flagged an invalid/self-signed management certificate and an NTP
information-disclosure issue on the gateway.

Second, two devices on the network (`UNKNOWN-01` and `UNKNOWN-02`) could not be conclusively
identified beyond their hardware manufacturer, indicating a gap in asset inventory. Unmanaged or
unpatched devices are common entry points for attackers, and a foothold on an unknown device may
go undetected.

**Recommendation.**

1. Apply the available pfSense update (25.07.1 → 26.03.1) after reviewing release notes and
   backing up the configuration.
2. Migrate the DHCP backend from ISC to Kea per the pfSense advisory.
3. Replace the default self-signed GUI certificate with a properly issued certificate.
4. Identify the two unknown devices and assign descriptive static DHCP mappings to maintain a
   complete, current asset inventory.

---

## 4. Technical Summary

### 4.1 Reconnaissance (OSINT)

Five open-source intelligence sources were used to assess the network's external exposure and the
tester's personal digital footprint. The results show a strong external posture — no exposed
services, minimal identity disclosure via network records — contrasted with a meaningful personal
data footprint that could support social engineering attacks.

> **Redaction note.** The original report tabulated specific OSINT results, including the public
> WAN address, a legacy personal email address, breach counts, a username, and data-broker
> aggregation results covering household members and relatives. Those values are withheld from
> this public copy. The methodology and the analytical conclusions are preserved in full.

| Tool / Source | What Was Searched | Findings | Security Implications |
|---|---|---|---|
| **Shodan.io** | Public WAN IP | No results returned; no open ports or services indexed. | Confirms the pfSense firewall blocks unsolicited inbound traffic. No externally exposed attack surface — a strong external security posture. |
| **Whois / ARIN** | Public WAN IP | IP registered to the residential fiber ISP; geolocated to the ISP's regional service area. | ISP and approximate region are discoverable, but no personally identifying subscriber information is exposed. Registration separation between ISP and subscriber is a net positive for privacy posture. |
| **Have I Been Pwned** | Three email addresses (academic, professional, personal) | Academic and professional addresses returned 0 breaches. A legacy personal webmail address appeared in more than a dozen breach corpora spanning 2012–2025. | The breached personal account is a credential exposure risk. If reused passwords are tied to device/cloud accounts (AP admin, camera, smart lock), credential stuffing could enable account takeover. |
| **Reverse DNS (MXToolbox)** | PTR record for the WAN IP | No PTR record published; reverse lookup returned no record via the ISP nameserver. | No hostname information disclosed via reverse lookup. Minimal information disclosure, consistent with residential service. |
| **Search engine reconnaissance** | Tester name + region; surname on professional networks; personal email address; username correlation | Data-broker profiles expose name, age, address history, phone, and relatives; obituary and alumni data confirm family/school ties; public social media activity found. No username-to-identity linkage found. | Aggregated public data forms a viable social-engineering profile enabling targeted phishing and password-recovery attacks. Recommend data-broker opt-outs and tightened privacy settings. |

**Overall**, external reconnaissance returned no exploitable network exposure. The primary OSINT
risk is personal: a breached legacy email address, combined with an aggregated public profile,
creates a realistic social-engineering pathway — underscoring the importance of unique passwords,
MFA, and data-broker record removal.

### 4.2 Physical Security Assessment

The residence employs multiple layered physical controls. The perimeter is well secured; the one
notable gap is the interior storage of the network infrastructure, documented as a finding below.

> **Figures omitted.** The original report included photographs of each control and of the network
> equipment location. These have been removed from the public copy, as they depict a private
> residence and its access-control hardware.

**Physical Control 1 — Electronic Smart Deadbolt (Primary Entrance).**
The primary entry is secured by an electronic smart deadbolt with keypad PIN entry and a
traditional keyway backup. This is a layered access control: entry requires either a PIN or a
physical key, and the lock re-engages automatically. Notably, this device is also a
network-connected asset (`IOT-LOCK-01`), making it both a physical control and an in-scope network
device. As a physical control it is effective; residual risk lies in its digital attack surface,
reinforcing the need for current firmware and a secure associated account.

**Physical Control 2 — Keyed Deadbolt Securing the ISP Network Equipment.**
The garage, which houses the ISP's Optical Network Terminal (ONT) where fiber service enters the
residence, is secured by a keyed deadbolt. This represents effective defense-in-depth: the
network's physical entry point is not directly accessible from outside without first defeating
the garage lock. The ONT itself is ISP-owned and was treated as out of scope per the ROE, but its
physical protection is a legitimate part of the residence's overall security posture.

**Physical Control 3 — Guard Animals.**
The residence is protected by two large working-breed dogs. Guard animals are a recognized
physical security measure functioning as both detection and deterrence. As detection, they
provide immediate audible alerting to unauthorized entry or unfamiliar presence — effectively a
biological intrusion-detection system that requires no power and cannot be disabled remotely. As
deterrence, their visible and audible presence discourages would-be intruders before an attempt
is made. This control compensates for gaps in electronic monitoring and provides 24/7 coverage of
the interior, including the area near the unsecured network closet discussed below.

**Physical Control 4 — Perimeter Fencing and Locked Gate.**
Backyard access is controlled by a six-foot privacy fence with a latching, locking gate, limiting
approach to the rear of the residence and the backyard-facing entry points.

#### Finding P-1: Network Infrastructure in Unsecured Interior Closet

| Field | Detail |
|---|---|
| **Risk Level** | **Medium (3)** |

While the residence perimeter is well secured through multiple layered controls, the assessment
identified one notable physical security gap: the core network infrastructure — the firewall
appliance and the primary wireless access point — is located in an unlocked interior closet.

Although the strong perimeter controls significantly reduce the likelihood of an unauthorized
outsider reaching this location, any individual who gains entry to the residence — whether an
intruder who defeats the perimeter or an invited guest — would have unrestricted physical access
to the network's core hardware. Physical access to this equipment would permit a factory reset,
direct console access to the firewall, or the covert installation of a rogue device on the LAN.

**Recommendation.** Relocate the network hardware to a lockable cabinet or closet, or install a
keyed lock on the existing closet door, completing a defense-in-depth approach.

### 4.3 Wireless Network Scan

A wireless reconnaissance scan was conducted using inSSIDer on a Wi-Fi 6E adapter.

> **Redaction note.** The original report included full inSSIDer screenshots enumerating dozens
> of neighboring residential networks by SSID, encryption, channel, and signal strength. Those
> networks were explicitly **out of scope** per the ROE and belong to uninvolved third parties;
> the raw captures and neighboring SSIDs are withheld from this public copy. Aggregate
> comparison figures are retained because they support the RF analysis.

#### Table 4.3.1 — Strongest Detected Networks (Assessed Network)

| SSID / Node | Encryption | Band / Ch | Signal | Max Rate |
|---|---|---|---|---|
| `[HOME-SSID]` (RBR50) | WPA2-Personal | 2.4 + 5 / 1, 48 | −37 to −41 dBm | 866.7 Mbps |
| *[Hidden]* (RBR50 backhaul) | WPA2-Personal | 2.4 / 1 | −39 dBm | 144.4 Mbps |
| *[Hidden]* `38:94:ED:**:**:**` (node) | WPA2-Personal | 5 / 155 | −43 dBm | 1,733.3 Mbps |
| *[Hidden]* `38:94:ED:**:**:**` (node) | WPA2-Personal | 5 / 155 | −54 dBm | 1,733.3 Mbps |
| *[Hidden]* `38:94:ED:**:**:**` (node) | WPA2-Personal | 5 / 155 | −65 dBm | 1,733.3 Mbps |

#### Table 4.3.2 — Neighboring Networks (Aggregate Comparison)

| Metric | Value |
|---|---|
| Neighboring networks detected | ~60 distinct SSIDs |
| Strongest neighboring signal | −82 dBm (Wi-Fi 6 / `ax` capable, 1,201 Mbps max rate) |
| Typical neighboring signal range | −82 to −98 dBm |
| Open (unencrypted) neighboring networks | Present but negligible signal |

*Individual neighboring SSIDs and BSSIDs are withheld — out of scope, third-party networks.*

#### Wireless Network Analysis

**Signal strength.** The assessed network presents an exceptionally strong signal (−37 to −41 dBm)
at the test location, while the strongest neighboring network measured −82 dBm — roughly 40 dB
weaker. Because the dBm scale is logarithmic, the assessed network dominates its local RF
environment. Neighboring networks are too faint to cause interference or rogue-AP confusion, and
an attacker would need close physical proximity to capture usable signal.

**Mesh topology.** The scan clearly reveals the three-node mesh configuration. The strongest
hidden BSSIDs all share the same Netgear OUI prefix and broadcast a hidden 5 GHz network on
channel 155 at the highest observed rate (1,733.3 Mbps), consistent with the dedicated wireless
backhaul linking the mesh nodes. The user-facing SSID broadcasts on both 2.4 GHz (ch 1) and 5 GHz
(ch 48).

**Modes and max rate.** The assessed network operates in a/b/g/n/ac mode (Wi-Fi 5 class). Notably,
the strongest neighbor operates in `ax` mode (Wi-Fi 6), a newer standard than the assessed
network's hardware — consistent with the observation that the RBR50 is due for modernization. No
channel contention issues were observed.

#### Findings from Wireless Assessment

| Finding | Description & Recommendation | Risk |
|---|---|---|
| **W-1: Open SSID** | An open, unencrypted SSID was detected, believed to originate from the Chromecast at `IOT-MEDIA-01` (192.168.1.120). This is the same device generating observed Suricata IDS alerts. Many casting devices broadcast an open setup beacon by design, but this represents a minor internal attack surface. **Recommend** confirming current firmware, disabling guest-mode casting if unused, and isolating IoT/casting devices on a separate SSID/VLAN. | Low |
| **W-2: WPA2 (not WPA3)** | The network uses WPA2-Personal across all nodes. WPA2 is not broken but is no longer best practice; WPA3 offers stronger protection against offline dictionary attacks via SAE. **Recommend** enabling WPA3 or WPA2/WPA3 transition mode if the hardware supports it; otherwise ensure a long, random, unique passphrase and plan for hardware modernization. | Low |

### 4.4 Nmap Scan Results

Host discovery was performed with `nmap -sn 192.168.1.0/24`, identifying **19 live hosts**.
Follow-on service and OS detection (`-sV`, `-O`) was performed against the responding hosts from a
Windows 11 system on the LAN. Where Nmap's OS fingerprinting was unreliable, device identities
were determined where possible by MAC OUI and DHCP correlation.

| Host | IP | OS / Desc | Purpose | Open TCP | Risk Analysis |
|---|---|---|---|---|---|
| `fw-01` | 192.168.1.1 | FreeBSD (Netgate) | Gateway / FW | 53, 80, 443 | LAN DNS + web admin. Expected, but management is reachable by all LAN hosts over HTTP and HTTPS. **Fix:** force HTTPS-only, restrict GUI access, use strong passwords + MFA. |
| `ap-01` | 192.168.1.110 | Netgear Orbi | Router (AP) | 53, 80, 443 | Management + DNS. HTTP (80) management is unencrypted. **Fix:** HTTPS-only admin, confirm non-default credentials, update firmware. |
| `ap-02` | 192.168.1.112 | Netgear Orbi | Satellite | 80, 443 | Management interface on LAN. Same risk/fix as `ap-01`. |
| `ap-03` | 192.168.1.115 | Netgear Orbi | Satellite | 80, 443 | Management interface on LAN. Same risk/fix as `ap-01`. |
| `PRINTER-01` | 192.168.1.117 | VxWorks (HP) | Printer | 80, 139, 443, 445, 515, 631, 6839, 7435, 9100, 9220, 8080 | Largest attack surface on the network. See detailed analysis below and Finding F-1. |
| `IOT-MEDIA-01` | 192.168.1.120 | Google Cast | Streaming | 8008, 8009, 8443, 9000, 10001 | Cast control ports (8008/8009) are unauthenticated by design; any LAN device can control the Chromecast. Correlates with Suricata alerts and the open SSID. **Fix:** IoT VLAN, current firmware. |
| `WORKSTATION-01` | 192.168.1.116 | Windows 11 | Laptop | 135, 139, 445, 5357 | Standard Windows services. SMB (445) / NetBIOS (139) are historically high-value targets. **Fix:** host firewall on untrusted networks, patch SMB, disable SMBv1/NetBIOS if unused. |
| `WORKSTATION-03` | 192.168.1.127 | Windows 11 | Laptop | 5357 | Only WSDAPI exposed. Minimal surface area, low risk. **Fix:** optionally disable WSD. |
| `IOT-HVAC-01` | 192.168.1.103 | Honeywell Home | Thermostat | None | No listening services. *Positive finding.* |
| `IOT-HVAC-02` | 192.168.1.113 | Honeywell Home | Thermostat | None | No listening services. *Positive finding.* |
| `IOT-IRRIG-01` | 192.168.1.106 | Orbit (lwIP) | Irrigation Controller | None | No open ports; cloud-only communications. *Positive finding.* |
| `IOT-CAM-01` | 192.168.1.109 | Ring embedded | Camera | None | No listening services. *Positive finding.* |
| `IOT-CAM-02` | 192.168.1.114 | Ring embedded | Doorbell / Camera | None | No listening services. *Positive finding.* |
| `MOBILE-03` | 192.168.1.119 | Apple iOS | Tablet | 49152, 62078 | Port 62078 (`lockdownd`) is standard on iOS and requires pairing. Low risk. **Fix:** keep iOS updated. |
| `MOBILE-02` | 192.168.1.135 | Android | Phone | None | No listening services. *Positive finding.* |
| `MOBILE-01` | 192.168.1.136 | Android | Phone | None | No listening services. *Positive finding.* |

**Service count:** firewall (3) + AP nodes (7) + printer (11) + laptops (5) + Chromecast (5) +
iOS (2) = **33 services** across the network.

#### Detailed Finding — Network Printer (192.168.1.117)

The printer presents by far the largest network attack surface, exposing eleven open TCP ports
across multiple protocol families:

- **80 / 443 / 8080 (web config)** — embedded web server administration. Default or absent
  credentials would allow any LAN host to reconfigure the device or extract stored credentials.
- **139 / 445 (NetBIOS / SMB)** — file-sharing services historically abused for lateral movement
  and SMB exploitation.
- **515 / 631 / 9100 (LPD / IPP / raw print)** — three print protocols. Port 9100 accepts raw,
  unauthenticated print jobs and has been abused for job exfiltration, malicious firmware
  loading, and denial of service.
- **6839 / 7435 / 9220 (vendor management services)** — additional vendor-specific services
  broadening the surface further.

**Why it matters.** Printers are frequently the least-patched devices on a network, yet are
trusted on the LAN with broad connectivity. A compromised printer can serve as a persistent
foothold, a credential store, and a pivot point. This device is the single highest-priority
hardening target identified by the scan (see Finding F-1).

### 4.5 Vulnerability Scan Results

An authenticated vulnerability scan was performed using **Qualys VMDR** via a Virtual Scanner
Appliance deployed on the LAN (`SCANNER-01`, 192.168.1.137). The scan covered 15 responding hosts
and identified **44 confirmed vulnerabilities**, alongside 9 potential and 257 information-gathered
items.

Full severity distribution and the complete findings inventory are documented separately in
[vulnerability-scan-summary.md](vulnerability-scan-summary.md). The most significant confirmed
vulnerabilities are presented below.

| Host | IP | Device | Severity | Vulnerability (QID) | Risk Analysis & Remediation |
|---|---|---|---|---|---|
| `PRINTER-01` | 192.168.1.117 | Network Printer | **Critical (5)** | Writable SNMP Information (78031) | SNMP accepts default community strings with WRITE access. Any LAN host can reconfigure the printer. **Fix:** disable SNMP or enforce SNMPv3. |
| `PRINTER-01` | 192.168.1.117 | Network Printer | **Critical (5)** | EOL SNMP v1/v2c Detected (105459) | Obsolete, cleartext SNMP transmits community strings in the clear. **Fix:** disable v1/v2c; use SNMPv3 with authentication. |
| `PRINTER-01` | 192.168.1.117 | Network Printer | Medium (3) | TLS 1.0 / 1.1 + Sweet32 (38628 / 38794 / 38657) | Web admin supports obsolete TLS and 64-bit block ciphers (CVE-2016-2183), enabling cipher-related attacks. **Fix:** disable TLS 1.0/1.1 and weak ciphers; update firmware. |
| `PRINTER-01` | 192.168.1.117 | Network Printer | Medium (3) | SMB Signing Disabled; NetBIOS shares (90043 / 70001) | SMB without signing and exposed NetBIOS shares enable tampering and lateral movement. **Fix:** disable SMB/NetBIOS on the printer if unused. |
| `ap-01` | 192.168.1.110 | Wireless Router | Medium (3) | Plaintext HTTP Basic Auth (86763 / 45242) | Admin credentials transmitted unencrypted and sniffable on the LAN. **Fix:** HTTPS-only management. |
| `ap-02` | 192.168.1.112 | Wireless AP | Medium (3) | Plaintext Basic Auth + TLS 1.0/1.1 (86763 / 38628) | Same cleartext-credential exposure, plus obsolete TLS. **Fix:** HTTPS-only, disable legacy TLS. |
| `ap-03` | 192.168.1.115 | Wireless AP | Medium (3) | Plaintext Basic Auth + TLS 1.0/1.1 (86763 / 38794) | Same exposure across the third node. **Fix:** HTTPS-only, disable legacy TLS. |
| `WORKSTATION-01` | 192.168.1.116 | Windows 11 | Medium (3) | SMBv2 Signing Not Required (92094) | SMB without required signing is susceptible to relay/MITM. **Fix:** enable "require SMB signing" via Group Policy. |
| `fw-01` | 192.168.1.1 | Gateway / Firewall | Low (2) | Invalid SSL Cert + NTP Info Disclosure (38173 / 38293) | Self-signed GUI certificate and NTP version disclosure. **Fix:** install a valid certificate; restrict/patch NTP. |

---

## 5. Methodology

This engagement followed a structured, four-phase methodology aligned with industry-standard
practice.

**Phase 1 — Pre-Engagement.** Defined scope, rules of engagement, and constraints in a signed ROE.
Selected a home network over a local business to keep scope and authorization clean, establishing
the formal client relationship through the property owner. Obtained administrative access to the
pfSense and Orbi interfaces.

**Phase 2 — Reconnaissance.** Conducted passive OSINT using five sources (Shodan, Whois/ARIN, Have
I Been Pwned, reverse DNS, and search-engine reconnaissance). Enumerated wireless networks with
inSSIDer. Built the initial system inventory from pfSense DHCP leases and the Orbi device list,
and documented physical security controls.

**Phase 3 — Active Scanning.** Performed host discovery and a service/OS scan with Nmap across the
subnet, then deployed a Qualys VMDR Virtual Scanner Appliance on the LAN to run an authenticated
vulnerability scan against the live hosts. No exploitation was attempted, per the ROE.

**Phase 4 — Documentation & Analysis.** Correlated findings across tools, performed risk analysis
on each, ranked the top three risks, and assembled this report. The Executive Summary was written
last, after all findings were collected and could be prioritized.

### 5.1 Lessons Learned

This assessment reinforced that **strong perimeter security doesn't guarantee a strong interior
security posture.** I had previously worked to get pfSense set up and working with Suricata. I
felt I'd done my due diligence to prevent outside traffic from compromising the network, but
hadn't spent much time looking inside. The initial external reconnaissance confirmed that, from
the outside looking in, the security posture wasn't bad — Shodan found no exposed services, and
the firewall silently dropped unsolicited inbound traffic. Once the assessment turned inward,
however, significant risks surfaced that I had been unaware of. The printer's exposure was the
clearest example, and finding that the most serious remaining issues were on wireless management
interfaces *I had configured myself* was an eye-opening result.

The single most valuable lesson was **the power of correlating multiple tools.** The Chromecast
appeared in Suricata IDS alerts, the Nmap port scan, the inSSIDer wireless scan, and the Qualys
VMDR scan — and each tool contributed a different piece of information about the same device. The
same held for the printer, where Nmap and Qualys each told half the story.

Process-wise, two lessons stand out. First, **"no result" is itself often a great result.** The
empty Shodan response and the absent PTR record are positive indicators worth documenting
explicitly. Second, **report section order differs from work order** — the executive summary
appears first but must be written last. Future assessments would also benefit from earlier asset
inventory work: two devices remain only partially identified, which is both a challenge to
resolve and a reminder that a current, complete asset inventory is a prerequisite to good
security.

---

## Appendix: De-identification Notes

This public copy was prepared from the original academic report. The following changes were made.

**Removed entirely**

- Public WAN IP address and all derived geolocation/ISP identification.
- Names of the property owner, household members, and the course instructor.
- Personal and academic email addresses; the tester's OSINT username.
- Specific data-broker, breach-corpus, and social-media OSINT results.
- All photographs (physical access-control hardware, network equipment location).
- Raw inSSIDer screenshots and all neighboring third-party SSIDs and BSSIDs.
- Raw Nmap output screenshots.
- Qualys account identifiers (user name, login name, subscription/company, scan reference ID).
- Handwritten signatures on the ROE.

**Generalized**

- Residence location → "a private residence."
- Device hostnames → functional labels (`WORKSTATION-01`, `IOT-CAM-01`, etc.).
- Home SSID → `[HOME-SSID]`; mesh BSSIDs masked to the vendor OUI.
- Client/owner identity → "the property owner / Network Owner."

**Deliberately preserved**

- All RFC 1918 private addressing (`192.168.1.0/24`). These addresses are non-routable and
  reveal nothing about a reachable target; removing them would render the analysis unreadable.
- All vulnerability findings, QIDs, CVE references, port lists, OS/vendor identification, and
  severity ratings — this is the technical substance of the assessment.
- All methodology, risk analysis, and conclusions.

**Scope of de-identification.** The intent is to remove the information that would let a reader
locate and target the assessed network, not to conceal authorship. The report is published under
the author's name; the network, its owner, and its occupants are not identified.

---

## Document Control

| Version | Date | Author | Notes |
|---|---|---|---|
| 0.1 | May 27, 2026 | Michael Keuhlen | Initial shell — placeholders pending scan results |
| 0.5 | June 9, 2026 | Michael Keuhlen | Added completed analysis, documentation, and formatting |
| 1.0 | June 14, 2026 | Michael Keuhlen | Final version submitted for grading |
| 1.1 | *(public release)* | Michael Keuhlen | De-identified copy prepared for public portfolio release |
