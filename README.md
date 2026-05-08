# Deployment-Site-to-Site-Remote-Access-VPN-with-Multi-Factor-Authentication-MFA-Integration
1. Project Overview

This internship builds a complete enterprise-grade VPN solution with two interconnected components: a Site-to-Site IPsec (IKEv2) tunnel connecting a Headquarters and a Branch site, and a Remote Access VPN (IKEv2/SSL) for mobile/home users. The distinguishing feature is the seamless integration of Multi-Factor Authentication (MFA) using FreeRADIUS and Google Authenticator (TOTP), enforced on all connection types.

Rather than relying on a GUI-based firewall appliance, all packet filtering and traffic control is implemented by writing firewall rules directly on the Linux gateway routers (R1 at HQ, R2 at Branch, R3 at Site C) using **iptables / nftables**. This approach reflects real-world enterprise practice and provides a deeper understanding of the underlying packet flow.

### Lab Topology Summary

| Site                   | Role                | Gateway           | LAN Subnet           |
| ---------------------- | ------------------- | ----------------- | -------------------- |
| Site A (HQ)            | MFA + FreeRADIUS    | R1 — `10.10.10.1` | `192.168.10.0/24`    |
| Site B (Branch)        | Branch office       | R2 — `10.10.20.1` | `192.168.20.0/24`    |
| Site C (Remote Access) | VPN remote users    | R3 — `10.10.30.1` | `192.168.30.0/24`    |
| WAN                    | Internet simulation | WAN Router        | `10.10.x.x/30` links |

**The work plan below divides the 12-week internship into three phases of 4 weeks each:**

- **Month 1** — Foundation & Research: theory, lab setup, PKI, baseline VPN
- **Month 2** — Full Implementation: MFA integration, Remote Access VPN, LDAP exploration
- **Month 3** — Hardening, Benchmarking & Thesis: security, performance metrics, documentation

---

## 2. High-Level Phase Overview

| Month | Phase | Focus Area | Key Deliverable |
|-------|-------|-----------|----------------|
| Month 1 (Weeks 1–4) | Foundation & Research | Theory, env setup, basic VPN | Working lab topology + research report |
| Month 2 (Weeks 5–8) | Implementation | IPsec, Remote Access VPN, MFA integration | Fully functional VPN with TOTP MFA |
| Month 3 (Weeks 9–12) | Hardening & Thesis | Security, performance, documentation | Final thesis + demo-ready system |

---

## 3. Detailed Weekly Work Plan

Each week is mapped to specific tasks and a concrete deliverable. Weeks 1–4 build the foundation, Weeks 5–8 complete implementation, and Weeks 9–12 focus on hardening and thesis writing.

| Week | Phase | Tasks | Deliverable |
|------|-------|-------|------------|
| W1 | Research & Planning | Study VPN protocols (IPsec IKEv2, SSL/TLS). Review MFA (TOTP, RADIUS). Study StrongSwan & iptables/nftables docs. Define network topology (R1/R2/R3 as Linux gateways). | Research notes, topology diagram draft |
| W2 | Environment Setup | Install VMware/VirtualBox/Proxmox. Deploy Linux gateway VMs (R1, R2, R3, WAN Router), FreeRADIUS VM, client VMs. Configure static IPs, routing, IP forwarding, and basic iptables rules. | Lab environment running, IP reachability verified |
| W3 | PKI & Certificate Authority | Set up internal CA using Easy-RSA. Generate server & client certificates for R1, R2, R3, and StrongSwan. Configure CRL. Test certificate issuance. | PKI infrastructure ready |
| W4 | Site-to-Site Baseline | Configure IPsec IKEv2 tunnel R1 (HQ) ↔ R2 (Branch) via StrongSwan. Write iptables rules on R1 & R2 to permit tunnel traffic. Test connectivity & basic throughput. | S2S tunnel working, baseline metrics |
| W5 | FreeRADIUS Setup | Install & configure FreeRADIUS 3.x. Integrate Google Authenticator (libpam-google-authenticator). Test TOTP locally. Create test user accounts. | RADIUS + TOTP functional |
| W6 | Remote Access VPN | Configure IKEv2 Remote Access on R3 (StrongSwan) for remote users (PC-B). Integrate RADIUS+MFA authentication. Write iptables rules on R3 for VPN client traffic. Test end-to-end login with TOTP. | Remote Access VPN + MFA working |
| W7 | S2S MFA Integration | Integrate RADIUS into Site-to-Site IPsec (EAP-RADIUS on StrongSwan). Update firewall rules on R1/R2 accordingly. Test IKEv2 EAP-RADIUS auth. Document auth flow diagram. | S2S VPN with MFA authenticated |
| W8 | LDAP / AD Integration | Explore LDAP/Active Directory integration (FreeIPA or Windows AD). Bind FreeRADIUS to LDAP for user lookup. Test LDAP-backed TOTP login. Document limitations. | LDAP integration tested & documented |
| W9 | Security Hardening | Disable 3DES, MD5, weak ciphers. Enforce AES-256-GCM, SHA-256, PFS. Apply Zero Trust network segmentation. Harden iptables/nftables rules on R1, R2, R3 (default-deny, per-zone policies). | Hardened system, security checklist |
| W10 | Performance Benchmarking | Measure throughput (iperf3). Measure latency (ping, traceroute). Measure CPU usage under load. Measure MFA auth time. Compare with/without MFA overhead. | Performance report with comparison tables |
| W11 | Threat Modeling & Review | Document threat model (STRIDE). Identify attack surfaces. Test fail scenarios (MFA bypass, replay, brute force). Write vulnerability assessment. | Threat model document |
| W12 | Thesis & Final Demo | Write complete thesis (background, architecture, implementation, results). Prepare demo environment. Create final presentation slides. Submit. | Final thesis + presentation |

