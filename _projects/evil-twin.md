---
title: "Evil Twin Wi-Fi Lab"
---

## Overview

This project explores the practical setup of an Evil Twin Wi-Fi attack lab using a Raspberry Pi and Parrot OS. It demonstrates wireless attack simulation, captive portal deployment, credential capture and logging in a controlled environment. The objective is to better understand real-world attack techniques and strengthen defensive and detection capabilities.

---

## Introduction

This is the documentation of my ongoing project on how to set up and simulate an Evil Twin attack with a Raspberry Pi for educational purposes.

An Evil Twin attack is a type of Wi-Fi phishing attack where a hacker sets up a rogue access point (AP) that mimics a legitimate wireless network. The goal is to trick users into connecting to the fake AP, allowing the attacker to intercept sensitive information such as login credentials or session cookies.

This project demonstrates how to simulate an Evil Twin attack in a controlled, ethical environment for educational purposes only.

By following this guide you will:

- Understand how Evil Twin attacks work in practice
- Learn how to configure a rogue AP using airgeddon
- Simulate a credential capture scenario using a fake captive portal
- Explore ways to detect and mitigate such attacks

> ⚠️ Disclaimer: This project must only be conducted in isolated environments with explicit permission from all participants. Unauthorized deployment of rogue access points is illegal and unethical.

---

## Objectives

- Set up a Raspberry Pi or Linux-based system as an Evil Twin
- Create a cloned wireless network to simulate phishing
- Capture credentials using a fake login page
- Log and monitor client connections
- Reflect on defense techniques against such attacks

---

## Tools & Environment

- Raspberry Pi or any Linux system (Parrot OS or Kali Linux recommended)
- External USB Wi-Fi adapter that supports monitor mode and packet injection
- Tool: airgeddon
- SSH connection (optional but recommended for easier handling)
- HTML/PHP files for a basic captive portal

---

## Preparation

### What you need to do before you can start with your rogue AP

**Note:** Since there were not many available Wi-Fi adapters and none compatible with Kali Linux, I used the following setup:

- Raspberry Pi 4B with 4GB RAM (including 32GB SD-Card, HDMI adapter and power supply)
- Parrot OS 64-bit Security version  
  https://deb.parrot.sh/parrot/iso/6.4/Parrot-security-6.4_rpi.img.xz
- Wi-Fi Adapter BrosTrend AC650

If you prefer working with Kali Linux, feel free to do so.

---

### 1. Download Raspberry Pi Imager

Download for your OS:

https://www.raspberrypi.com/software/

---

### 2. Install Parrot or Kali on the SD card

Install the OS on the SD card using Raspberry Pi Imager.

Choose:

- Your Raspberry Pi model
- Your operating system
- Your SD card

Some OS images are available directly inside the imager.

It is recommended to:

- Configure your home Wi-Fi
- Enable SSH access
- Set credentials

---

### 3. First Boot

Assemble your Raspberry Pi and start it.

Using a monitor is recommended. Connecting only via SSH initially can cause configuration difficulties.

---

### 4. Login

Default credentials are often:
- user: pi
- password: raspberrypi
Or your chosen credentials. You logged in succesfully? Great!

After login, update the system:

sudo apt update && sudo apt upgrade

---

### 5. Setup

Connect to Wi-Fi (if not already connected)
ip addr show (for me it is wlan0)
nmcli device wifi connect "SSID" password "password" ifname wlan0

Keep the project organized:
mkdir -p ~/evil-twin/{configs,scripts,logs,pcap,portal}

Check current graphical target:
sudo systemctl get-default
If it is: multi-user.target
Switch to graphical mode:
sudo systemctl set-default graphical.target
sudo reboot

Connect external Wi-Fi adapter
Verify detection: 
lsusb
And install software

# YOU ARE READY TO GO!!


### Monitoring

Coming soon

### Reflection

Coming soon
