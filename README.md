# Secure Multi-Site WAN Network with ACLs

## Project Overview

This project was developed in **Cisco Packet Tracer** as an academic network simulation focused on WAN connectivity, routing, network services, wireless access, and access-control policies.

The topology interconnects the local area networks (LANs) of three branches located in **São Paulo, Rio de Janeiro, and Vitória** through a WAN.

The architecture uses Cisco routers, switches, access points, local servers, DHCP services, centralized DNS, RIPv2 dynamic routing, and Access Control Lists (ACLs).

---

## Technologies and Concepts Used

- Cisco Packet Tracer
- Cisco 2911 Routers
- Cisco 2950-24 Switches
- WAN and LAN networking
- Wireless LAN (WLAN)
- DHCP
- DNS
- RIPv2
- Standard ACLs
- Extended ACLs
- Telnet / VTY remote administration
- Layer 3 traffic filtering
- Network segmentation
- HTTP access control

---

## Components and Equipment Used

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

## IP Addressing Plan

| Device / Link | Interface | IP Address | Subnet Mask | Default Gateway | Function |
|---|---|---:|---|---|---|
| WAN SP-RIO | Gig0/0 (SP) | 10.0.0.1 | 255.0.0.0 (/8) | N/A | SP-RIO WAN link |
| WAN SP-RIO | Gig0/0 (RIO) | 10.0.0.2 | 255.0.0.0 (/8) | N/A | SP-RIO WAN link |
| WAN RIO-VIT | Gig0/1 (RIO) | 20.0.0.1 | 255.0.0.0 (/8) | N/A | RIO-VIT WAN link |
| WAN RIO-VIT | Gig0/0 (VIT) | 20.0.0.2 | 255.0.0.0 (/8) | N/A | RIO-VIT WAN link |
| SP Gateway | Gig0/1 | 192.168.1.1 | 255.255.255.0 (/24) | N/A | São Paulo LAN gateway |
| SP Server | Fa0 | 192.168.1.2 | 255.255.255.0 (/24) | 192.168.1.1 | Web server (`sao.com`) |
| RIO Gateway | Gig0/2 | 192.168.2.1 | 255.255.255.0 (/24) | N/A | Rio de Janeiro LAN gateway |
| RIO Server | Fa0 | 192.168.2.2 | 255.255.255.0 (/24) | 192.168.2.1 | Web server and master DNS (`rio.com`) |
| VIT Gateway | Gig0/1 | 192.168.3.1 | 255.255.255.0 (/24) | N/A | Vitória LAN gateway |
| VIT Server | Fa0 | 192.168.3.2 | 255.255.255.0 (/24) | 192.168.3.1 | Web server (`vitoria.com`) |

### DHCP Pools

- São Paulo: `192.168.1.11` - `192.168.1.254`
- Rio de Janeiro: `192.168.2.11` - `192.168.2.254`
- Vitória: `192.168.3.11` - `192.168.3.254`

All DHCP clients use the centralized DNS server:

`192.168.2.2`

---

## DHCP Configuration

Each branch router acts as a local DHCP server.

The first ten addresses of each subnet are excluded from the DHCP pool using:

```text
ip dhcp excluded-address
```

Each DHCP scope automatically provides:

- IP address
- Subnet mask
- Default gateway
- DNS server

The centralized DNS server is located in Rio de Janeiro at:

```text
192.168.2.2
```

---

## Centralized DNS

The DNS service is hosted on the Rio de Janeiro server.

The following internal domain names are configured:

| Domain | IP Address |
|---|---:|
| `rio.com` | `192.168.2.2` |
| `sao.com` | `192.168.1.2` |
| `vitoria.com` | `192.168.3.2` |

This allows hosts from different branches to access the web servers using domain names instead of IP addresses.

---

## Wireless Network

Each branch has its own wireless access point.

The SSIDs are configured separately by location, and wireless devices receive their IP configuration through DHCP.

The wireless security configuration uses **WEP**, following the original academic project requirements.

> This project is a simulated academic environment. WEP is a legacy protocol and should not be used in modern production networks.

---

## Remote Management with Telnet

Remote administration was configured using **Telnet over VTY lines** on the Cisco routers.

The following configuration concept is used:

```text
line vty 0 4
password cisco
login
```

Privileged EXEC mode is also protected with:

```text
enable password cisco
```

### Simulation Credentials

These credentials are used **only inside this Cisco Packet Tracer lab**.

| Purpose | Password |
|---|---|
| Telnet / VTY access | `cisco` |
| Privileged EXEC (`enable`) | `cisco` |

No real-world credentials are stored in this repository.

---

## How to Test Telnet Access

Because the ACL configuration restricts router administration to the **Rio de Janeiro subnet (`192.168.2.0/24`)**, Telnet tests should be performed from an authorized workstation located in the Rio de Janeiro LAN.

For example, from the Command Prompt of an authorized Rio workstation:

```text
telnet 192.168.1.1
```

Then enter:

```text
Password: cisco
```

To access privileged EXEC mode:

```text
enable
```

Then enter:

```text
Password: cisco
```

The same process can be used with the other router gateway addresses:

```text
192.168.2.1
192.168.3.1
```

---

## Access Control Lists (ACLs)

The project uses ACLs to implement security policies across the WAN.

The ACL logic has two main objectives:

1. Restrict remote administrative access to network devices.
2. Control traffic between specific branch networks.

---

## Administrative Access Control

A standard ACL is applied to the VTY lines of the routers.

The purpose is to prevent customers, suppliers, and third-party users connected through Wi-Fi from accessing the routers remotely.

Only devices located in the **Rio de Janeiro subnet (`192.168.2.0/24`)** are authorized to perform remote router administration.

This centralizes network administration and reduces the risk of unauthorized configuration changes.

---

## Inter-Branch Traffic Restriction

An extended ACL is configured on the São Paulo router.

The policy restricts traffic originating from the Vitória network (`192.168.3.0/24`) and destined for the São Paulo network (`192.168.1.0/24`).

General direct access is blocked.

However, HTTP traffic is explicitly allowed so that users in Vitória can access the São Paulo institutional web server:

```text
sao.com
192.168.1.2
```

This allows access to approved web services without exposing the full São Paulo local network.

---

## Dynamic Routing with RIPv2

RIPv2 is used to provide dynamic routing between the three branch networks.

The configuration uses:

```text
no auto-summary
```

This allows proper operation with classless networks and supports communication between:

### WAN Networks

```text
10.0.0.0/8
20.0.0.0/8
```

### LAN Networks

```text
192.168.1.0/24
192.168.2.0/24
192.168.3.0/24
```

With RIPv2 enabled, routes are dynamically exchanged between the routers, allowing hosts from different branches to communicate and access web services using DNS names.

---

## Project File

The `.pkt` file included in this repository contains the complete network simulation created in **Cisco Packet Tracer**.

To test the project:

1. Install or open Cisco Packet Tracer.
2. Open the `.pkt` file from this repository.
3. Allow the network to converge.
4. Test communication between hosts using `ping`.
5. Test DNS resolution and HTTP access.
6. Test Telnet access from an authorized Rio de Janeiro workstation.
7. Test the ACL restrictions from other devices and networks.

---

## Security Note

This is an **academic simulation environment**.

Some technologies used in the project, such as **Telnet and WEP**, are legacy protocols and are not recommended for modern production environments.

They were used according to the original project requirements and for educational purposes.

---

## Author

**Caio Folena**  
Computer Engineering Student
