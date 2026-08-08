
# Enterprise Perimeter Defense: pfSense ACLs & Suricata IDS/IPS

##  Project Overview
This project simulates an enterprise perimeter defense architecture using a **pfSense firewall gateway** to control inter-zone communication between a **Linux Mint analyst workstation** and an **Ubuntu server**. The lab demonstrates hands-on implementation of granular Access Control Lists (ACLs), stateful inspection, and traffic analysis

##  Lab Network Topology
The environment utilizes a split network architecture separated by a pfSense 2.8.1-RELEASE firewall, featuring distinct LAN and DMZ/Server zones[cite: 1].

* **WAN Interface (`em0`):** `10.0.2.15/24` (Bridged/NAT to Internet)
* **LAN Zone (`em1` / `in0`):** `192.168.10.1/24` — Hosting the Linux Mint Client VM (`192.168.10.x`)
* **DMZ/Server Zone (`em2` / `in1`):** `192.168.20.1/24` — Hosting the Ubuntu Server 22.04 LTS VM (`192.168.20.x`)

 ![image alt](https://github.com/acyber22/pfsense-enterprise-perimeter-defense/blob/c5022bc362e84eb87309cb37e0ecdb40619e3fdd/Lab%20Network%20Topology.png)

 ## ⚙️ Phase 1: pfSense Firewall & ACL Configuration

Before building rulesets, IPv4 was strictly enforced by disabling IPv6 under **System > Advanced > Networking** and stripping IPv6 rules from the WAN and LAN interfaces

### Key Firewall Design Principles Applied:
* **Top-Down Processing:** Rules are evaluated sequentially; frequently matched rules are placed higher to optimize system load
* **Stateful Inspection:** The firewall state table tracks source, destination, protocol, ports, and connection states
* **Block vs. Reject:** 
  * *Block:* Silently drops packets (ideal for WAN interfaces)
  * *Reject:* Drops packets and returns an unreachable reply (ideal for LAN interfaces)

### Traffic Validation & Testing (Port 1337)
To verify policy enforcement, a Netcat listener was established on the Ubuntu server (`192.168.20.10:1337`), and a Telnet probe was launched from the Linux Mint workstation.

1. **Initial Baseline Verification (PCAP Analysis):** Packet captures on the `em2` interface confirmed standard 3-way handshakes with a baseline TCP length of 0, expanding to length 15 once raw application data payloads transmitted.
2. **Policy Enforcement:** To restrict unauthorized application traffic, an explicit rule was inserted near the top of the LAN ruleset to reject traffic targeting port 1337.
3. **Result:** Subsequent Telnet attempts returned a `connection refused` error, successfully validated by pfSense log entries.
