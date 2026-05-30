# Commands and Technical Explanations

## Overview

This document contains the major Linux, networking, Suricata, and troubleshooting commands used throughout the Suricata IDS Lab project.

Each command includes:

* syntax
* purpose
* technical explanation
* practical usage inside the lab

The objective of this document is not only to memorize commands, but also to understand why each command was required during the setup and troubleshooting process.

---

# Linux Networking Commands

---

# 1. Display Network Interfaces

```bash
ip a
```

## Purpose

Displays:

* network interfaces
* IP addresses
* interface states
* subnet information

---

## Why It Was Important

Used to:

* identify Ubuntu network interface names
* verify static IP configuration
* determine which interface Suricata should monitor

---

## Important Lesson

Linux interface names are critical in IDS systems.

Using the wrong interface causes:

* packet capture failure
* missing alerts
* traffic visibility problems

---

# 2. Test Network Connectivity

```bash
ping 192.168.56.30
```

---

## Purpose

Sends ICMP echo requests to another machine.

Used to verify:

* communication between VMs
* network connectivity
* packet visibility

---

## Why It Was Important

Used to:

* confirm Internal Network configuration
* generate detectable traffic
* test custom Suricata ICMP rules

---

# 3. Packet Capture Verification

```bash
sudo tcpdump -i enp0s3
```

---

## Purpose

Captures and displays raw network packets from a selected interface.

---

## Parameter Breakdown

| Parameter | Meaning              |
| --------- | -------------------- |
| sudo      | run as administrator |
| tcpdump   | packet capture tool  |
| -i        | select interface     |
| enp0s3    | monitored interface  |

---

## Why It Was Important

Used to verify:

* whether traffic actually reached Ubuntu Server
* if Suricata visibility issues were networking-related
* packet flow between virtual machines

---

## Important Lesson

Packet capture verification is one of the first troubleshooting steps in SOC environments.

Before debugging IDS rules, engineers must first confirm:

* traffic exists
* packets reach monitoring interface

---

# Suricata Commands

---

# 4. Install Suricata

```bash
sudo apt install suricata -y
```

---

## Purpose

Installs the Suricata IDS engine.

---

## Important Lesson

Suricata is not only an IDS.

It can function as:

* IDS
* IPS
* network security monitor
* protocol analyzer

depending on configuration.

---

# 5. Validate Suricata Configuration

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

---

## Purpose

Validates:

* configuration syntax
* rule loading
* interface configuration
* YAML formatting

without starting live monitoring.

---

## Parameter Breakdown

| Parameter | Meaning             |
| --------- | ------------------- |
| -T        | test mode           |
| -c        | specify config file |

---

## Why It Was Important

Used repeatedly during troubleshooting to:

* detect YAML mistakes
* verify rule loading
* confirm Suricata configuration correctness

---

## Important Lesson

Configuration validation should always be performed before starting production services.

---

# 6. Start Suricata in Live Monitoring Mode

```bash
sudo suricata -i enp0s3
```

---

## Purpose

Starts the Suricata engine and begins:

* packet capture
* traffic inspection
* intrusion detection

---

## Parameter Breakdown

| Parameter | Meaning                     |
| --------- | --------------------------- |
| -i        | interface selection         |
| enp0s3    | monitored network interface |

---

## Why It Was Important

Used to:

* monitor live attack traffic
* detect ICMP packets
* generate alerts in real time

---

## Important Lesson

Running Suricata in foreground mode provides:

* direct debugging visibility
* live packet statistics
* runtime monitoring information

This was more useful than systemd service debugging during the setup process.

---

# 7. Update Suricata Rules

```bash
sudo suricata-update
```

---

## Purpose

Downloads and updates Suricata detection rules.

---

## Why It Was Important

Without rules:

* Suricata captures traffic
* but cannot generate meaningful alerts

Rules provide:

* attack signatures
* threat intelligence
* detection logic

---

## Important Lesson

IDS systems rely heavily on:

* signature databases
* rule feeds
* threat intelligence updates

---

# 8. Monitor Suricata Alerts

```bash
sudo tail -f /var/log/suricata/eve.json
```

---

## Purpose

Displays live Suricata events from:

```text
eve.json
```

---

## Parameter Breakdown

| Parameter | Meaning             |
| --------- | ------------------- |
| tail      | display end of file |
| -f        | follow live updates |

