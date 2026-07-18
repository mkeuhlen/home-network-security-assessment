# Rules of Engagement

**Network Security Assessment — Private Residence**

| Field | Value |
|---|---|
| Prepared By | Michael Keuhlen |
| Client | Property Owner / Network Owner (name withheld) |
| Engagement Start | May 27, 2026 |
| Engagement End | June 10, 2026 |
| Document Version | 2.0 |
| Course | CSIA 440: Cyber Testing and Penetration |
| Institution | Columbia Basin College |

> **Public release note.** This is a de-identified copy of a document originally executed
> for an academic engagement. Names of private individuals, the residence location, the
> public WAN address, and handwritten signatures have been removed. The scope,
> constraints, and authorization structure are unmodified.

---

## 1. Purpose and Parties

### 1.1 Purpose

This document establishes the Rules of Engagement (ROE) — the agreed-upon terms, conditions,
and constraints governing the conduct of a limited network security assessment of a private
residence's home network. This document was executed in connection with coursework for
CSIA 440: Cyber Testing and Penetration at Columbia Basin College.

The purpose of the assessment was to identify potential security vulnerabilities in the home
network infrastructure, assess the effectiveness of existing security controls, and produce a
professional report documenting findings and recommendations.

### 1.2 Parties

| Role | Name | Affiliation | Contact |
|---|---|---|---|
| Client / Network Owner | *Withheld* | Private residence | *Withheld* |
| Lead Tester / Student | Michael Keuhlen | Columbia Basin College — CSIA 440 | *Withheld* |

---

## 2. Client Objectives

The client authorized this assessment to accomplish the following objectives:

- Identify unauthorized or improperly secured access points on the home network.
- Assess the risk of unauthorized access to network-connected devices and data.
- Evaluate the effectiveness of the pfSense firewall configuration on the Netgate 1100.
- Identify potential data exfiltration pathways or exposure of sensitive information.
- Assess the security posture of the wireless network (Netgear Orbi RBR50 mesh system).
- Produce findings and actionable recommendations to improve the overall security of the
  home network.

---

## 3. Scope and Boundaries

### 3.1 In-Scope Systems and Infrastructure

The following systems and infrastructure components were explicitly authorized for testing:

- All devices connected to the home LAN and wireless network at the residence.
- The Netgate 1100 running pfSense (gateway/firewall) — **LAN-facing interfaces only**.
- The Netgear Orbi RBR50 (operating in Access Point mode) and its two satellite units.
- All IP addresses assigned within the home network's DHCP scope.
- Wireless networks (SSIDs) broadcast from the Orbi mesh system.
- Physical security controls of the network infrastructure and the residence.

### 3.2 Out-of-Scope Systems

The following were explicitly excluded from the scope of this engagement:

- The WAN-facing interface of the Netgate 1100 and the ISP infrastructure beyond it.
- The Optical Network Terminal (ONT) — ISP-owned equipment that must not be tested.
- Any third-party networks visible during wireless scanning (neighboring SSIDs).
- Any cloud-based services or external accounts associated with devices on the network.
- Active exploitation of discovered vulnerabilities beyond confirming their presence
  (no destructive testing).
- Any denial-of-service (DoS) or network disruption techniques.

---

## 4. Engagement Timeframe

| Milestone | Date |
|---|---|
| Engagement Start Date | May 27, 2026 |
| Reconnaissance Phase | May 27 – May 31, 2026 |
| Active Scanning Phase | June 1 – June 5, 2026 |
| Vulnerability Scanning Phase | June 1 – June 7, 2026 |
| Report Assembly & Review | June 7 – June 10, 2026 |
| Engagement End Date | June 10, 2026 |
| Academic Submission Deadline | June 14, 2026 |

Testing activities were to be conducted during hours that minimize disruption to normal
household network usage. Scans were to be avoided during periods of high household activity
where possible. No other time-of-day restrictions were imposed by the client.

---

## 5. Authorized Tools and Techniques

