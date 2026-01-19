# Part 5: IPv6 Configuration

## Instructions
Student ID (last 3 digits): **145**

Use IPv6 prefix /64 for all subnets.

IPv6 format:
2001:6707:<StudentID>:<VLAN>::<Interface-ID>

No IPv6 Internet connectivity.

---

## Network Variables

| VLAN | VLAN ID | IPv6 Prefix |
|-----|--------|------------|
| VLAN A | 153 | 2001:6707:145:153::/64 |
| VLAN B | 106 | 2001:6707:145:106::/64 |
| VLAN C | 182 | 2001:6707:145:182::/64 |

---

## IPv6 Address Table

| Device.Interface | GUA | Prefix | Link-Local | Gateway |
|-----------------|-----|--------|-----------|---------|
| router1.eth0 | 2001:6707:145:182::1 | /64 | FE80::1 | N/A |
| router2.eth0.153 | 2001:6707:145:153::1 | /64 | FE80::1 | N/A |
| router2.eth0.106 | 2001:6707:145:106::1 | /64 | FE80::1 | N/A |
| router2.eth0_1 | 2001:6707:145:182::2 | /64 | FE80::2 | N/A |
| switch.vlan153 | 2001:6707:145:153::3 | /64 | FE80::3 | FE80::1 |
| ubuntu1.eth0 | 2001:6707:145:153::2 | /64 | FE80::2 | FE80::1 |
| ubuntu2.eth0 | SLAAC | /64 | N/A | FE80::1 |

---

## Configuration

### Router1
```
conf t
ipv6 unicast-routing
interface e0/0
 ipv6 address 2001:6707:145:182::1/64
 ipv6 address FE80::1 link-local
 no shutdown
```

### Router2
```
conf t
ipv6 unicast-routing

interface e0/0.153
 encapsulation dot1Q 153
 ipv6 address 2001:6707:145:153::1/64
 ipv6 address FE80::1 link-local

interface e0/0.106
 encapsulation dot1Q 106
 ipv6 address 2001:6707:145:106::1/64
 ipv6 address FE80::1 link-local

interface e0/1
 ipv6 address 2001:6707:145:182::2/64
 ipv6 address FE80::2 link-local
```

### Ubuntu1
```
sudo ip -6 addr add 2001:6707:145:153::2/64 dev eth0
sudo ip -6 route add default via fe80::1 dev eth0
```

### Ubuntu2 (SLAAC)
```
sudo sysctl -w net.ipv6.conf.eth0.accept_ra=1
sudo dhclient -6 eth0
```

### Switch
```
interface vlan 153
 ipv6 address 2001:6707:145:153::3/64
 ipv6 address FE80::3 link-local
```

---

## Verification
```
ip -6 addr
ping6 fe80::1
ping6 2001:6707:145:153::1
```
