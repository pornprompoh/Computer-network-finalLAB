# Final Network SpecialTopicsInComputer อาจารย์ : อำพล

## 1.คั้งค่า ospf และ default routing

### - CoreSW (Catalyst 3560 – L3 Switch)
```cisco
enable
conf t
hostname CoreSW
no ip domain-lookup

! --- VLAN / SVI ---
vlan 10
 name VLAN10
vlan 20
 name VLAN20
exit

interface Vlan10
 ip address 192.23.1.1 255.255.255.0
 no shutdown
interface Vlan20
 ip address 192.23.2.1 255.255.255.0
 no shutdown

! --- Link ไป R1 ---
interface GigabitEthernet0/1
 description to-R1
 no switchport
 ip address 192.23.3.2 255.255.255.252
 no shutdown

! --- Trunk ไป SW1–SW4 ---
interface FastEthernet0/1
 description trunk-to-SW1
 switchport mode trunk
 no shutdown
interface FastEthernet0/2
 description trunk-to-SW2
 switchport mode trunk
 no shutdown
interface FastEthernet0/3
 description trunk-to-SW3
 switchport mode trunk
 no shutdown
interface FastEthernet0/4
 description trunk-to-SW4
 switchport mode trunk
 no shutdown

! --- เปิด L3 Routing + OSPF ---
ip routing

router ospf 1
 router-id 1.1.1.1
 network 192.23.1.0 0.0.0.255 area 0
 network 192.23.2.0 0.0.0.255 area 0
 network 192.23.3.0 0.0.0.3   area 0

! --- Default route ออกไป R1 ---
ip route 0.0.0.0 0.0.0.0 192.23.3.1

end
wr mem

```

### - ISP Router

```cisco
enable
conf t
hostname ISP
no ip domain-lookup

interface GigabitEthernet0/0/0
 description to-R1
 ip address 200.200.200.1 255.255.255.252
 no shutdown
interface GigabitEthernet0/0/1
 description to-Server
 ip address 200.200.100.1 255.255.255.252
 no shutdown

! default กลับมาที่ R1
ip route 0.0.0.0 0.0.0.0 200.200.200.2

end
wr mem

```

### - R1

```cisco
enable
conf t
hostname R1
no ip domain-lookup

! --- to CoreSW ---
interface GigabitEthernet0/0/1
 description to-CoreSW
 ip address 192.23.3.1 255.255.255.252
 no shutdown

! --- to ISP ---
interface GigabitEthernet0/0/0
 description to-ISP
 ip address 200.200.200.2 255.255.255.252
 no shutdown

! --- OSPF กับ CoreSW ---
router ospf 1
 router-id 2.2.2.2
 network 192.23.3.0 0.0.0.3 area 0

! --- Default route ไป ISP ---
ip route 0.0.0.0 0.0.0.0 200.200.200.1

end
wr mem

```

### - SW1
```cisco
enable
conf t
hostname SW1
no ip domain-lookup
interface FastEthernet0/1
switchport mode trunk
no shutdown
interface FastEthernet0/2
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
interface FastEthernet0/3
switchport mode access
switchport access vlan 20
spanning-tree portfast
no shutdown
end
wr mem

```

### - SW2
```cisco
enable
conf t
hostname SW2
no ip domain-lookup
interface FastEthernet0/1
switchport mode trunk
no shutdown
interface FastEthernet0/2
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
interface FastEthernet0/3
switchport mode access
switchport access vlan 20
spanning-tree portfast
no shutdown
end
wr mem

```

### - SW3
```cisco
enable
conf t
hostname SW3
no ip domain-lookup
interface FastEthernet0/1
switchport mode trunk
no shutdown
interface FastEthernet0/2
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
interface FastEthernet0/3
switchport mode access
switchport access vlan 20
spanning-tree portfast
no shutdown
end
wr mem

```

### - SW4
```cisco
enable
conf t
hostname SW4
no ip domain-lookup
interface FastEthernet0/1
switchport mode trunk
no shutdown
interface FastEthernet0/2
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
interface FastEthernet0/3
switchport mode access
switchport access vlan 20
spanning-tree portfast
no shutdown
end
wr mem

```

## 2.บล็อก ACL บล็อกไม่ให้ VLAN 10 เข้า WebServer 
### R1
```cisco
conf t
access-list 100 deny tcp 192.23.1.0 0.0.0.255 host 200.200.100.2 eq 80
access-list 100 deny tcp 192.23.1.0 0.0.0.255 host 200.200.100.2 eq 443
access-list 100 permit ip any any

interface g0/0/0
 ip access-group 100 out
```

## 3.เลือกใช้ NAT หน่วยงานมี Public IP เพี่ยง 1 อัน

```cisco
conf t
!
! ระบุ ACL สำหรับ LAN ที่จะถูกแปล
access-list 1 permit 192.23.0.0 0.0.255.255
!
! ตั้งค่า NAT Overload
ip nat inside source list 1 interface g0/0/0 overload
!
! กำหนด inside/outside
interface g0/0/1
 ip nat inside
!
interface g0/0/0
 ip nat outside
!
end
wr mem
```

## รูปภาพเพิ่มเติมเสริมเพื่อดู

PC1
<img width="700" height="316" alt="image" src="https://github.com/user-attachments/assets/bd3400c5-46bd-4cc9-8a09-6ae50910b043" />
PC2
<img width="699" height="307" alt="image" src="https://github.com/user-attachments/assets/26c763cd-c197-4aa4-a31f-1736dc5fb514" />
PC3
<img width="701" height="313" alt="image" src="https://github.com/user-attachments/assets/b97bf2f6-e4a6-4ac0-a29c-f3f1f295964b" />
PC4
<img width="701" height="303" alt="image" src="https://github.com/user-attachments/assets/1a8f134b-0d3a-437a-9086-2a673ef99b02" />
PC5
<img width="702" height="301" alt="image" src="https://github.com/user-attachments/assets/a2acd692-a393-4f0c-9254-ebd405154166" />
PC6
<img width="700" height="305" alt="image" src="https://github.com/user-attachments/assets/1f03f4ea-eec7-4811-a577-440a55a15a23" />
PC7
<img width="699" height="314" alt="image" src="https://github.com/user-attachments/assets/c8ead6e8-e096-4dde-a239-fe0b24f8a0c5" />
PC8
<img width="699" height="296" alt="image" src="https://github.com/user-attachments/assets/9ccc605c-3883-440c-87f0-63d910c5f02b" />










