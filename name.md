ช้ Public IP ของ g0/0/0 ของ R1 สำหรับ NAT เฉพาะ VLAN10
conf t
!
access-list 1 permit 192.23.10.0 0.0.0.255
!
interface g0/0/0
 ip nat outside
!
interface g0/0/1.10
 ip nat inside
!
interface g0/0/1.20
 ip nat inside
!
ip nat inside source list 1 interface g0/0/0 overload
end



conf t
!
access-list 101 deny tcp 192.23.10.0 0.0.0.255 any eq 23
access-list 101 permit ip any any
!
interface g0/0/0
 ip access-group 101 in
end