---

## Why It Was Important

Used to:

* observe live alerts
* monitor packet analysis
* verify custom rule detection

---

## Important Lesson

Modern Suricata deployments mainly use:

```text
eve.json
```

instead of:

```text
fast.log
```

because JSON telemetry is easier to integrate with SIEM platforms.

---

# 9. Filter Specific Alerts

```bash
sudo tail -f /var/log/suricata/eve.json | grep "ICMP Ping Detected"
```

---

## Purpose

Filters only matching alerts from live Suricata logs.

---

## Why It Was Important

Reduced noise and displayed only:

* custom ICMP alerts
* relevant detection events

---

## Important Lesson

Filtering logs is critical in SOC environments because raw telemetry can become extremely noisy.

---

# Linux Service Management Commands

---

# 10. Start Suricata Service

```bash
sudo systemctl start suricata
```

---

## Purpose

Starts Suricata as a background system service.

---

# 11. Enable Suricata at Boot

```bash
sudo systemctl enable suricata
```

---

## Purpose

Automatically starts Suricata when Ubuntu boots.

---

# 12. Check Service Status

```bash
sudo systemctl status suricata
```

---

## Purpose

Displays:

* service state
* runtime information
* startup failures
* process status

---

## Important Lesson

Most Linux-based cybersecurity tools operate as:

* background services
* daemons
* continuously running monitoring systems

---

# 13. View Detailed Service Logs

```bash
sudo journalctl -xeu suricata.service
```

---

## Purpose

Displays detailed service logs and startup errors.

---

## Why It Was Important

Used during troubleshooting when:

* Suricata service failed
* startup errors occurred
* runtime issues needed investigation

---

# Nmap Commands

---

# 14. Basic Nmap Scan

```bash
nmap 192.168.56.30
```

---

## Purpose

Performs a basic port scan against a target machine.

---

# 15. Aggressive Nmap Scan

```bash
nmap -A 192.168.56.30
```

---

## Purpose

Performs:

* OS detection
* service detection
* traceroute
* script scanning

---

## Why It Was Important

Generated:

* suspicious traffic
* reconnaissance activity
* detectable network patterns

Used to test Suricata visibility and detection capabilities.

---

# 16. SYN Scan

```bash
nmap -sS 192.168.56.30
```

---

## Purpose

Performs a TCP SYN scan (half-open scan).

---

## Important Lesson

SYN scans:

* are faster
* generate suspicious traffic
* are commonly used during reconnaissance

Many IDS systems monitor this behavior.

---

# File Editing Commands

---

# 17. Edit Suricata Configuration

```bash
sudo nano /etc/suricata/suricata.yaml
```

---

## Purpose

Opens the main Suricata configuration file.

---

## Why It Was Important

Used to configure:

* HOME_NET
* monitored interface
* rule files
* logging settings

---

# 18. Edit Local Rules

```bash
sudo nano /var/lib/suricata/rules/local.rules
```

---

## Purpose

Creates or modifies custom Suricata rules.

---

## Why It Was Important

Used to create:

```rules
alert icmp any any -> any any (msg:"ICMP Ping Detected"; sid:1000001; rev:1;)
```

---

# 19. Search Inside Files

Inside nano editor:

```text
CTRL + W
```

---

## Purpose

Searches for specific configuration sections.

---

## Why It Was Important

Used to quickly locate:

* HOME_NET
* af-packet
* rule-files

inside large configuration files.

---

# Git Commands

---

# 20. Initialize Git Repository

```bash
git init
```

---

## Purpose

Initializes local Git repository tracking.

---

# 21. Stage Files

```bash
git add .
```

---

## Purpose

Stages all project files for commit.

---

# 22. Create Commit

```bash
git commit -m "Initial Suricata IDS Lab Setup"
```

---

## Purpose

Creates a snapshot of project changes.

---

# 23. Push Repository to GitHub

```bash
git push -u origin main
```

---

## Purpose

Uploads local project to GitHub repository.

---

# Final Technical Lessons

This project provided practical understanding of:

* Linux networking
* packet capture
* IDS architecture
* network visibility
* Suricata configuration
* troubleshooting methodology
* detection engineering fundamentals

The most important realization during the project was:

```text
Packet visibility, packet capture, detection rules, and alert generation are separate stages inside an IDS pipeline.
```
