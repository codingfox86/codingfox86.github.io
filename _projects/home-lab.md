---
title: "Virtual Pentesting Lab"
description: Personal penetration testing lab built with VirtualBox for practicing exploitation and network attacks
date: 2026-03-05
image:
path: /assets/img/projects/home-lab/lab-overview.png
alt: Virtual pentesting lab architecture
---

## Overview

This project documents the setup of my personal penetration testing lab using VirtualBox.  
The goal of this environment is to provide a safe and isolated space for practicing penetration testing techniques, vulnerability exploitation, and Active Directory attack scenarios.

The lab provides a safe and controlled environment where attacks and defensive techniques can be tested without impacting real systems.

---

## Lab Architecture

The lab runs on a Windows host system using VirtualBox and consists of multiple virtual machines connected through an isolated internal network.

{% highlight js %}
LAB_NET 192.168.56.0/24

Kali Linux (Attacker)
│ 192.168.56.10
│
├── Ubuntu Server (Docker Lab Host)
│ 192.168.56.20
│
├── Metasploitable2 (Vulnerable Linux Target)
│ 192.168.56.30
│
└── Windows Server 2019 (Domain Controller - future)
192.168.56.40
{% endhighlight %}


---

## Lab Goals

The objectives of this environment include:

- Practicing penetration testing techniques
- Simulating a small enterprise network
- Performing vulnerability assessments and exploitation
- Testing Active Directory attack techniques

---

## Technologies Used

The following technologies are used to build the lab environment:

- VirtualBox
- Kali Linux
- Ubuntu Server
- Metasploitable2
- Windows Server 2019

---

## Network Configuration

All machines are connected to an isolated VirtualBox internal network called **LAB_NET**.

| Machine | IP Address | Role |
|--------|------------|------|
| Kali Linux | 192.168.56.10 | Attacker |
| Ubuntu Server | 192.168.56.20 | Docker Lab Host |
| Metasploitable2 | 192.168.56.30 | Vulnerable Linux Target |
| Windows Server | 192.168.56.40 | Domain Controller (future) |

![network-config](/assets/img/projects/home-lab/01.png)

_Network configuration for Kali Linux._

---

## Example Test

After setting up the environment, network connectivity and services can be tested from the Kali attacker machine.

Example scan from Kali Linux:
![nmap scan](/assets/img/projects/home-lab/02.png)

The scan reveals intentionally vulnerable services running on the Metasploitable machine.

This allows safe exploitation practice and vulnerability analysis.

---

## Future Improvements

Planned expansions for the lab include:

- Active Directory domain setup
- Windows client machine
- Vulnerable Docker applications

