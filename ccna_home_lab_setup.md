# Practical CCNA Home Lab Setup: Internet Access via Cisco Devices

This document details the step-by-step configuration of a home lab using Cisco devices, connecting a Cisco 819 router directly to a home modem for internet access, a Catalyst 3650 switch, and a Cisco AIR-CAP1702I-B-K9 access point. It includes the configurations, common errors encountered, and the solutions applied.

## Objective

Set up a working home network using Cisco equipment, demonstrating practical CCNA knowledge. Provide internet access to wireless clients through VLAN segmentation, DHCP, and NAT, while troubleshooting real-world issues.

---

## 1. **Topology Overview**

```
[Home Modem]
    |
[Router: Cisco 819] - Fa1 (switchport) ↔ G1/0/5 [Switch: Catalyst 3650] ↔ G1/0/3 [Access Point: AIR-CAP1702I-B-K9]
```

- VLAN 30 is used for wireless clients.
- The AP broadcasts SSID `WeWorshipThor`.
- The router provides DHCP and NAT for internet access.

---

## 2. **Router (Cisco 819) Configuration**

### Internet Connection

```bash
interface GigabitEthernet0
 description Connected to Modem
 ip address dhcp
 ip nat outside
 no shutdown
```

### Internal LAN Setup (VLAN 30)

```bash
interface Vlan30
 ip address 192.168.30.1 255.255.255.0
 ip nat inside
 no shutdown

ip dhcp pool VLAN30-POOL
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
```

### NAT Configuration

```bash
ip access-list standard NAT-LAN
 permit 192.168.30.0 0.0.0.255

ip nat inside source list NAT-LAN interface GigabitEthernet0 overload
```

### Fa1 as Trunk Port (Switchport)

```bash
interface FastEthernet1
 switchport mode trunk
 switchport trunk allowed vlan 1,30,1002-1005
```

> **Issue Encountered:** Command rejected: Bad VLAN allowed list. Required all default VLANs. **Fix:** Included `1,30,1002-1005` in trunk allowed VLANs.

---

## 3. **Switch (Catalyst 3650) Configuration**

### VLAN Creation

```bash
vlan 30
 name WiFi
```

### Interface Configurations

```bash
interface GigabitEthernet1/0/5
 description Uplink to Router
 switchport mode trunk
 switchport trunk allowed vlan 1,30,1002-1005

interface GigabitEthernet1/0/3
 description Access Point
 switchport access vlan 30
 switchport mode access
 spanning-tree portfast
```

---

## 4. **Access Point (AIR-CAP1702I-B-K9) Configuration**

> Note: This AP runs autonomous IOS (k9w7 image)

### Interface and SSID Setup

```bash
interface Dot11Radio0
 encryption mode ciphers aes-ccm
 encryption vlan 30 mode ciphers aes-ccm
 ssid WeWorshipThor
 no shutdown
 station-role root

interface GigabitEthernet0.30
 encapsulation dot1Q 30
 ip address dhcp
 bridge-group 30
 bridge-group 30 spanning-disabled
 no bridge-group 30 source-learning

interface Dot11Radio0.30
 encapsulation dot1Q 30
 bridge-group 30

interface BVI30
 ip address dhcp
```

### SSID Configuration

```bash
dot11 ssid WeWorshipThor
 vlan 30
 authentication open
 authentication key-management wpa
 wpa-psk ascii SuperSecure123
 guest-mode
```

> **Issue Encountered:** SSID `WeWorshipThor` not visible **Fixes Applied:**

- Confirmed radio was up with `no shutdown`
- Configured encryption for vlan 30
- Ensured `mbssid` and `guest-mode` enabled

> **Issue Encountered:** Clients stuck on connecting **Fix:** Set encryption to `aes-ccm` on Dot11Radio0 and vlan 30 explicitly

> **Issue Encountered:** No IP assigned to interface BVI30 **Fix:** Verified DHCP was working and switchport was correctly assigned to vlan 30

---

## 5. **Final Verifications**

- `show ip interface brief` on AP showed BVI30 with DHCP-acquired IP.
- Router could ping AP and vice versa.
- Mobile device could associate to `WeWorshipThor` and get IP address.
- Internet access confirmed.

---

## 6. **Lessons Learned and Practical CCNA Applications**

- VLAN trunking with switchports and subinterfaces
- Interfacing router-on-a-stick model using non-routable switchports
- DHCP and NAT configuration
- Troubleshooting VLAN and SSID issues
- Use of BVI interfaces in autonomous APs
- Understanding dot1Q tagging and subinterfaces

This project is a full application of CCNA fundamentals in a real-world home lab scenario.

---



## 7. **Conclusion**

This guide provides a solid foundation for using CCNA knowledge in a home environment. By manually configuring devices and troubleshooting through trial and error, you build real confidence and understanding of networking concepts.

