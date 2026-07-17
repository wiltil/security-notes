# 🛡️ TShark – CLI Network Protocol Analyzer

## 📌 Overview
- TShark is the **command-line version of Wireshark**.
- Used for **packet capture and analysis** directly in terminal.
- Lightweight, scriptable, and ideal for **automation, remote forensics, and headless servers**.
- Supports capture filters (BPF syntax) and display filters (Wireshark syntax).
---

## ⚙️ Basic Usage
```bash
# Capture live traffic on interface eth0 [-i]
tshark -i eth0

# Read from a saved pcap file [-r]
tshark -r capture.pcap

# Limit number of packets [-c]
tshark -c 50 -i eth0

# Save capture to file [-w]
tshark -i eth0 -w output.pcap
```
---

## Level 2 usage :

**Capture Filters (BPF syntax, applied during capture)[-f]**
```bash  
tshark -i eth0 -f "tcp port 80"   # capture only HTTP traffic
tshark -i eth0 -f "icmp"          # capture only ping packets
```

**Display Filters (Wireshark syntax, applied after capture)** [-Y]
```bash 
tshark -r capture.pcap -Y "http"                        # show only HTTP packets
tshark -r capture.pcap -Y "ip.src==192.168.1.10"        # filter by source IP
tshark -r capture.pcap -Y "tcp.flags.syn==1 && tcp.flags.ack==0"  # detect SYN scans
```

**Useful Options**
```bash
# -T fields -e fieldname → extract specific fields
tshark -r capture.pcap -T fields -e ip.src -e ip.dst
```

**Example Scenarios**
1. Detect suspicious ping exfiltration
```bash
tshark -i eth0 -Y "icmp && data.len > 0"
```

2.Extract credentials from HTTP traffic:
```bash
tshark -r web.pcap -Y "http.request" -T fields -e http.host -e http.user_agent
```

3.Monitor DNS queries in real time:
```bash
tshark -i eth0 -Y "dns"
```
