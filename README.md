# WAN Network Project with ACLs

## Project Overview

This document describes the project for interconnecting the local area networks (LANs) of the São Paulo, Rio de Janeiro, and Vitória branches through a WAN.

The architecture was designed in a linear topology using Cisco routers and corporate switches, providing dynamic IP address assignment through DHCP and centralized name resolution through DNS.

> **Note:** The original document mentioned redundancy of essential services. This was removed here because the implemented topology does not include actual redundancy.

---

## Components and Equipment Used

The physical infrastructure simulated in Cisco Packet Tracer is composed of the following devices at each location:

### São Paulo
- 01 Cisco 2911 Router (`Router SP`)
- 01 Cisco 2950-24 Switch (`Switch2`)
- 01 Generic Access Point (`Access Point Sao`)
- 01 Local Server (`Server-PT Sao.com`)
- 01 Workstation (`PC2`)
- 01 Network Printer (`Printer0`)

### Rio de Janeiro
- 01 Cisco 2911 Router (`Router RIO`)
- 01 Cisco 2950-24 Switch (`Switch1`)
- 01 Generic Access Point (`Access Point RIO`)
- 01 Centralized Server (`Server-PT Rio.com`)
- 02 Workstations (`PC0` and `PC1`)
- 01 Wireless Laptop (`Laptop0`)

### Vitória
- 01 Cisco 2911 Router (`Router Vitoria`)
- 01 Cisco 2950-24 Switch (`Switch3`)
- 01 Generic Access Point (`Access Point Vitoria`)
- 01 Local Server (`Server-PT Vitoria.com`)
- 01 Workstation (`PC3`)
- 01 Wireless Laptop (`Laptop1`)

---

## IP Addressing Plan and Interfaces

| Device / Link | Interface | IP Address | Subnet Mask | Default Gateway | Function / Notes |
|---|---|---:|---|---|---|
| WAN Link SP-RIO | Gig0/0 (SP) | 10.0.0.1 | 255.0.0.0 (/8) | N/A | Direct communication between SP and RIO |
| WAN Link SP-RIO | Gig0/0 (RIO) | 10.0.0.2 | 255.0.0.0 (/8) | N/A | Direct communication between SP and RIO |
| WAN Link RIO-VIT | Gig0/1 (RIO) | 20.0.0.1 | 255.0.0.0 (/8) | N/A | Direct communication between RIO and Vitória |
| WAN Link RIO-VIT | Gig0/0 (VIT) | 20.0.0.2 | 255.0.0.0 (/8) | N/A | Direct communication between RIO and Vitória |
| SP Gateway | Gig0/1 | 192.168.1.1 | 255.255.255.0 (/24) | N/A | São Paulo LAN gateway |
| SP Server | Fa0 | 192.168.1.2 | 255.255.255.0 (/24) | 192.168.1.1 | Local Web Server (`sao.com`) |
| RIO Gateway | Gig0/2 | 192.168.2.1 | 255.255.255.0 (/24) | N/A | Rio de Janeiro LAN gateway |
| RIO Server | Fa0 | 192.168.2.2 | 255.255.255.0 (/24) | 192.168.2.1 | Web Server and Master DNS (`rio.com`) |
| VIT Gateway | Gig0/1 | 192.168.3.1 | 255.255.255.0 (/24) | N/A | Vitória LAN gateway |
| VIT Server | Fa0 | 192.168.3.2 | 255.255.255.0 (/24) | 192.168.3.1 | Local Web Server (`vitoria.com`) |
| SP LAN Hosts | DHCP | 192.168.1.11 - 192.168.1.254 | 255.255.255.0 (/24) | 192.168.1.1 | Dynamic IPs (DNS: 192.168.2.2) |
| RIO LAN Hosts | DHCP | 192.168.2.11 - 192.168.2.254 | 255.255.255.0 (/24) | 192.168.2.1 | Dynamic IPs (DNS: 192.168.2.2) |
| VIT LAN Hosts | DHCP | 192.168.3.11 - 192.168.3.254 | 255.255.255.0 (/24) | 192.168.3.1 | Dynamic IPs (DNS: 192.168.2.2) |

---

## Basic and Essential Service Configuration

### Dynamic Host Configuration Protocol (DHCP)

To optimize host administration, the Cisco routers at each branch were configured as local DHCP servers.

To avoid connectivity conflicts with devices using static IP addresses, the first ten addresses of each subnet were explicitly excluded from the automatic allocation pool using the `ip dhcp excluded-address` command.

