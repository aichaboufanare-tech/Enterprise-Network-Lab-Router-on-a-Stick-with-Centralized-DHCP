

# Enterprise Network Lab – Router-on-a-Stick with Centralized DHCP

## 📌 Project Overview

This project simulates a small enterprise network using **GNS3** and Cisco IOU devices.
It demonstrates VLAN segmentation, Inter-VLAN routing (Router-on-a-Stick), and centralized DHCP deployment using an Ubuntu server.

The lab is designed for CCNA–CCNP level practice.

---

## 🏗️ Network Architecture

The topology includes:

* Core Router (IOU2)
* Access Switch (IOU1)
* Ubuntu DHCP Server
* End Devices (VPCS)

### VLAN Structure

| VLAN           | Network          | Gateway       |
| -------------- | ---------------- | ------------- |
| VLAN 10        | 192.168.10.0/24  | 192.168.10.1  |
| VLAN 20        | 192.168.20.0/24  | 192.168.20.1  |
| Server Network | 192.168.100.0/24 | 192.168.100.1 |

DHCP Server IP: **192.168.100.10**

---

## 🔥 Key Technologies Used

* Router-on-a-Stick (802.1Q Subinterfaces)
* VLAN Segmentation
* Trunk Links
* Inter-VLAN Routing
* DHCP Relay (`ip helper-address`)
* Ubuntu ISC DHCP Server
* Static Server IP Configuration

---

## ⚙️ Router Configuration (IOU2)

### Server Network Interface

```
interface Ethernet0/0
 ip address 192.168.100.1 255.255.255.0
```

### VLAN 10 Subinterface

```
interface Ethernet0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.100.10
```

### VLAN 20 Subinterface

```
interface Ethernet0/2.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.100.10
```

---

## 🖥️ Switch Configuration (IOU1)

### VLAN Creation

```
vlan 10
 name USERS_VLAN10

vlan 20
 name USERS_VLAN20
```

### Access Ports

```
interface range e0/0-1
 switchport mode access
 switchport access vlan 10

interface range e0/2-3
 switchport mode access
 switchport access vlan 20
```

### Trunk Ports (Connected to Router)

```
interface e0/4
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

---

## 🖥️ DHCP Server Configuration (Ubuntu)

### Install DHCP Server

```
sudo apt update
sudo apt install isc-dhcp-server -y
```

### Assign Static IP (Netplan)

```
addresses:
 - 192.168.100.10/24
gateway4: 192.168.100.1
```

### DHCP Pools

```
authoritative;

subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.100 192.168.10.200;
  option routers 192.168.10.1;
}

subnet 192.168.20.0 netmask 255.255.255.0 {
  range 192.168.20.100 192.168.20.200;
  option routers 192.168.20.1;
}
```

### Disable Firewall (Lab Only)

```
sudo ufw disable
```

### Start Service

```
sudo systemctl restart isc-dhcp-server
```

---

## 🔎 How DHCP Works in This Lab

1. Client sends DHCP Discover (broadcast).
2. Router receives it on VLAN subinterface.
3. `ip helper-address` forwards request to DHCP server.
4. Server responds with Offer.
5. Client receives IP configuration.

---

## 🧪 Verification Commands

### On Router

```
show ip interface brief
show ip route
show run | include helper
```

### On Switch

```
show vlan brief
show interfaces trunk
```

### On Client (VPCS)

```
ip dhcp
show ip
```

---

## 🎯 Learning Objectives

This lab helps you understand:

* VLAN isolation and segmentation
* 802.1Q trunking
* Router-on-a-Stick design
* DHCP Relay operation
* Centralized DHCP architecture
* Enterprise Layer 3 routing basics

