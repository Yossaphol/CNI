# Instructions

## IP Assignment & Subnetting

ใส่ค่าลงในตารางด้านล่าง และ configure อุปกรณ์ให้ถูกต้องครบถ้วน ตามข้อกำหนดต่อไปนี้

---

### General Requirements

- กำหนดให้ IPv4 Network หลักเป็นรูปแบบ
  
  **172.X.Y.128/25**

- ใช้ **VLAN A และ VLAN B** โดยที่ค่า **X, Y, A, B** เป็นค่าที่สุ่มเฉพาะของแต่ละคน
- ต้องทำ **VLSM** ให้เหมาะสมกับจำนวน Host

---

### Subnet Design Rules

- **Subnet แรก (VLAN A)**
  - มี `ubuntu1` และ `Switch`
  - ต้องรองรับ Host ได้ **อย่างน้อย 30 IP**
  - ต้องเลือก Subnet ที่ *เล็กและใกล้เคียงที่สุด*

- **Subnet ที่สอง (VLAN B)**
  - มี `ubuntu2`
  - ต้องรองรับ Host ได้ **อย่างน้อย 50 IP**
  - ต้องเลือก Subnet ที่ *เล็กและใกล้เคียงที่สุด*

- ต้องแบ่ง **VLAN A ก่อน VLAN B**
  - ดังนั้น **Subnet Number ของ VLAN A ต้องน้อยกว่า VLAN B**

---

### Router-to-Router Network

- กำหนดให้ Network ระหว่าง **Router1 และ Router2** เป็น

```
192.168.1.0/24
```

---

### IPv4 Assignment Rules

- **Router1 และ Router2**
  - ใช้ Host ID ที่สามารถใช้งานได้ **ลำดับที่ 1** ของ Subnet
  - **ยกเว้น** Interface `E0/1` ของ Router2 ให้ใช้ **Host ลำดับสุดท้าย** ของ Subnet

- **ubuntu1**
  - ใช้ Host ID ลำดับที่ **2** ของ Subnet
  - DNS Server คือ **Router1**

- **ubuntu2**
  - ต้องได้รับค่า **IPv4, Subnet Mask, Gateway และ DNS ผ่าน DHCPv4 จาก Router1**
  - IPv4 ที่ได้รับต้องอยู่ในช่วง

```
Host offset 5 – 10 ของ Subnet VLAN B
```

  - DNS Server คือ **Router1**

- **Switch (Management Interface)**
  - ใช้ Host ID ลำดับ **สุดท้ายของ Subnet VLAN A**

---

## Network Variables

| Variable | VLAN ID | Variable | Network (CIDR) |
|--------|--------|---------|----------------|
| VLAN A | 153 | CIDR Q | 172.61.153.128/27 |
| VLAN B | 106 | CIDR R | 172.61.106.192/26 |
| VLAN C | 182 | CIDR S | - |

---

## Your Devices

| Device (Management Interface) | Management IP |
|-----------------------------|---------------|
| Switch | Not assigned |
| Router1 (GigabitEthernet0/0) | 10.70.38.107 |
| Router2 | Not assigned |
| ubuntu1 | Not assigned |
| ubuntu2 | Not assigned |

---

## IP Address Configuration Table

| Device.Interface | IPv4 Address | Subnet Mask | Default Gateway | DNS |
|------------------|-------------|-------------|-----------------|-----|
| router1.eth0_0 | 192.168.1.1 | 255.255.255.0 | — | — |
| router2.eth0_0.153 | 172.61.153.129 | 255.255.255.224 | — | — |
| router2.eth0_0.106 | 172.61.106.193 | 255.255.255.192 | — | — |
| router2.eth0_1 | 192.168.1.254 | 255.255.255.0 | 192.168.1.1 | — |
| switch.vlan153 | 172.61.153.158 | 255.255.255.224 | 172.61.153.129 | — |
| ubuntu1.eth0 | 172.61.153.130 | 255.255.255.224 | 172.61.153.129 | 192.168.1.1 |
| ubuntu2.eth0 | 172.61.106.197 *(DHCP)* | 255.255.255.192 | 172.61.106.193 | 192.168.1.1 |

---