Each DHCP scope automatically distributes the IP address, subnet mask, corresponding gateway, and points clients to the centralized DNS server located in the Rio de Janeiro network (`192.168.2.2`).

### Centralized Domain Name System (DNS)

The DNS service is centralized on the main server in Rio de Janeiro (`192.168.2.2`).

It statically maps fully qualified domain names (FQDNs) to their respective internal IP addresses:

- `rio.com` → `192.168.2.2`
- `sao.com` → `192.168.1.2`
- `vitoria.com` → `192.168.3.2`

---

## Wireless Access Layer

Connectivity for mobile devices (laptops) was implemented through access points operating at Layers 1 and 2 of the OSI model.

Following the project requirements, wireless authentication was configured using WEP (Wired Equivalent Privacy) with a static hexadecimal key.

> **Public repository note:** The original lab key was intentionally removed from this README.

The SSIDs were configured individually for each location. Devices connected through the wireless network receive their IP configuration directly from the DHCP service configured on the Cisco routers.

---

## Remote Management Service (Telnet VTY)

To enable centralized remote administration of the infrastructure without requiring physical access to the routers, the Telnet virtual terminal service was enabled on all routers.

The virtual terminal lines (`line vty 0 4`) were configured with password authentication. An additional password was configured to protect privileged EXEC mode.

> **Public repository note:** The original lab passwords were intentionally removed from this README.

This setup allows administrators connected through workstations or laptops on the WAN to perform diagnostics and configuration changes remotely through the command line.

---

## Security Policies and Access Control Lists (ACLs)

To reduce vulnerabilities and optimize data flow across the WAN interconnecting the three branches, access-control policies were implemented using Layer 3 packet-filtering rules.

The implemented logic separates administrative router traffic and restricts direct communication between specific local networks, handling employees, third-party users, and customers differently.

### Administrative Access Control and Remote Management

To protect network devices from unauthorized access by regular users, a clear distinction was established between internal employee privileges and the permissions granted to customers or third-party users connected through the wireless network.

A standard Access Control List (ACL) was applied to the VTY lines of all routers to restrict Telnet access.

With this policy, customers, suppliers, and third-party service providers connected through Wi-Fi are prevented from initiating remote management sessions or accessing the routers' command-line interface.

Even if these external users receive valid dynamic IP addresses through DHCP, any attempt to remotely connect to the routers is denied.

Remote administrative privileges are restricted exclusively to workstations located in the Rio de Janeiro branch subnet (`192.168.2.0/24`).

This centralizes administrative control of the WAN infrastructure with the internal technical team and reduces the risk of unauthorized monitoring or configuration changes by third parties.

---

## Inter-Branch Traffic Restriction

### Network Segmentation and Security Policy

To meet security, departmental isolation, and confidential-data protection requirements, a unidirectional security policy was implemented between Vitória and São Paulo through an extended ACL configured on the São Paulo router (`Router SP`).

The rule was designed around the scenario in which the Vitória branch receives a constant flow of third-party service providers, suppliers, and customers using its facilities and Wi-Fi network.

To prevent these external users from gaining visibility into or establishing direct connectivity with local hosts, file servers, or printers inside the São Paulo network (`192.168.1.0/24`), all traffic originating from Vitória and destined for the São Paulo LAN was blocked.

However, to support essential business operations and customer/supplier self-service, an explicit exception allows HTTP traffic.

As a result, external users and employees in Vitória can access the São Paulo institutional web server (`sao.com` - `192.168.1.2`) to consult catalogs and approved corporate systems without exposing São Paulo's private local network.

---

## Dynamic Routing Protocol (RIPv2)

To provide end-to-end communication between geographically separated networks and allow computers to access servers located in other branches, the RIPv2 dynamic routing protocol was implemented.

The configuration uses the `no auto-summary` command, enabling support for classless networks (VLSM) and ensuring correct routing-table behavior between the WAN `/8` networks (`10.0.0.0` and `20.0.0.0`) and the `/24` LANs (`192.168.X.0`).

With this configuration, routing convergence occurs automatically, allowing hosts throughout the topology to make HTTP requests using domain names and communicate across all three locations.

---

## Project File

The Cisco Packet Tracer `.pkt` file included in this repository contains the complete simulated topology and configuration used in this project.

---

## Author

**Caio Folena**  
Computer Engineering Student