---

## 4. Month-by-Month Breakdown

### Month 1 (Weeks 1–4): Foundation & Research

#### Goals

- Understand all relevant protocols and tools (IPsec, IKEv2, StrongSwan, RADIUS, TOTP)
- Set up the complete lab virtualization environment with Linux gateway routers
- Establish a functioning PKI with internal Certificate Authority
- Achieve a working Site-to-Site VPN baseline with firewall rules written directly on the gateways

#### Key Activities

- **Week 1:** Deep-dive into protocol documentation — IKEv2 RFCs, StrongSwan `swanctl.conf` reference, Google Authenticator TOTP spec (RFC 6238), FreeRADIUS configuration guides, and Linux iptables/nftables firewall documentation. Draw the full network topology (HQ R1, Branch R2, Remote R3, WAN simulation).

- **Week 2:** Install hypervisor (VMware/VirtualBox/Proxmox). Spin up: **R1** (HQ Linux gateway + StrongSwan), **R2** (Branch Linux gateway + StrongSwan), **R3** (Remote Access Linux gateway + StrongSwan), a WAN Router VM, FreeRADIUS VM, Windows/Linux client VMs. Enable IP forwarding on all gateway VMs. Configure static IPs and basic iptables FORWARD rules to permit inter-site traffic.

- **Week 3:** Set up internal CA using Easy-RSA. Issue server certificates for R1, R2, R3. Issue client certificates for test users (PC-A, PC-C). Configure CRL (Certificate Revocation List) distribution.

- **Week 4:** Configure IKEv2 Site-to-Site tunnel between R1 and R2 using `swanctl.conf`. Start with pre-shared keys, then migrate to certificate-based auth. Write iptables rules to allow ESP, IKE (UDP 500/4500), and tunnel traffic. Verify tunnel stability. Collect baseline performance metrics using iperf3.

#### Week 1 Study Checklist

- IPsec protocol stack (IKEv1 vs IKEv2), ESP and AH headers
- AES-256-GCM, SHA-256, Diffie-Hellman groups, Perfect Forward Secrecy (PFS)
- StrongSwan IKEv2 configuration: `swanctl.conf`, `ipsec.secrets`
- Linux IP forwarding: `sysctl net.ipv4.ip_forward`, routing tables
- iptables fundamentals: chains (INPUT, FORWARD, OUTPUT), NAT table, stateful tracking (`-m state --state`)
- nftables as a modern alternative to iptables
- RADIUS protocol (RFC 2865) and EAP extensions (EAP-TTLS, EAP-PEAP)
- TOTP algorithm (RFC 6238) and Google Authenticator integration

---

### Month 2 (Weeks 5–8): Full Implementation

#### Goals

- Deploy FreeRADIUS with Google Authenticator TOTP successfully
- Integrate MFA into both Remote Access VPN and Site-to-Site VPN
- Maintain correct iptables/nftables rules on each gateway as services are added
- Explore optional LDAP/Active Directory integration
- Produce a fully functional, demo-ready VPN + MFA stack

#### Key Activities