| Tool | Category | Intended Use |
|---|---|---|
| inSSIDer | Wireless reconnaissance | Enumerate visible and hidden wireless networks; identify SSIDs, encryption types, signal strength, and channel usage. |
| Nmap | Network scanning | Discover live hosts, identify open TCP/UDP ports, detect operating systems and running services. |
| Vulnerability scanner (OpenVAS / Nessus Essentials / Qualys VMDR) | Vulnerability scanning | Authenticated and unauthenticated vulnerability scans against in-scope hosts. No exploitation. |
| OSINT tools (Shodan, Whois, DNS lookup, search engines, Have I Been Pwned) | Open-source intelligence | Collect publicly available information about the network, IP ranges, and the tester's personal digital footprint. |
| pfSense web interface | Network enumeration | Review DHCP leases, firewall rules, interface configurations, and connected device inventory. |
| Netgear Orbi web interface | Network enumeration | Review connected device lists, wireless configuration, and access point status. |
| Physical inspection & photography | Physical security assessment | Document physical access controls including locks, hardware enclosures, and other deterrents. |

**No exploit frameworks (e.g., Metasploit), password crackers, denial-of-service tools, or
packet injection tools were authorized for this engagement.** All scanning was to be passive
or non-destructive in nature.

---

## 6. Client-Specific Concerns and Constraints

- Network availability must be maintained throughout the engagement. No testing activity
  shall cause an outage or persistent disruption to household internet or local network
  services.
- Smart home devices (e.g., streaming devices, smart TVs, voice assistants) are in scope for
  discovery and scanning only. No interaction with device-specific applications or cloud
  accounts is authorized.
- **No credentials belonging to other household members shall be accessed, captured, or used
  during the engagement.**
- Physical security photography shall be limited to network hardware, entry points, and
  authorized deterrents within the residence.
- All findings and collected data shall be handled confidentially and used solely for the
  purposes of this academic assignment.

---

## 7. Authorization Statement

> ### AUTHORIZATION TO CONDUCT SECURITY TESTING
>
> I, the undersigned, as owner and authorized representative of the residence and network
> described in this document, hereby grant explicit written authorization to Michael Keuhlen
> to conduct a limited network security assessment of the home network infrastructure
> described herein.
>
> This authorization is valid for the period of **May 27, 2026 through June 14, 2026**. All
> testing activities shall be conducted within the scope and constraints defined in this
> Rules of Engagement document.
>
> Any person or organization questioning the legitimacy of network scanning or security
> testing activities originating from the above-described network during the authorized
> period should contact the lead tester or the supervising course instructor at Columbia
> Basin College.
>
> This document serves as the sole written authorization for this engagement. All activities
> remain limited to the in-scope systems and techniques defined herein.

---

## 8. Confidentiality

All information collected during this engagement — including network topology, device
inventories, vulnerability findings, and any credentials or data encountered — is considered
confidential. This information shall be used solely for the academic purposes of CSIA 440 and
shall not be disclosed to any third party outside of the student, client, and instructional
staff of Columbia Basin College.

The report produced from this engagement shall be submitted to the course instructor in
accordance with the course requirements and shall not be published or distributed elsewhere
without the express consent of the client.

> **Note on this public copy:** the de-identified version of this engagement's materials is
> published with the client's consent, consistent with the clause above.

---

## 9. Liability and Limitations

The client acknowledges that security testing activities, while conducted in good faith and
within the defined scope, carry an inherent risk of unintended network disruption. The tester
agrees to exercise reasonable care to minimize disruption and to notify the client immediately
in the event of any unexpected impact on network availability.

Both parties agree that this engagement is conducted for educational purposes only. The tester
assumes no liability for pre-existing vulnerabilities or network conditions discovered during
the course of the test.

---

## 10. Signatures

By signing, both parties acknowledged that they had read, understood, and agreed to the terms
set forth in this Rules of Engagement document.

**CLIENT SIGNATURE**
`/s/ on file` — Network Owner — Dated 5/27/2026

**TESTER SIGNATURE**
`/s/ on file` — Michael Keuhlen, Lead Tester / Student — Dated 5/27/2026
*CSIA 440 | Columbia Basin College*

---

## 11. Document Control

| Version | Date | Author | Notes |
|---|---|---|---|
| 2.0 | May 27, 2026 | Michael Keuhlen | Final draft for signature |
| 2.0-r | *(public release)* | Michael Keuhlen | De-identified copy prepared for public portfolio release |
