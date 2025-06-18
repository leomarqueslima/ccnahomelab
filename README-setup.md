# Practical CCNA Home Lab Setup: Internet Access via Cisco Devices

Basic configuration of a homelab to get internet connectivty at home using Cisco equipment. The goal of this repository is to document my journey in applying the knowledge gained during my studies for the CCNA exam in a SIMPLE, but real lab. I faced several challenges in setting up this basic topology. This showed me that knowing the theory and doing packet tracer labs, although essential, are not enough. For example, I used a Cisco Router 819. It only has one Layer 3 routable port - GigabitEthernet 0. This is the port connected to the modem. Therefore, Subinterfaces ROA (Router on a stick) connection to the switches would not work for this configuration. SVIs were the solution. Also, CCNA only deals with APs configurations via an WLC GUI interface. Autonomous AP configuration are not covered in the CCNA material. So, I had to learn how to configre an Autonomous AP. In the next days, I will add hosts to the switches, add vlans, access-lists, and IP phones and QOS. Stay tunned!

## Objective

Set up a working home network using Cisco equipment, demonstrating practical CCNA knowledge. Provide internet access to wireless clients through VLAN segmentation, DHCP, and NAT, while troubleshooting real-world issues.

---

## 1. **Topology Overview**
![image alt](https://github.com/leomarqueslima/ccnahomelab/blob/93534e2dde9139dffd42ef3880cac8737ed5fb00/Topology%20photo.jpg)
![image alt](https://github.com/leomarqueslima/ccnahomelab/blob/743ba55e36a51e2322a847b01aeee0bb51101c04/topology.png)
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

> **Issue Encountered:** Command rejected: Bad VLAN allowed list. Required all default VLANs. This took me a while to figure it out. It is because these ports of the router behave like switchports. So, if you want to use trunk ports you have to include all vlans. **Fix:** Included `1,30,1002-1005` in trunk allowed VLANs.

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
> 
- Confirmed radio was up with `no shutdown`
- Configured encryption for vlan 30
- Ensured `mbssid` and `guest-mode` enabled - I think these two were the main thing why clients could not see the SSID

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
- Interfacing router-on-a-stick model using non-routable switchports and SVIs
- DHCP and NAT configuration
- Troubleshooting VLAN and SSID issues
- Use of BVI interfaces in autonomous APs
- Understanding dot1Q tagging and subinterfaces

This project used many of the CCNA fundamentals in a real-world home lab scenario. It is still small and it will be further expanded.

---



## 7. **Conclusion**

This guide provides a solid foundation for using CCNA knowledge in a home environment. By manually configuring devices and troubleshooting through trial and error, you build real confidence and understanding of networking concepts.