- **Week 5:** Install FreeRADIUS 3.x on the FreeRADIUS VM (192.168.10.10). Install `libpam-google-authenticator`. Configure the PAM module for FreeRADIUS. Create test user accounts with provisioned TOTP keys. Test OTP login via `radtest`. Ensure R1's iptables allows UDP 1812/1813 from trusted sources only.

- **Week 6:** On R3, configure StrongSwan for IKEv2 Remote Access (Road Warrior). Point authentication to the FreeRADIUS server at HQ (username + TOTP via EAP-RADIUS). Write iptables rules on R3: allow VPN client traffic in, NAT/route to the 192.168.30.0/24 LAN. Test with a StrongSwan or native IKEv2 client on PC-B. Capture Wireshark traces of the authentication exchange.

- **Week 7:** Configure EAP-RADIUS within the IKEv2 Phase 1 setup in StrongSwan on R1 and R2. Link authentication to FreeRADIUS with TOTP. Update iptables on R1/R2 to reflect auth plane traffic. Test S2S re-authentication on tunnel restart. Document the full auth flow diagram.

- **Week 8 (Stretch):** Evaluate LDAP integration — either FreeIPA on Linux or a Windows Server AD lab. Bind FreeRADIUS to LDAP for user lookup. Test LDAP-backed TOTP login. Document what works, what limitations exist.

#### MFA Integration Architecture

```
Remote User (PC-B)
  └─► R3 (IKEv2 Road Warrior) ──► FreeRADIUS (192.168.10.10) ──► PAM/TOTP ──► ACCEPT/REJECT
        └─ iptables: allow VPN pool traffic, forward to LAN

Branch R2 ──► IKEv2 EAP (StrongSwan) ──► R1 (HQ) ──► FreeRADIUS ──► TOTP ──► Tunnel UP/DOWN
               └─ iptables on R1/R2: allow ESP, IKE, permit tunnel subnets

FreeRADIUS ◄── Optional LDAP/AD for centralized user directory lookup
```

---

### Month 3 (Weeks 9–12): Hardening, Performance & Thesis

#### Goals

- Harden the system against known attack vectors
- Collect and analyze all performance metrics
- Complete threat modeling and security assessment
- Write and submit a comprehensive thesis

#### Key Activities

- **Week 9 — Security Hardening:** Disable 3DES, RC4, MD5, DH Group 1/2 in `swanctl.conf`. Enforce AES-256-GCM + SHA-256 + DH Group 14 or 20 (ECP). Apply Zero Trust segmentation: define strict iptables/nftables zone policies on R1, R2, and R3 with **default-deny FORWARD** rules and explicit allow entries per traffic flow. Validate cipher suites with `openssl s_client` and `testssl.sh`. Audit all gateway firewall rulesets.

- **Week 10 — Performance Benchmarking:** Run `iperf3` for throughput: with and without MFA, S2S vs Remote Access. Measure avg/min/max latency. Monitor CPU usage on the Linux gateway VMs (`top`, `htop`, `sar`) during peak VPN load. Time the full authentication cycle (VPN connect → TOTP entry → tunnel UP). Build comparison tables.

- **Week 11 — Threat Modeling:** Apply STRIDE methodology (Spoofing, Tampering, Repudiation, Info Disclosure, DoS, Elevation). Document attack scenarios: credential theft, replay attack, MFA bypass, MITM. Run simulated tests where possible. Write vulnerability assessment and countermeasures.

- **Week 12 — Thesis & Final Demo:** Compile all chapters: Abstract, Background, Related Work, Architecture, Implementation, Results, Conclusion. Integrate all diagrams, tables, screenshots. Write deployment guide appendix (including gateway firewall rule listings). Prepare live demo. Submit thesis.

#### Thesis Outline

- **Chapter 1:** Introduction & Problem Statement
- **Chapter 2:** Background — VPN Types, Authentication Mechanisms, Zero Trust
- **Chapter 3:** Related Work & Comparison of Existing Solutions
- **Chapter 4:** System Architecture & Design (topology, gateway role, firewall design)
- **Chapter 5:** Implementation Details (S2S, Remote Access, MFA, PKI, iptables/nftables rules)
- **Chapter 6:** Security Hardening & Threat Modeling
- **Chapter 7:** Performance Evaluation & Results
- **Chapter 8:** Conclusion & Future Work
- **Appendix A:** Full Configuration Files (StrongSwan, FreeRADIUS, iptables rulesets)
- **Appendix B:** Deployment Guide

---

## 5. Key Milestones

