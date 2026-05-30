# Installation and Setup Guide

## Project Overview

This project demonstrates a beginner-friendly Intrusion Detection System (IDS) lab environment built using Suricata inside Oracle VirtualBox.

The main objective of this lab was to:

* Learn IDS fundamentals
* Understand network traffic monitoring
* Configure Suricata IDS
* Simulate attack traffic safely
* Analyze alerts and packet behavior
* Gain practical SOC-related experience

The entire environment was isolated inside a virtual lab network for safe cybersecurity testing.

---

# Lab Environment Architecture

```text
Kali Linux (Attacker)
        |
        | 192.168.56.0/24
        |
Ubuntu Server + Suricata IDS
        |
Metasploitable2 (Target)
```

---

# Technologies Used

| Tool              | Purpose                                |
| ----------------- | -------------------------------------- |
| Oracle VirtualBox | Virtualization platform                |
| Kali Linux        | Attack simulation                      |
| Ubuntu Server     | IDS monitoring server                  |
| Suricata          | Intrusion Detection System             |
| Metasploitable2   | Vulnerable target machine              |
| tcpdump           | Packet capture verification            |
| Nmap              | Network scanning and attack simulation |

---

# Step 1 — Install Oracle VirtualBox

## Download VirtualBox

Official website:

```text
https://www.virtualbox.org/
```

Download:

* VirtualBox platform package
* Extension Pack (optional but recommended)

---

# Why VirtualBox Was Used

VirtualBox allows multiple operating systems to run inside a single physical computer using virtualization.

This lab required:

* isolated environments
* multiple operating systems
* safe attack simulation
* internal network communication

Without virtualization, building this lab would require multiple physical computers.

---

# Step 2 — Download Operating Systems

## Kali Linux

Official website:

```text
https://www.kali.org/get-kali/
```

Purpose:

* attacker machine
* penetration testing tools
* traffic generation

---

## Ubuntu Server

Official website:

```text
https://ubuntu.com/download/server
```

Purpose:

* host Suricata IDS
* monitor traffic
* generate alerts

Ubuntu Server was chosen because:

* lightweight
* stable
* commonly used in servers and SOC environments

---

## Metasploitable2

Official website:

```text
https://sourceforge.net/projects/metasploitable/
```

Purpose:

* intentionally vulnerable machine
* target for attack simulation
* security testing practice

Metasploitable2 is designed for cybersecurity learning purposes only.

---

# Step 3 — Create Virtual Machines

## Kali Linux VM

### Recommended Configuration

| Setting | Value            |
| ------- | ---------------- |
| RAM     | 4 GB             |
| CPU     | 2 cores          |
| Storage | 30 GB            |
| Network | Internal Network |

---

## Ubuntu Server VM

### Recommended Configuration

| Setting           | Value            |
| ----------------- | ---------------- |
| RAM               | 4 GB             |
| CPU               | 2 cores          |
| Storage           | 25 GB            |
| Network Adapter 1 | Internal Network |
| Network Adapter 2 | NAT              |

---

## Metasploitable2 VM

Metasploitable2 was imported directly using the provided virtual machine image.

### Recommended Configuration

| Setting | Value            |
| ------- | ---------------- |
| RAM     | 2 GB             |
| CPU     | 1 core           |
| Network | Internal Network |

---

# Step 4 — Configure Internal Network

All machines were connected to a VirtualBox Internal Network named:

```text
SOC-LAB
```

---

# Why Internal Network Was Used

Internal Network mode creates:

* isolated communication between virtual machines
* safe attack environment
* no exposure to external networks

This allowed attack traffic to remain inside the lab environment.

---

# Why NAT Was Added to Ubuntu Server

Initially, Ubuntu Server had only Internal Network configured.

Problem:

* no internet access
* unable to install Suricata packages
* unable to update repositories

Solution:

* added second network adapter using NAT

Purpose of NAT:

* provide internet access
* download packages
* update Suricata rules

