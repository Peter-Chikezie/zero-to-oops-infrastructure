# zero-to-oops-infrastructure

> Building and hopefully not breaking a full hybrid enterprise lab from scratch: Cisco IOS, pfSense, RHEL, Windows Server, Splunk, and Terraform/Ansible.

This repository is the working documentation, configuration archive, and automation codebase for my hybrid enterprise lab. Instead of doing disconnected tutorials, I built one unified environment to practice how enterprise networking, Linux/Windows systems, security monitoring, and cloud automation actually fit together—and to have a safe sandbox to break things, troubleshoot them, and document the fixes.

---

## Architecture Overview

The lab mirrors a multi-tier enterprise environment bridging on-premises infrastructure (virtualized in EVE-NG and VMware Workstation) with cloud workloads:

```
[ On-Premises Topology (EVE-NG & VMware) ]
 |--- Core & Access Layer: Cisco IOS L3/L2 (OSPF, HSRP, 802.1Q VLANs, Port Security)
 |--- Perimeter Edge: pfSense Firewall (NAT, Stateful Rules, IPsec Site-to-Site VPN)
 |--- Linux Compute: Red Hat Enterprise Linux 9 (LVM, SELinux, firewalld, Podman, SSSD)
 |--- Identity & Directory: Windows Server 2022 AD DS + Microsoft Entra ID Sync
 |--- Security Operations: Splunk SIEM (Centralized Syslog, auditd, Custom SPL Alerts)
          │
          ▼ (IPsec VPN / Hybrid Cloud Integration)
[ Cloud & Automation Layer (AWS / IaC) ]
 |--- Provisioning: HashiCorp Terraform (VPC, Subnets, Security Groups, EC2)
 |--- Configuration Management: Ansible Playbooks & Bash/PowerShell Scripts
```

---

## Repository Structure

```text
zero-to-oops-infrastructure/
├── networking/        # EVE-NG topologies, Cisco IOS startup-configs, pfSense rules
├── systems/           # RHEL 9 setup notes (LVM, SELinux, SSSD), AD/GPO baselines, scripts
├── security/          # Splunk SIEM forwarding configs, custom SPL queries, Wireshark PCAP notes
├── cloud-iac/         # Terraform modules for AWS VPC/EC2 & Ansible hardening playbooks
└── docs/              # Network topology diagrams (draw.io) & troubleshooting post-mortems
```

---

## Core Lab Domains & Objectives

### 1. Enterprise Networking & Perimeter (`/networking`)
* **Routing & High Availability:** Multi-site topology running **OSPF** for dynamic routing and **HSRP** for default gateway redundancy.
* **Switching & Segmentation:** **802.1Q** trunking, Inter-VLAN routing, VLSM subnetting, extended ACLs, and switch **Port Security**.
* **Edge Security:** **pfSense** stateful firewall rules, NAT policies, and an AES-256 **IPsec VPN** tunnel connecting remote segments.

### 2. Hybrid Systems Administration (`/systems`)
* **Red Hat Enterprise Linux (RHEL 9):** Custom **LVM** storage pools (`pvcreate`, `vgcreate`, `lvextend`), `systemd` service units, `SELinux` enforcing policies, `firewalld` zones, and rootless **Podman** containers.
* **Windows Server & Hybrid Identity:** Multi-OU **Windows Server 2022 Active Directory** domain (`oops.internal`), security **GPOs**, **Microsoft Entra ID** sync, and Linux domain joining via **SSSD/Realm**.
* **Administrative Scripting:** **Bash** and **PowerShell** (`Posh-SSH`) scripts for automated configuration backups, connectivity sweeps, and user provisioning.

### 3. Security Monitoring & Defense (`/security`)
* **Centralized SIEM:** **Splunk Enterprise** ingesting Syslog streams from Cisco devices, RHEL `auditd` logs, and Windows Domain Controllers.
* **Detection Engineering:** Custom **SPL (Search Processing Language)** queries and alerts tuned to catch SSH brute-force attempts, privilege escalation, and anomalous logins.
* **Packet Analysis:** **Wireshark** PCAP inspections to diagnose DNS anomalies, cleartext protocols, and firewall rule behavior.

### 4. Cloud & Infrastructure as Code (`/cloud-iac`)
* **Provisioning:** Modular **Terraform** configurations deploying segmented **AWS VPCs**, public/private subnets, Internet Gateways, and Security Groups.
* **Configuration Management:** Idempotent **Ansible** playbooks automating post-deployment OS updates (`dnf`), SSH key setup, and firewall baselines.

---

## Build Status & Roadmap

* [x] **Environment Staging:** EVE-NG & VMware Workstation hypervisors configured with RHEL 9 `.qcow2` and ISO nodes.
* [ ] **Phase 1 — Hybrid Identity & Linux Core:** Windows Server 2022 AD DS setup, GPO hardening, and RHEL 9 SSSD integration.
* [ ] **Phase 2 — Enterprise Network Core:** Multi-site Cisco OSPF/HSRP topology and pfSense IPsec VPN perimeter.
* [ ] **Phase 3 — SIEM & Log Telemetry:** Splunk indexer deployment and Syslog/auditd forwarding pipelines.
* [ ] **Phase 4 — Cloud & Automation:** AWS VPC deployment via Terraform and RHEL configuration via Ansible.

---

## Certifications & Continuous Learning

* **CompTIA Security+ (SY0-701)** — *November 2026*
* **Cisco Certified Network Associate (CCNA 200-301)** — *December 2026*
* **Red Hat Certified System Administrator (RHCSA EX200)** — *Target: Q1 2027*

---

*Built and maintained by **Peter Chikezie O.** 