| Milestone | Description | Success Criteria | Target Date |
|-----------|-------------|-----------------|-------------|
| M1 | Lab Environment Live | All VMs running, IP reachability verified, IP forwarding enabled | End of Week 2 |
| M2 | PKI & Certificates | CA functional, certs issued & tested | End of Week 3 |
| M3 | Site-to-Site VPN | R1–R2 tunnel passing traffic, iptables rules verified | End of Week 4 |
| M4 | RADIUS + MFA Ready | TOTP login tested successfully via `radtest` | End of Week 5 |
| M5 | Full VPN + MFA Stack | Both VPN types with MFA working, firewall rules complete | End of Week 7 |
| M6 | Hardened System | Weak protocols disabled, ZT rules applied on all gateways | End of Week 9 |
| M7 | Performance Report | Metrics collected, comparison tables complete | End of Week 10 |
| M8 | Final Submission | Thesis submitted, demo prepared | End of Week 12 |

---

## 6. Tools & Technologies

| Category | Tool / Technology | Purpose |
|----------|-----------------|---------|
| Hypervisor | VMware Workstation / VirtualBox / Proxmox | Host all VMs in lab environment |
| Gateway OS | Debian / Ubuntu Server (R1, R2, R3, WAN Router) | Linux-based VPN gateways with routing & firewall |
| IPsec / IKEv2 | StrongSwan 5.x (`swanctl`) | IKEv2 daemon for S2S and Remote Access VPN |
| Firewall | iptables / nftables | Packet filtering & NAT written directly on each gateway |
| RADIUS Server | FreeRADIUS 3.x | Auth server integrating TOTP (located at HQ, 192.168.10.10) |
| MFA / TOTP | Google Authenticator (`libpam-google-authenticator`) | Time-based one-time passwords |
| PKI | Easy-RSA | Internal CA, server/client certificate management |
| VPN Client | StrongSwan client / native OS IKEv2 client | Remote access client connections (PC-B) |
| Directory Services | FreeIPA or Windows Server AD | LDAP user management (stretch goal) |
| Performance Testing | iperf3, ping, traceroute, top/htop/sar, Wireshark | Benchmarking throughput, latency, CPU |
| Security Tools | nmap, openssl, testssl.sh | Verify hardening & cipher validation |
| Documentation | Draw.io, LaTeX / Markdown, Wireshark | Diagrams, thesis, packet capture analysis |

---

## 7. Risk Register

| Risk | Description | Level | Mitigation |
|------|-------------|-------|-----------|
| VM Resource Constraints | Lab VMs may be slow on low-end hardware | **Medium** | Use lightweight distros (Debian minimal); allocate RAM carefully |
| iptables Rule Complexity | Stateful rules across 3 gateways can be error-prone | **High** | Build rules incrementally; use `iptables -L -v -n` to verify; document each rule with comments |
| FreeRADIUS Complexity | TOTP + EAP config can be tricky | **High** | Test each component separately (radtest first, then EAP); follow FreeRADIUS community guides |
| StrongSwan EAP-RADIUS | Interop/config issues between StrongSwan and FreeRADIUS | **High** | Use StrongSwan wiki; start with PSK, migrate to EAP incrementally; check `/var/log/charon.log` |
| Scope Creep | LDAP/AD integration may take longer than expected | **Medium** | Make LDAP a stretch goal; protect core deliverables in Weeks 5–7 |
| Documentation Delay | Thesis writing underestimated | **Medium** | Write incrementally each week, not all at the end |

---

## 8. Tips for Success

- **Document everything as you go — don't leave thesis writing to the last week.** Keep a running markdown lab journal noting what worked, what failed, and why.
- Take VM snapshots before major config changes (especially before editing iptables chains or StrongSwan config) so you can roll back quickly.
- Use Draw.io to maintain a living network diagram that evolves as your implementation grows — include subnet labels and firewall rule zones.
- Build your iptables/nftables ruleset in a shell script from day one. This forces you to think declaratively, makes it reproducible, and provides ready-made content for the thesis appendix.
- Test each component in isolation before integrating — e.g., test FreeRADIUS TOTP with `radtest` before wiring it into StrongSwan EAP.
- Always flush and reload firewall rules from a script rather than adding rules interactively, to avoid rule ordering bugs.
- For the performance chapter, run each benchmark at least 3 times and report the average.
- Make the LDAP/AD integration a stretch goal, not a core commitment. Protect Weeks 5–7 for the primary MFA deliverables.
- Prepare the demo environment in Week 11, not Week 12 — leave Week 12 purely for thesis polish and submission.

---

*University of Science & Technology of Hanoi | Cyber Security | AY 2025–2026*
