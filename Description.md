## Description:

This project focuses on researching and building a secure network connectivity solution for enterprises through two primary VPN types: Site-to-Site VPN for connecting multiple

offices/datacenters and Remote Access VPN for secure remote user access. A core differentiator of this research is the integration of Multi-Factor Authentication (MFA) to enhance security beyond traditional password-based methods. The system will be implemented using professional-grade open-source platforms such as nftables and StrongSwan, adhering to modern security standards including IKEv2, AES-256-GCM, and Zero Trust principles.

## Expected outcomes:

- Functional Lab Environment: A complete network topology consisting of a Headquarters
(HQ) Site, Branch Site, and Remote Users.
- MFA Integration: Successful implementation of FreeRADIUS integrated with Google
Authenticator (TOTP) for all VPN connections.
- Security Hardening: A hardened system utilizing strong encryption (AES-256-GCM), SHA-25 
hashing, and Perfect Forward Secrecy (PFS) while disabling legacy/weak protocols like 3DES
or MD5.
- Performance Metrics: Detailed empirical data and comparison tables regarding Throughput,
Latency, CPU Usage, and Authentication Time.
- Technical Documentation: A comprehensive thesis covering threat modeling, architecture
design, and deployment guidelines.

## Used Methods and Techniques:

- VPN Protocols: Implementation of IPsec (IKEv2) for Site-to-Site tunnels and OpenVPN/SSL
VPN for remote clients.
- Authentication Mechanisms: Utilizing RADIUS + MFA, Certificate-based (PKI) authentication,
and exploring LDAP/Active Directory integration.
- Virtualization & Tools: Deploying the environment on VMware, VirtualBox, or Proxmox.