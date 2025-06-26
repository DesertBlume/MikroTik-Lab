## MikroTik OSPF Lab Setup Guide

Author: Hmoad Hajali
Last Updated: June 25th 2025

---

### 📜 MikroTik Command Reference

| Command                                                                              | Description                                             |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------- |
| `/interface print`                                                                   | Lists all interfaces and their current status           |
| `/interface set [find default-name=NAME] name=NEWNAME`                               | Renames an interface                                    |
| `/ip address add address=X.X.X.X/24 interface=NAME`                                  | Assigns an IP address to an interface                   |
| `/ip address print`                                                                  | Lists all IP addresses on the router                    |
| `/ip address remove numbers=X`                                                       | Removes an IP address by its ID number                  |
| `/interface bridge add name=NAME`                                                    | Creates a new loopback-style interface (as a bridge)    |
| `/routing ospf instance add name=NAME router-id=X.X.X.X`                             | Creates a new OSPF process with a router ID             |
| `/routing ospf area add name=NAME area-id=X.X.X.X instance=NAME`                     | Adds a new OSPF area to a process                       |
| `/routing ospf interface-template add interfaces=NAME networks=X.X.X.X/YY area=NAME` | Enables OSPF on an interface and advertises its network |
| `/routing ospf neighbor print`                                                       | Shows OSPF neighbors and adjacency states               |
| `ping X.X.X.X`                                                                       | Tests connectivity to another host or router            |

---

### Lab Overview

This lab guides you through setting up a Cisco router and two MikroTik CHR routers (T1 and T2) for an OSPF routing scenario in a virtualized environment using VMware Workstation.

---

## Part 1: Cisco Router Configuration (R1)

```plaintext
hostname username-R1
enable secret class
no ip domain-lookup
service password-encryption
no ip tftp source-interface
banner motd #Authorized Users Only! Definitely No One Named Rich!#

! OSPF Configuration
router ospf U
 router-id U.0.0.U

! Console Line Settings
line con 0
 logging synchronous
 exec-timeout 0 0

! VTY Line Settings
line vty 0 15
 logging synchronous
 login local
 transport input telnet ssh

! Interfaces
interface range g0/0/0 - 1
 no shutdown
 ip ospf U area 0
 ip ospf priority U

interface g0/0/0
 ip address 10.U.11.U 255.255.255.0
 description Default Gateway

interface g0/0/1
 ip address 10.U.10.U 255.255.255.0
 description Link to T1 and T2 through T1

interface g0/0/2
 ip address 203.0.113.U 255.255.255.0
 description Link to Remote

! User and SSH Setup
username cisco privilege 15 secret cisco
ip domain-name cnap.cst
ip ssh version 2
crypto key generate rsa modulus 1024
```

---

## Part 2: MikroTik CHR Router Preparation

### VMware Network Settings

1. **VMnet0** → Bridged (Realtek/Black Port – physically connected to router)
2. **VMnet1** → Host-only

   * Go to **Edit > Virtual Network Editor**
   * Click **Change Settings**
   * Add **VMnet1** and **Uncheck DHCP Server**

### Replace VMX File

Copy the provided `.vmx` file to:

```
C:\Users\Student\Documents\Virtual Machines\chr-7.19.1.vmdk
```

Replace the original MikroTik VM’s config file. Start the VM and confirm it boots.

* **Username**: `admin`
* **Password**: `admin`

---

## Part 3: Clone MikroTik CHR Routers

### Clone CHR

* Clone the working CHR VM twice

  * First clone: Name it `T1`
  * Second clone: Name it `T2`
* Ensure you choose **Full Clone**, **not Linked Clone**

---

## Part 4: Configure T1 and T2

### T1

* Add two **custom adapters**:

  * Adapter 1: **VMnet0** (Bridged)
  * Adapter 2: **VMnet1** (Host-only)

#### Interface Setup

```shell
/interface print
/interface set [find default-name=ether1] name=[new-name]
/ip address add address=X.X.X.X/24 interface=[interface-name]
```

To **remove** an IP:

```shell
/ip address print
/ip address remove numbers=X
```

### T2

* Add **one adapter only**:

  * Adapter: **VMnet1** (Host-only)

#### Interface Setup

```shell
/interface bridge add name=loopback
/ip address add address=10.U.22.2/24 interface=loopback
```

### ✅ Ping Test

From T1:

```shell
ping [T2 IP on VMnet1]
ping [R1 IP directly connected]
```

> If both pings work: You are ready to configure OSPF.
> If not: **Do not proceed. Fix this first.**

---

## Part 5: Configure OSPF on MikroTik

### T1 Configuration:

```shell
/routing ospf instance add name=ospf1 router-id=U.0.0.1
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ospf1

/routing ospf interface-template add interfaces=[interface-name] networks=10.U.10.0/24 area=backbone
/routing ospf interface-template add interfaces=[interface-name] networks=10.U.12.0/24 area=backbone
```

> Repeat similar configuration on T2 with its own router ID and correct interface/network.

---

## Part 6: Verify OSPF

### On Cisco:

```bash
show ip ospf neighbor
```

### On MikroTik:

```shell
/routing ospf neighbor print
```

> You should see neighbors in `Full` state.

---

## You're Done!

If OSPF neighbor relationships are formed and pings work, your lab is successful.
