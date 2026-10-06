# Day 01 Lab Log - Phase 1: Virtual Lab Foundation for SOC Analyst Home Lab
**Date:** Sept 30, 2026    
**Focus:** VmWare lab architecture, NAT/host-only networking, Windows/Ubuntu/Kali VM roles, SSH workflow, snapshots, and troubleshooting
---

## Phase 1 Title

**Phase 1 - Virtual Lab Environment Setup**
This phase built the foundation for the entire SOC Analyst home lab. The goal is to build a safe, repeatable, isolated lab environment that could support future phases involving SIEM deployment, endpoint monitoring, IDS traffic analysis, Wazuh XDR, phishing investigation, SOC ticket writing, and attack simulation.

---

# 1. Purpose of Phase 1

A SOC analyst home lab needs multiple systems that can communicate with each other in a controlled environment. In a real company, a SOC analyst reviews logs from endpoints, servers, firewalls, EDR tools, SIEM platforms, identity systems, and network security tools.

In this lab, those pieces are simulated with virtual machines.

The purpose of this phase was to create:
- A Windows victim endpoint
- An Ubuntu SIEM server
- A Kali Linux attacker/testing machine
- A baseline snapshot strategy
- A clean foundation before installing security tools

The main idea:

```text
Build the lab first.
Confirm networking.
Take snapshots.
Then install security tools in later phases.
```

---

# 2. High-Level Lab Design

The initial Phase 1 lab design included three main virtual machines:

```text
Ubuntu SIEM VM       = Security monitoring / SIEM server
Windows 10 Victim VM = Endpoint being monitored and tested
Kali Linux VM        = Attacker/testing machine for later phases
```

Later phases added Wazuh as a separate VM, but Phase 1 focused on the original core environment.

## Phase 1 VM Roles

| VM | Role | Why It Exists |
|---|---|---|
| Ubuntu SIEM VM | Monitoring server | Hosts Elastic/Kibana/Fleet/Suricata in later phases |
| Windows 10 Victim VM | Endpoint | Generates logs, Sysmon telemetry, Windows events, user activity |
| Kali Linux VM | Testing/attacker box | Used later for scanning, traffic generation, and safe attack simulation |

---
# 3. VmWare Network Design

Phase 1 used two main network adapter types:

## NAT Adapter

Purpose:

```text
Allows the VM to access the internet through the host machine.
```

Used for:

- Downloading Ubuntu packages
- Installing Elastic/Wazuh packages later
- Updating tools
- Accessing public repositories


## Host-Only Adapter

Purpose:

```text
Allows VMs to communicate with each other on a private VmWare network.
```

Used for:

- Windows victim talking to Ubuntu SIEM
- Ubuntu SIEM receiving logs
- Kali testing against the lab machines
- Wazuh communicating with agents in later phases

Typical host-only network range:

```text
192.168.56.0/24
```

Example lab IPs:

```text
Ubuntu SIEM VM:       192.168.56.101
Windows 10 Victim VM: 192.168.56.104
Wazuh Server later:   192.168.56.105
```

---

# 4. Phase 1 Network Diagram

Basic Phase 1 layout:

```text
                    Internet
                       |
                 NAT Adapter
                       |
          -------------------------
          |                       |
     Ubuntu SIEM VM          Windows Victim VM
     192.168.56.101         192.168.56.104
          |                       |
          -------- Host-only -------
                    Network
                       |
                  Kali Linux VM
              192.168.56.x later
```

Later phases added:

```text
Wazuh Server VM
192.168.56.105
```

But Phase 1 created the foundation.

---

# 5. VM 1 - Ubuntu SIEM VM

## Purpose

The Ubuntu SIEM VM was created to become the central security monitoring server.

Later phases used this VM to install:

- Elasticsearch
- Kibana
- Elastic Agent
- Fleet Server
- Suricata IDS
- Dashboards
- Log ingestion pipelines

## Recommended Ubuntu VM Settings

Suggested settings:

```text
Name: Ubuntu-SIEM or SIEM-UbuntuVM
Type: Linux
Version: Ubuntu 64-bit
RAM: 8 GB preferred if available
CPU: 2-4 cores
Disk: 80 GB dynamically allocated
Network Adapter 1: NAT
Network Adapter 2: Host-only Adapter
```

## Why These Specs Matter

Elastic and Kibana are resource-heavy. The SIEM VM needs more memory and disk than a basic Linux VM.

If resources are limited, the SIEM VM should be prioritized because it runs the main logging stack.

---

# 6. VM 2 - Windows 10 Victim VM

## Purpose

The Windows 10 victim VM acts as the monitored endpoint.

This VM is where user and endpoint activity happens. Later phases installed:

- Sysmon
- Elastic Agent
- Wazuh Agent
- Windows logging integrations

This VM generated logs such as:

