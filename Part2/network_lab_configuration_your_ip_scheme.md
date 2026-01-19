# Network Lab Configuration (Your IP Scheme)

---

## Instructions – Basic Connectivity

ต้องสามารถทดสอบได้ครบตามเงื่อนไขต่อไปนี้

1. PC-1 (ubuntu1) สามารถ Ping IPv4 Switch ได้
2. PC-1 (ubuntu1) สามารถ Ping IPv4 PC-2 (ubuntu2) ได้
3. PC-1 (ubuntu1) สามารถ Ping IPv4 Router1 (E0/0) ได้
4. PC-2 (ubuntu2) สามารถ Ping IPv4 Switch ได้ *(ไม่จำเป็น หาก Switch มี SVI เฉพาะ VLAN A)*
5. PC-1 (ubuntu1) สามารถ Ping IPv4 1.1.1.1 ได้
6. PC-2 (ubuntu2) สามารถ Ping IPv4 1.1.1.1 ได้

> **หมายเหตุ**: ตั้งค่า Hostname ใน GNS3 ให้ตรงกับชื่ออุปกรณ์ (router1, router2, switch, ubuntu1, ubuntu2)

---

## IP Addressing Summary

| Device | Interface | IP Address | Subnet |
|------|----------|-----------|--------|
| Router1 | e0/0 | 192.168.1.1 | /24 |
| Router2 | e0/1 | 192.168.1.254 | /24 |
| Router2 | e0/0.153 | 172.61.153.129 | /27 |
| Router2 | e0/0.106 | 172.61.106.193 | /26 |
| Switch | VLAN 153 | 172.61.153.158 | /27 |
| ubuntu1 | eth0 | 172.61.153.130 | /27 |
| ubuntu2 | eth0 | DHCP (172.61.106.197–202) | /26 |

---

## ubuntu1 Configuration (Static IP)

```bash
# /etc/network/interfaces

auto eth0
iface eth0 inet static
    address 172.61.153.130
    netmask 255.255.255.224
    gateway 172.61.153.129
    dns-nameservers 192.168.1.1
```

---

## ubuntu2 Configuration (DHCP)

```bash
# /etc/network/interfaces

auto eth0
iface eth0 inet dhcp
```

> ubuntu2 จะได้รับ
> - IP: 172.61.106.197–202
> - Gateway: 172.61.106.193
> - DNS: 192.168.1.1

---

## Switch Configuration

### Create VLANs & SVI

```bash
conf t
vlan 153
vlan 106

interface vlan 153
 ip address 172.61.153.158 255.255.255.224
 no shutdown

ip default-gateway 172.61.153.129
no ip routing
do wr
```

### Interface Mode Selection

```bash
conf t
interface e0/1      # ubuntu1
 switchport mode access
 switchport access vlan 153
 exit

interface e0/2      # ubuntu2
 switchport mode access
 switchport access vlan 106
 exit

interface e0/0      # to Router2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 exit

do wr
```

---

## Router1 Configuration (DHCP + NAT)

### Interface IP

```bash
conf t
interface e0/1
 no shutdown
 ip address dhcp

interface e0/0
 no shutdown
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 exit

do wr
```

### Routing

```bash
ip route 172.61.153.128 255.255.255.224 192.168.1.254
ip route 172.61.106.192 255.255.255.192 192.168.1.254
```

### PAT (NAT Overload)

```bash
access-list 1 permit any
interface e0/1
 ip nat outside
exit

ip nat inside source list 1 interface e0/1 overload
do wr
```

### DHCP Configuration (VLAN 106)

```bash
ip dhcp excluded-address 172.61.106.193 172.61.106.196
ip dhcp excluded-address 172.61.106.203 172.61.106.254

ip dhcp pool VLAN106
 network 172.61.106.192 255.255.255.192
 default-router 172.61.106.193
 dns-server 192.168.1.1
 exit

do wr
```

---

## Router2 Configuration (Inter-VLAN Routing)

### Interface IP

```bash
conf t
interface e0/1
 ip address 192.168.1.254 255.255.255.0
 no shutdown

interface e0/0
 no shutdown
```

### Sub-Interfaces

```bash
interface e0/0.153
 encapsulation dot1q 153
 ip address 172.61.153.129 255.255.255.224
 no shutdown
 exit

interface e0/0.106
 encapsulation dot1q 106
 ip address 172.61.106.193 255.255.255.192
 no shutdown
 exit
```

### Default Route

```bash
ip route 0.0.0.0 0.0.0.0 192.168.1.1
```

### DHCP Relay

```bash
interface e0/0.106
 ip helper-address 192.168.1.1
 exit

do wr
```

---

## Final Result

- Inter-VLAN Routing ทำงานผ่าน Router2
- DHCP & NAT ทำงานที่ Router1
- ubuntu1 (Static) และ ubuntu2 (DHCP) สื่อสารกันได้
- ออก Internet ได้ทั้งสองเครื่อง

**✔ Network ถูกต้องครบตามโจทย์**