Ubuntu Server therefore used:

* Internal Network for IDS monitoring
* NAT for internet connectivity

---

# Step 5 — Configure Static IP Addresses

Static IPs were assigned to maintain consistent communication between virtual machines.

| Machine         | IP Address    |
| --------------- | ------------- |
| Kali Linux      | 192.168.56.10 |
| Metasploitable2 | 192.168.56.20 |
| Ubuntu Server   | 192.168.56.30 |

---

# Why Static IPs Were Important

Dynamic IP addresses can change after reboot.

IDS systems require:

* predictable addresses
* stable communication
* consistent monitoring targets

Static addressing simplified:

* attack simulation
* packet analysis
* rule creation

---

# Step 6 — Configure Ubuntu Server Networking

Ubuntu Server uses Netplan for network configuration.

Network configuration file:

```bash
/etc/netplan/50-cloud-init.yaml
```

Example configuration:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.56.30/24
```

---

# Apply Network Configuration

```bash
sudo netplan apply
```

---

# Verify Configuration

```bash
ip a
```

This command displays:

* network interfaces
* IP addresses
* interface status

---

# Step 7 — Verify Network Communication

Connectivity between machines was verified using:

```bash
ping
```

Example:

```bash
ping 192.168.56.30
```

This confirmed:

* machines could communicate
* network configuration was working correctly

---

# Step 8 — Install Suricata IDS

## Update Package Repository

```bash
sudo apt update
```

---

## Install Suricata

```bash
sudo apt install suricata -y
```

---

# Verify Installation

```bash
suricata --build-info
```

This command displays:

* installed version
* packet capture support
* enabled features

---

# Step 9 — Configure Suricata

Main configuration file:

```bash
/etc/suricata/suricata.yaml
```

---

# Configure HOME_NET

The HOME_NET variable defines the protected network.

Configuration:

```yaml
HOME_NET: "[192.168.56.0/24]"
```

Purpose:

* tell Suricata which network should be monitored

---

# Configure Monitoring Interface

Suricata was configured to monitor:

```text
enp0s3
```

Configuration section:

```yaml
af-packet:
  - interface: enp0s3
```

---

# Step 10 — Download Detection Rules

```bash
sudo suricata-update
```

Purpose:

* download Suricata rule sets
* enable attack detection
* update threat intelligence rules

---

# Step 11 — Create Custom Detection Rule

Custom rule file:

```bash
/var/lib/suricata/rules/local.rules
```

Rule:

```rules
alert icmp any any -> any any (msg:"ICMP Ping Detected"; sid:1000001; rev:1;)
```

Purpose:

* generate alert whenever ICMP ping traffic is detected

---

# Step 12 — Validate Suricata Configuration

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Purpose:

* validate configuration syntax
* verify rule loading
* confirm interface configuration

---

# Step 13 — Start Suricata

```bash
sudo suricata -i enp0s3
```

This started:

* packet capture
* traffic inspection
* intrusion detection engine

---

# Step 14 — Generate Attack Traffic

Attack traffic was generated from Kali Linux using:

## Ping Test

```bash
ping 192.168.56.30
```

---

## Nmap Scan

```bash
nmap -A 192.168.56.30
```

Purpose:

* simulate reconnaissance traffic
* generate detectable packets
* test IDS behavior

---

# Step 15 — Monitor Alerts

Live alerts were monitored using:

```bash
sudo tail -f /var/log/suricata/eve.json
```

Filtered alerts:

```bash
sudo tail -f /var/log/suricata/eve.json | grep "ICMP Ping Detected"
```

---

# Final Outcome

Successfully implemented:

* isolated IDS lab
* Suricata intrusion detection
* packet monitoring
* custom detection rules
* real-time alert generation
* attack traffic simulation

This project provided practical understanding of:

* Linux networking
* IDS architecture
* packet analysis
* network visibility
* Suricata rule processing
* SOC fundamentals
