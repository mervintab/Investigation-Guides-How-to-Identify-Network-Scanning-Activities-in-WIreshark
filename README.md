# 🛡️ Wireshark Network Scan Investigation Guide

*A SOC Analyst Training Resource*

## 📘 Overview

This repository contains a comprehensive guide for identifying and analyzing **network scanning activity** in packet captures (PCAPs) using **Wireshark**.

The guide is designed for entry-level and intermediate SOC analysts who want a practical, repeatable workflow for:

* Detecting scanning hosts
* Identifying horizontal and vertical scan patterns
* Understanding TCP/ICMP response behaviors
* Determining which ports are open, closed, or filtered
* Building proper incident documentation

The included guide also maps each major investigative step to:

* **NIST 800-61** (Incident Handling Process)
* **NIST CSF** (Detect / Respond)
* **MITRE ATT&CK** (T1046 – Network Service Discovery)
* **CIS Controls** (Logging & Network Monitoring)

---

## 📄 Contents

This repository includes:

* **Guide PDF** – A full workflow for identifying network scanning activity
* **Screenshot placeholders** – Replace these with your own Wireshark captures
* **PCAP sample (optional)** – For practice and demonstration
* **Documentation templates** – Ready-to-use incident note formats

---

## 📁 Recommended Repository Structure

```
/wireshark-network-scan-guide/
│
├── guide.pdf                     # Final guide exported to PDF
├── README.md                     # Repository overview (this file)
├── screenshots/                  # Replace placeholders with your own
│     ├── endpoints_view.png
│     ├── syn_probes.png
│     ├── syn_ack_open_ports.png
│     ├── rst_closed_ports.png
│     ├── icmp_filtered.png
│     └── conversations.png
│
└── DISCOVERY-SCAN.pcap           # Sample PCAP (optional)
```

---

## 🧰 What This Guide Will Teach You

The included guide walks through:

* How to identify the scanning IP
* How to distinguish horizontal vs vertical scanning
* Key Wireshark filters (SYN, SYN/ACK, RST, ICMP)
* How to identify **open**, **closed**, and **filtered** ports
* How to correlate scan behavior across multiple hosts
* How to document evidence like a SOC analyst

---

## 🖼️ Screenshots

Screenshot placeholders are embedded in the guide for:

* Endpoints view
* SYN packet filtering
* SYN/ACK open port identification
* RST closed port identification
* ICMP unreachable analysis
* Conversations window

Replace the placeholder images with your own captures from Wireshark.

---

## 🧪 Sample PCAP

If you included a PCAP file (like `DISCOVERY-SCAN.pcap`), you can use it to:

* Practice recognizing scan patterns
* Identify responsive vs. unresponsive hosts
* Build your own notes and incident summary

---

## 📜 License

This guide is provided for educational and defensive security purposes only.

---

## 🙌 Contributing

Ideas for improvements? Submit a pull request or open an issue!

---

## ⭐ Support

If this guide helps you develop your SOC analyst skills, consider starring the repository on GitHub!
