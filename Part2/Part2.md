# Network Lab Configuration (Your IP Scheme)

---

## Instructions – Basic Connectivity

ต้องสามารถทดสอบได้ครบตามเงื่อนไขต่อไปนี้

1. PC-1 (ubuntu1) สามารถ Ping IPv4 Switch ได้
2. PC-1 (ubuntu1) สามารถ Ping IPv4 PC-2 (ubuntu2) ได้
3. PC-1 (ubuntu1) สามารถ Ping IPv4 Router1 (E0/0) ได้
4. PC-2 (ubuntu2) สามารถ Ping IPv4 Switch ได้
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
sudo nano /etc/netplan/01-netconfig.yaml
```

```bash
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses: 
        - 172.61.153.130/27
      gateway4: 172.61.153.129
```

```bash
sudo netplan apply
```

---

## ubuntu2 Configuration (DHCP)

```bash
sudo nano /etc/netplan/01-netconfig.yaml
```

```bash
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
```

```bash
sudo netplan apply
```

---

## Switch Configuration

### Create VLANs & SVI

```bash
conf t
(config)# vlan 153
(config-vlan)# ex
(config)# vlan 106
(config-vlan)# ex

(config)# int vlan 153
(config-if)# ip add 172.61.153.158 255.255.255.224
(config-if)# no shutdown
(config-if)# ex
(config)# int vlan 106
(config-if)# no shut
(config-if)# ex

(config)# ip default-gateway 172.61.153.129
(config)# no ip routing
(config)# do wr
```

### Interface Mode Selection

```bash
conf t
(config)# int e0/1   # ubuntu1
(config-if)# switchport mode access
(config-if)# switchport access vlan 153
(config-if)# exit

(config)# int e0/1   # ubuntu2
(config-if)# switchport mode access
(config-if)# switchport access vlan 106
(config-if)# exit

(config)# int e0/0.  # to Router 2
(config-if)# switchport trunk encapsulation dot1Q
(config-if)# switchport mode trunk
(config-if)# exit
(config)# do wr

do wr
```

---

## Router1 Configuration (DHCP + NAT)

### Add IP

```bash
conf t
(config)# int e0/1
(config-if)# no shut
(config-if)# ip address dhcp
(config-if)# ex
(config)# int e0/0
(config-if)# no shut
(config-if)# ip address 192.168.1.1 255.255.255.0
(config-if)# ip nat inside
(config-if)# ex
(config-if)# do wr
```

### Routing

```bash
(config)# ip route 172.61.153.128 255.255.255.224 192.168.1.254
(config)# ip route 172.61.106.192 255.255.255.192 192.168.1.254
(config)# do wr
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

```bash
# ของ Stang
(config)# ip access-list standard net
(config-std-ncal)# permit any
(config-std-ncal)# ex
(config)# int e0/0
(config-if)# ip nat inside
(config-if)# ex
(config)# int e0/1
(config-if)# ip nat outside
(config-if)# ex
(config)# ip nat inside source list net interface e0/1 overload
(config)# do wr
```

### DHCP Configuration (VLAN 106)

```bash
(config)# ip dhcp excluded-address 172.61.106.193 172.61.106.196
(config)# ip dhcp excluded-address 172.61.106.203 172.61.106.254

(config)# ip dhcp pool VLAN106
(dhcp-config)# network 172.61.106.192 255.255.255.192
(dhcp-config)# default-router 172.61.106.193
(dhcp-config)# dns-server 192.168.1.1
(dhcp-config)# exit
(config)# do wr
```

---

## Router2 Configuration (Inter-VLAN Routing)

### Add IP

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