- Process creation
- DNS queries
- Network connections
- Login events
- Failed logons
- Account changes
- File activity
- PowerShell activity

## Recommended Windows VM Settings

```text
Name: windows10victim
Type: Microsoft Windows
Version: Windows 10 64-bit
RAM: 4 GB minimum
CPU: 2 cores minimum
Disk: 60-80 GB dynamically allocated
Network Adapter 1: NAT
Network Adapter 2: Host-only Adapter
```

## Why Windows Was Needed

Most entry-level SOC jobs involve Windows logs. Windows endpoints generate many of the events analysts review daily:

- 4624 successful logon
- 4625 failed logon
- 4672 special privileges
- 4720 user created
- 4732 user added to local group
- Sysmon Event ID 1 process creation
- Sysmon Event ID 3 network connection
- Sysmon Event ID 22 DNS query

Phase 1 created the endpoint that would later generate all of that telemetry.

---

# 7. VM 3 - Kali Linux VM

## Purpose

Kali Linux was included as the attacker/testing machine for later phases.

Can use Kali for:

- Nmap scans
- Safe traffic generation
- Testing detection rules
- Simulated reconnaissance
- IDS alert generation
- Controlled attack exercises

## Recommended Kali VM Settings

```text
Name: kali-lab
Type: Linux
Version: Debian 64-bit
RAM: 2-4 GB
CPU: 2 cores
Disk: 40-60 GB dynamically allocated
Network Adapter 1: NAT
Network Adapter 2: Host-only Adapter
```

## Why Kali Was Included

SOC analysts do not need to become penetration testers immediately, but understanding attacker behavior helps them recognize suspicious activity.

Kali gives a safe way to generate controlled scanning and attack-like activity later in the roadmap.

---

# 8. Installation Process Overview

The general VM installation process for each machine followed this pattern:

```text
1. Download ISO
2. Create VM in VmWare
3. Assign RAM/CPU/disk
4. Attach ISO
5. Configure NAT + Host-only adapters
6. Install OS
7. Log in
8. Confirm IP addresses
9. Test internet access
10. Test VM-to-VM connectivity
11. Take snapshot
```

This process matters because every future phase depends on clean networking and repeatable VM states.

---

# 9. Ubuntu Installation Steps


## Configure Network

Use:

```text
Adapter 1: NAT
Adapter 2: Host-only Adapter
```

## Install Ubuntu



## Install OpenSSH Server

If prompted, install OpenSSH server or install it later:

```bash
sudo apt update
sudo apt install -y openssh-server
```

Why:

```text
SSH allows easier copy/paste and administration from the host laptop.
```

## Confirm IP Addresses

Run:

```bash
ip addr
```

Look for:

```text
NAT IP: usually 10.x.x.x
Host-only IP: usually 192.168.56.x
```

The SIEM VM was later referenced as:

```text
192.168.56.101
```

---

# 10. Windows 10 Installation Steps

## Configure Network

Use:

```text
Adapter 1: NAT
Adapter 2: Host-only Adapter
```

## Confirm IP

Open PowerShell:

```powershell
ipconfig
```

Look for:

```text
Host-only IPv4 Address: 192.168.56.x
```

The Windows victim later used:

```text
192.168.56.104
```

---

# 11. Kali Installation Steps

## Configure Network

Use:

```text
Adapter 1: NAT
Adapter 2: Host-only Adapter
```

## Confirm IP

Run:

```bash
ip addr
```

Confirm the host-only address.

---

# 12. Basic Connectivity Testing

Once the VMs were installed, networking needed to be verified.

## Test Internet from Ubuntu

```bash
ping -c 4 google.com
```

Expected result:

```text
Replies received
```

If this works, NAT internet access is working.

## Test Internet from Windows

```powershell
ping google.com
```

or:

```powershell
nslookup google.com
```

Expected result:

```text
DNS resolves and network responds
```

## Test Lab Connectivity

From Ubuntu SIEM to Windows:

```bash
ping -c 4 192.168.56.104
```

From Windows to Ubuntu SIEM:

```powershell
ping 192.168.56.101
```

Expected result:

```text
VMs can reach each other over host-only network
```

---

# 13. Windows Firewall Note

Sometimes Windows does not respond to ping because Windows Defender Firewall blocks ICMP echo requests.

Important lesson:

```text
A failed ping does not always mean the host is unreachable.
It may mean the firewall is blocking ICMP.
```

If needed, Windows firewall ICMP rules can be enabled later. However, for many later phases, agent communication can still work even if ping is blocked.

SOC lesson:

```text
Network troubleshooting requires understanding both connectivity and firewall behavior.
```

---

# 14. SSH Access to Ubuntu SIEM

Once Ubuntu had OpenSSH installed, the host laptop could connect by SSH.

From Windows PowerShell:

