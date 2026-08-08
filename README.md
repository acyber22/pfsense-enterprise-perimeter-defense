# pfsense-enterprise-perimeter-defense

# Enterprise Perimeter Defense: pfSense ACLs & Suricata IDS/IPS

##  Project Overview
This project simulates an enterprise perimeter defense architecture using a **pfSense firewall gateway** to control inter-zone communication between a **Linux Mint analyst workstation** and an **Ubuntu server**. The lab demonstrates hands-on implementation of granular Access Control Lists (ACLs), stateful inspection, and traffic analysis

##  Lab Network Topology
The environment utilizes a split network architecture separated by a pfSense 2.8.1-RELEASE firewall, featuring distinct LAN and DMZ/Server zones[cite: 1].

* **WAN Interface (`em0`):** `10.0.2.15/24` (Bridged/NAT to Internet)
* **LAN Zone (`em1` / `in0`):** `192.168.10.1/24` — Hosting the Linux Mint Client VM (`192.168.10.x`)
* **DMZ/Server Zone (`em2` / `in1`):** `192.168.20.1/24` — Hosting the Ubuntu Server 22.04 LTS VM (`192.168.20.x`)
