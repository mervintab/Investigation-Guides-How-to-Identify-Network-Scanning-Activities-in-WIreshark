# **Wireshark Network Scan Investigation Guide**
*A step‑by‑step SOC Analyst learning guide using packet captures (PCAPs)*

---

## **Table of Contents**
1. Introduction
2. Understanding Network Scans
3. Key TCP/ICMP Behaviors to Know
4. Workflow Overview
5. Step‑by‑Step Investigation
6. Identifying Open, Closed, and Filtered Ports
7. Useful Wireshark Filters
8. Evidence Collection Checklist
9. Example Investigation Flow (Using Your PCAP)
10. Analyst Documentation Template

---

# **1. Introduction**
This guide teaches you how to investigate **network scanning activity** using Wireshark. It covers:
- How to identify the source of a scan
- How to detect horizontal and vertical scanning patterns
- How to determine which IPs responded
- How to identify which ports appear open, closed, or filtered
- How to document findings like a SOC analyst

This workflow aligns with:
- **NIST 800‑61 (Incident Handling)**
- **NIST CSF (Detect & Respond)**
- **MITRE ATT&CK (Discovery — T1046 Network Service Discovery)**
- **CIS Controls (Network Monitoring & Logging)**

---

# **2. Understanding Network Scans**
A network scan is an attempt to identify:
- Live hosts
- Open ports
- Running services
- Potential vulnerabilities

Typical scan types:
- **Horizontal scan**: same port, multiple IPs
- **Vertical scan**: same IP, multiple ports
- **ICMP sweep**: ping multiple hosts
- **Stealth scans**: NULL, FIN, Xmas flag scans

---

# **3. Key TCP/ICMP Behaviors to Know**
Before diving into Wireshark, understand these standard behaviors:

### **Open TCP port → SYN/ACK response**
Scanner sends SYN → Target replies with **SYN/ACK**.

### **Closed TCP port → RST response**
Scanner sends SYN → Target replies with **RST**.

### **Filtered port → No response or ICMP unreachable**
Firewalls often drop packets silently.
Common ICMP unreachable codes:
- Code 1: Host unreachable
- Code 3: Port unreachable
- Code 13: Administratively filtered

---

# **4. Workflow Overview**
1. Identify top talkers
2. Find the suspected scanner IP
3. Filter SYN packets from scanner
4. Analyze replies: SYN/ACK, RST, ICMP
5. Map results to IP/port list
6. Classify the scan type
7. Document and report

---

# **5. Step‑by‑Step Investigation Workflow**
Each step includes Action + Reasoning for SOC learning.

## **Step 1 — Identify high‑volume IPs**
**Action:**  
Go to **Statistics → Endpoints → IPv4**.

**Reasoning:**  
Scanning hosts often show abnormally high packet counts to many targets.

---

## **Step 2 — Identify potential scanner IP**
**Action:**  
Sort by packet count or number of peers.

**Reasoning:**  
The scanner usually has traffic going *out* to many different IP addresses.

---

## **Step 3 — Filter on suspected scanner**
**Action:**
Use:
```
ip.addr == <scanner_ip>
```
**Reasoning:**  
This isolates traffic just from that source.

---

## **Step 4 — Show all SYN probes**
**Action:**
```
tcp.flags.syn == 1 && tcp.flags.ack == 0 && ip.src == <scanner_ip>
```
**Reasoning:**  
This shows every connection *attempt* the scanner made.

---

## **Step 5 — Identify horizontal vs vertical scan**
### Vertical (same IP, many ports):
```
ip.src == <scanner_ip> && ip.dst == <target_ip> && tcp.flags.syn == 1
```
### Horizontal (same port, many IPs):
```
ip.src == <scanner_ip> && tcp.dstport == <port>
```

**Reasoning:**  
Pattern recognition confirms scan type.

---

## **Step 6 — Review Conversations**
**Action:**  
`Statistics → Conversations → TCP`

**Reasoning:**  
Targets that reply will show B→A packets. No-reply hosts appear with zero return packets.

---

# **6. Identifying Open, Closed, and Filtered Ports**
This is the heart of scan analysis.

## **6.1 Identify Open Ports (SYN/ACK)**
**Action:**
```
tcp.flags.syn == 1 && tcp.flags.ack == 1 && ip.dst == <scanner_ip>
```
**Reasoning:**  
A SYN/ACK means the target port is **open**.