```powershell
ssh <username>@192.168.56.101
```

Example:

```powershell
ssh mmajeed@192.168.56.101
```

Why SSH matters:

- Easier copy/paste
- Easier command execution
- More realistic Linux administration
- Better than typing long commands inside VmWare console

---

# 15. Why Snapshots Were Important

Snapshots were taken after clean setup milestones.

A snapshot preserves the VM state so the lab can be restored if something breaks.

Recommended Phase 1 snapshots:

```text
Ubuntu SIEM:
Phase1-Clean-Ubuntu-Networking-Working

Windows 10 Victim:
Phase1-Clean-Windows-Networking-Working

Kali:
Phase1-Clean-Kali-Networking-Working
```

Why snapshots matter:

- Security tools can break configs.
- Package installs can fail.
- Network settings can get misconfigured.
- It is faster to revert than rebuild from scratch.
- Professional lab work requires rollback points.

---

# 16. Phase 1 Issues and Troubleshooting

## Issue 1 - Default IP Address configuration


## Issue 2 - Assign enough space for a VM to run 

The laptop has limited resources, so running every VM at the same time can cause lag or service problems.


## Issue 3 - Need for SSH / Copy-Paste

Typing long commands directly into the VmWare console is inefficient and error-prone.

Resolution:

Install and use SSH for Linux VMs.

```bash
sudo apt install -y openssh-server
```

Then connect from host:

```powershell
ssh <username>@<host-only-ip>
```

Lesson:

```text
SSH improves workflow and mirrors real Linux administration.
```

---

# 17. What Phase 1 Proved

By the end of Phase 1, the lab had a working virtual foundation.

Confirmed:

- VmWare installed and usable
- Ubuntu SIEM VM created
- Windows 10 victim VM created
- Kali Linux VM created or planned as attacker/testing machine
- NAT internet access configured
- Host-only lab network configured
- VMs could be assigned private lab IPs
- SSH access could be used for Linux administration
- Snapshots could protect progress
- The lab was ready for Elastic, Sysmon, Suricata, and Wazuh in future phases

---

# 18. Why Phase 1 Matters for SOC Analyst Skills

Phase 1 may look like basic setup, but it maps directly to real IT/security work.

A SOC analyst does not only look at alerts. They also need to understand:

- Hosts
- IP addressing
- Network paths
- Internal vs external traffic
- Firewalls
- Services
- Logs
- Troubleshooting
- System roles
- Endpoint vs server responsibilities

Phase 1 introduced those concepts through hands-on setup.

---

# 19. Interview Translation

A strong way to explain Phase 1 in an interview:

```text
I built a virtual SOC lab using VmWare with separate Ubuntu, Windows, and Kali virtual machines. I configured NAT networking for internet access and host-only networking for private lab communication. The Windows VM acts as the monitored endpoint, the Ubuntu VM acts as the SIEM server, and Kali is reserved for controlled testing and traffic generation. I validated connectivity using ipconfig, ip addr, ping, and SSH, then took clean snapshots before installing security tools. This gave me a safe environment to build Elastic, Wazuh, Sysmon, Suricata, and future SOC investigations without affecting my real network.
```

---

# 20. Community Explanation for Aspiring SOC Analysts

If someone new to SOC labs asks why Phase 1 matters, explain it like this:

```text
Before you can investigate alerts, you need machines that create alerts and a place to collect them. Phase 1 builds that environment. The Windows VM becomes the endpoint. The Ubuntu VM becomes the SIEM. Kali becomes the testing machine. NAT gives the VMs internet access, and host-only networking lets the lab machines talk privately. Once that works, you can safely install logging agents, generate events, and practice real SOC workflows.
```

---

# 21. Phase 1 Checklist

Completed or established:

- VmWare used as the hypervisor
- Ubuntu SIEM VM created
- Windows 10 victim VM created
- Kali Linux testing VM created/planned
- NAT networking configured
- Host-only networking configured
- VM IP addresses identified
- Internet connectivity tested
- VM-to-VM connectivity tested
- SSH access established for Ubuntu
- Snapshot strategy established
- Lab roles defined
- Lab ready for Phase 2 Elastic/Sysmon/Suricata work

---

# 22. Final Phase 1 Summary

Phase 1 created the foundation for the entire SOC Analyst home lab. The lab was designed around a realistic security operations structure: a monitored Windows endpoint, an Ubuntu-based SIEM server, and a Kali testing machine. VmWare networking was configured with NAT for internet access and host-only networking for private lab communication.

The phase also introduced important troubleshooting concepts such as IP identification, adapter roles, ping behavior, firewall considerations, SSH access, and snapshot management. This foundation made the later phases possible, including Elastic SIEM deployment, Sysmon telemetry, Suricata IDS, Wazuh XDR, ticket writing, and SOC investigation practice.

Phase 1 is complete.
