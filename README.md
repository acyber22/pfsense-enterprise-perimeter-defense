
# Enterprise Perimeter Defense: pfSense ACLs & Suricata IDS/IPS

##  Project Overview
This project simulates an enterprise perimeter defense architecture using a **pfSense firewall gateway** to control inter-zone communication between a **Linux Mint analyst workstation** and an **Ubuntu server**. The lab demonstrates hands-on implementation of granular Access Control Lists (ACLs), stateful inspection, and traffic analysis

##  Lab Network Topology
The environment utilizes a split network architecture separated by a pfSense 2.8.1-RELEASE firewall, featuring distinct LAN and DMZ/Server zones.

* **WAN Interface (`em0`):** `10.0.2.15/24` (Bridged/NAT to Internet)
* **LAN Zone (`em1` / `in0`):** `192.168.10.1/24` — Hosting the Linux Mint Client VM (`192.168.10.x`)
* **DMZ/Server Zone (`em2` / `in1`):** `192.168.20.1/24` — Hosting the Ubuntu Server 22.04 LTS VM (`192.168.20.x`)

 ![image alt](https://github.com/acyber22/pfsense-enterprise-perimeter-defense/blob/c5022bc362e84eb87309cb37e0ecdb40619e3fdd/Lab%20Network%20Topology.png)

Figure 1: Network Topology Diagram

 ## Phase 1: pfSense Firewall & ACL Configuration

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

![image alt](https://github.com/acyber22/pfsense-enterprise-perimeter-defense/blob/5a837cf7afc50089f95a3b4106d06faa3c452b8f/Traffic%20Analysis%20and%20Protcol%20Verification.png)

 Figure 2: Traffic Analysis and Protocol Verification

 ![image alt](https://github.com/acyber22/pfsense-enterprise-perimeter-defense/blob/main/Active%20Client%20Traffic%20Generation.png)

 Figure 3: Active Client Traffic Generation

Since port 1337 was something I wanted to reject I went ahead and made a rule inside of the LAN interface. This rule was placed up higher in the ACL since it needed to be executed before others. After the creation I was no longer able to use telnet and was now receiving telnet: unable to connect to remote host connection refused. Inside of the Firewall Log Entries I was able to see that the rule rejected it.

![image alt](https://github.com/acyber22/pfsense-enterprise-perimeter-defense/blob/main/Policy-Enforced%20TCP%20Reject%20on%20Port%201337.png)

Figure 4: pfSense Firewall Log Entry Confirming a Policy-Enforced TCP Reject on Port 1337
 
## Phase 2: Least-Privilege Hardening

To transition away from a default-allow posture, the LAN interface was hardened using explicit egress filtering and destination-oriented aliases:

* **`LAN_TCP_Outbound_Whitelist`:** Ports `22`, `53`, `80`, `443`
* **`LAN_UDP_Outbound_Whitelist`:** Ports `53`, `123`
* **Implicit Deny:** A final rule dropping all non-permitted outbound traffic from LAN subnets

![image alt](https://github.com/acyber22/pfsense-enterprise-perimeter-defense/blob/main/Firewall%20LAN%20Interface.png)

Figure 5: Hardened LAN Ruleset Matrix with TCP And UDP Alias Details



## 🛡️ Phase 3: NIDS Implementation (Suricata)
Following core firewall configurations, the **Suricata** package was deployed on pfSense to supply network-based intrusion detection and prevention capabilities