**Interpretation Example:**
```
10.0.0.12:445 → <scanner_ip> SYN, ACK
```
= Port **445** open on **10.0.0.12**.

---

## **6.2 Identify Closed Ports (RST)**
**Action:**
```
tcp.flags.reset == 1 && ip.dst == <scanner_ip>
```
**Reasoning:**  
RST responses indicate **closed** ports.

---

## **6.3 Identify Filtered Ports (ICMP)**
**Action:**
```
icmp.type == 3
```
**Reasoning:**  
Firewalls often send ICMP unreachable messages.

ICMP code meanings:
- **Code 1:** Host unreachable
- **Code 3:** Port unreachable
- **Code 13:** Blocked by firewall

---

# **7. Useful Wireshark Filters (Cheat Sheet)**
### SYN probes from scanner
```
tcp.flags.syn == 1 && tcp.flags.ack == 0
```
### SYN/ACK responses (open ports)
```
tcp.flags.syn == 1 && tcp.flags.ack == 1
```
### RST responses (closed ports)
```
tcp.flags.reset == 1
```
### ICMP unreachable (filtered)
```
icmp.type == 3
```
### Show scan activity for specific port
```
tcp.dstport == <port>
```
### Focus on scanner only
```
ip.addr == <scanner_ip>
```

---

# **8. Evidence Collection Checklist**
A SOC analyst should capture:
- Scanner IP
- Time window of activity
- Scan type (horizontal/vertical)
- Destination IPs targeted
- Open ports discovered
- Closed ports observed
- Filtered ports/ICMP errors
- Whether scan succeeded or was blocked
- Any follow-up activity (exploitation attempts)

---

# **9. Example Investigation Flow 
1. Open PCAP in Wireshark
2. Go to **Statistics → Endpoints** to find the likely scanner
3. Apply filter:
   - `ip.addr == <scanner_ip>`
4. Show SYN probes:
   - `tcp.flags.syn == 1 && tcp.flags.ack == 0`
5. Identify responses:
   - Open ports: `tcp.flags.syn == 1 && tcp.flags.ack == 1`
   - Closed ports: `tcp.flags.reset == 1`
   - Filtered: `icmp.type == 3`
6. Document open ports per IP
7. Confirm scan type via Conversations
8. Build final findings summary

---

# **10. Analyst Documentation Template**
Use this for incident notes.

```
Incident Title: Network Scan Identified in PCAP
Date/Time:
Analyst:
Case ID:

1. Scanner Information
- Suspected source IP:
- Time window:
- Scan type (horizontal/vertical/ICMP):

2. Target Summary
- Total targets scanned:
- Open ports found:
- Closed ports observed:
- Filtered/IP unreachable:

3. Key Evidence
- SYN → SYN/ACK packet examples:
- SYN → RST examples:
- ICMP unreachable examples:

4. Classification
- FP / Benign / Suspicious / Confirmed Malicious
- Rationale:

5. Recommended Actions
- Network containment (if needed)
- Monitoring steps
- Further investigation

6. Lessons Learned
```

---

## **Screenshot Placeholders **
Below are suggested screenshot placeholders you can capture and replace in your local PDF version.

### **1. Endpoints View (Identifying Top Talkers)**
![Wireshark Endpoints Screenshot Placeholder](path/to/your/endpoints_screenshot.png)

### **2. SYN Scan Traffic Filtered**
![Wireshark SYN Filter Screenshot Placeholder](path/to/your/syn_filter_screenshot.png)

### **3. SYN/ACK Responses (Open Ports)**
![Wireshark SYN-ACK Screenshot Placeholder](path/to/your/synack_screenshot.png)

### **4. TCP Reset Responses (Closed Ports)**
![Wireshark RST Screenshot Placeholder](path/to/your/rst_screenshot.png)

### **5. ICMP Unreachable Messages (Filtered Ports)**
![Wireshark ICMP Screenshot Placeholder](path/to/your/icmp_screenshot.png)

### **6. Conversations Window**
![Wireshark Conversations Screenshot Placeholder](path/to/your/conversations_screenshot.png)

### **7. Follow TCP Stream Example**
![Wireshark Follow Stream Screenshot Placeholder](path/to/your/follow_stream_screenshot.png)

---

**End of Guide**

You can save this document as PDF using your browser or system export functions.

