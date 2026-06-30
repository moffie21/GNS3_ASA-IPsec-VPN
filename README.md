# GNS3: Configuring ASA AND IPsec VPN
### INTENTION
For educational purposes showcasing my technical networking abilities.

### DESCRIPTION
Built and configured a logical topology featuring ASA and IPsec VPN protocols in GNS3 lab environment.

### GOALS
- Allow DMZ devices to ping devices in LA network.
- Successfully have each L3 device, except ISP; SF; and MI, learn single-area OSPF routes.
- Successfully SSH from a L2 or L3 device to LA_3650_DMZ and, or LA_3650 devices.

### PROTOCOLS IMPLEMENTED
- Subnetting one /8 organization into 5 /16 city branches (LA, SD, SF, NY, MI) down to /30 subnets
- VLANs
- VTP
- Trunking
- Inter-VLAN routing
- NAT (PAT), static to LA_3650_DMZ, static to LA_3650
- ASA and ASA Failover config
- Site-to-Site IPsec VPN (LA and NY, LA and SD, SD and SF, NY and MI)
- OSPF (single-area) (not for ISP_3650_R, FW_SF_R, FW_MI_R)
- Basic Security and SSH

### CHALLENGES
- Getting NAT and ACLs correct so they don't interfere with other protocols.

### TOPOLOGY
```
 ISP_3650_R  ➔  LA_3650_WAN  ➔   LA_ASA1  ⤵  
                                              LA_3650_DMZ   ➔  DMZ_SRV  
                           (Failover) ↕     ╳  
                                               LA_3650      ➔  LA_Server_20  
                              ➔   LA_ASA2   ⤴              ➔  LA_IT_30  
                                                            ➔  LA_2960      ➔  LA_Users_40  
                                                                             ➔  LA_Wireless_50  
  
             ➔     SD_ASA    ➔   SD_3650   ➔   SD_2960    ➔  SD_User_40  
                                                            ➔  SD_Wireless_50  
  
             ➔     FW_SF_R   ➔  SF_2960_1  ➔  SF_2960_2   ➔  SF_User_40  
                                                            ➔  SF_Wireless_50  
  
             ➔     NY_ASA    ➔   NY_3650   ➔   NY_2960    ➔  NY_User_40  
                                                            ➔  NY_Wireless_50  
 
            ➔     FW_MI_R   ➔  MI_2960_1  ➔  MI_2960_2   ➔  MI_User_40  
                                                           ➔  MI_Wireless_50
```
***
### CONFIGURATIONS
- ISP
- LA Network
- SD Network
- SF Network
- NY Network
- MI Network

==ISP_3650_R (L3) device==
hostname ISP_3650_R
no ip routing
!
int g0/0
 ip address 2.2.10.1 255.255.255.248
 no shutdown
int g0/1
 ip address 3.3.20.65 255.255.255.252
 no shutdown
int g0/2
 ip addr 4.4.21.129 255.255.255.252
 no shutdown
int g0/3
 ip address 33.33.30.65 255.255.255.252
 no shutdown
int g1/0
 ip address 44.44.31.129 255.255.255.252
 no shutdown
!
!
! **SSH Access**
!
ip domain-name CIS.EDU
crypto key generate rsa modulus 2048
!
username admin privilege 15 secret LabPass061
!
line vty 0 4
 login local
 transport input ssh
!

### CONFIGURATIONS: LA NETWORK
==LA_3650_WAN (L2) device==
hostname LA_3650_WAN
no ip routing
!

==LA_ASA1 (5506) (L3) (Active) device==
enable password P@ssw0rd
!
hostname LA_ASA1
domain-name CIS.EDU
!  
interface g0/0
 no shut
!
interface g0/0.250
 vlan 250
 nameif inside
 security-level 100
 ip address 10.10.250.2 255.255.255.248 standby 10.10.250.3
 no shutdown
!
interface g0/1
 nameif outside
 security-level 0
 ip address 2.2.10.2 255.255.255.248 standby 2.2.10.3
 no shutdown
!
interface g0/3
 nameif dmz
 security-level 50
 ip address 192.168.250.1 255.255.255.0 standby 192.168.250.2
 no shutdown
!
interface g0/5
 no shutdown
!
interface g0/6
 no shutdown
!
route outside 0.0.0.0 0.0.0.0 2.2.10.1
!
!
! **Allowing ICMP through ASA**
!
policy-map global_policy
 class inspection_default
  inspect icmp 
  inspect icmp error
!
!
! **Allow ICMP to ASA's inside interface from another site**
!
management-access inside
!
!
! **Failover**
!
failover lan unit primary
failover lan interface FAILOVER g0/5
failover interface ip FAILOVER 172.16.2.1 255.255.255.0 standby 172.16.2.2
failover link STATEFUL g0/6
failover interface ip STATEFUL 172.16.1.1 255.255.255.0 standby 172.16.1.2
failover key CIS101101
failover
prompt hostname priority state
!
!
! **Phase 1 (IKEv1)**
!
crypto ikev1 enable outside
!
crypto ikev1 policy 10
 hash sha
 authentication pre-share
 group 5
 lifetime 86400
 encryption aes-256
!
tunnel-group 3.3.20.66 type ipsec-l2l
tunnel-group 3.3.20.66 ipsec-attributes
 ikev1 pre-shared-key LA10SD20
!
tunnel-group 33.33.30.66 type ipsec-l2l
tunnel-group 33.33.30.66 ipsec-attributes
 ikev1 pre-shared-key LA10NY130
!
!
! **Phase 2 (IPsec)**
!
object network N_10.10.0.0_16
 subnet 10.10.0.0 255.255.0.0
object network N_10.20.0.0_16
 subnet 10.20.0.0 255.255.0.0
object network N_10.130.0.0_16
 subnet 10.130.0.0 255.255.0.0
!
access-list IPSEC_NY_ACL extended permit ip object N_10.10.0.0_16 object N_10.130.0.0_16
!
access-list IPSEC_SD_ACL extended permit ip object N_10.10.0.0_16 object N_10.20.0.0_16
!
!
! **Auto NAT**
!
object network INSIDE_LA
 subnet 10.10.0.0 255.255.0.0
 nat (inside,outside) dynamic interface
!
object network DMZ_L2
 host 192.168.250.10
 nat (dmz,outside) static 1.1.1.10
!
object network LA_L3L2
 host 10.10.32.1
 nat (inside,outside) static 1.1.1.5
!
!
! **Allow ICMP Echo from DMZ to Inside**
!
access-list DMZ_INSIDE extended permit icmp host 192.168.250.10 10.10.0.0 255.255.0.0
!
access-group DMZ_INSIDE in interface dmz
!
!
! **Allow SSH from Outside to 3650_DMZ and LA_3650**
!
access-list INBOUND extended permit tcp any host 1.1.1.10 eq 22
access-list INBOUND extended permit tcp any host 1.1.1.5 eq 22
!
access-group INBOUND in interface outside
!
!
! **NAT Exemption (No NAT)**
!
nat (inside,outside) source static N_10.10.0.0_16 N_10.10.0.0_16 destination static N_10.20.0.0_16 N_10.20.0.0_16 no-proxy-arp route-lookup
nat (inside,outside) source static N_10.10.0.0_16 N_10.10.0.0_16 destination static N_10.130.0.0_16 N_10.130.0.0_16 no-proxy-arp route-lookup
!
crypto IPsec ikev1 transform-set IPSEC_SET esp-aes-256 esp-sha-hmac
!
crypto map IPSEC_MAP 20 match address IPSEC_SD_ACL
crypto map IPSEC_MAP 20 set peer 3.3.20.66
crypto map IPSEC_MAP 20 set ikev1 transform-set IPSEC_SET
crypto map IPSEC_MAP 20 set security-association lifetime seconds 86400
crypto map IPSEC_MAP 20 set pfs group14
!
crypto map IPSEC_MAP 30 match address IPSEC_NY_ACL
crypto map IPSEC_MAP 30 set peer 33.33.30.66
crypto map IPSEC_MAP 30 set ikev1 transform-set IPSEC_SET
crypto map IPSEC_MAP 30 set security-association lifetime seconds 86400
crypto map IPSEC_MAP 30 set pfs group14
!
crypto map IPSEC_MAP interface outside
!
same-security-traffic permit intra-interface
!
!
! **OSPF**
!
router ospf 10
 network 10.10.250.0 255.255.255.248 area 0
 network 192.168.250.0 255.255.255.0 area 0
!

==LA_ASA2 (5506) (L3) (Standby) device==
enable password P@ssw0rd
!
hostname LA_ASA2
domain-name CIS.EDU
!
interface g0/0
 no shutdown
!
interface g0/0.250
 vlan 250
 nameif inside
 security-level 100
 ip address 10.10.250.2 255.255.255.248 standby 10.10.250.3
 no shutdown
!
interface g0/1
 nameif outside
 security-level 0
 ip address 2.2.10.2 255.255.255.248 standby 2.2.10.3
 no shutdown
!
interface g0/3
 nameif dmz
 security-level 50
 ip address 192.168.250.1 255.255.255.0 standby 192.168.250.2
 no shutdown
!
interface g0/5
 no shutdown
!
interface g0/6
 no shutdown
!
!
! **Failover**
!
failover lan unit secondary
failover lan interface FAILOVER g0/5
failover interface ip FAILOVER 172.16.2.1 255.255.255.0 standby 172.16.2.2
failover link STATEFUL g0/6
failover interface ip STATEFUL 172.16.1.1 255.255.255.0 standby 172.16.1.2
failover key CIS101101
failover
!

==LA_3650 (L3/L2) (VTP Server) (Inter-VLAN Routing) device==
hostname LA_3650
ip routing
!
ip route 0.0.0.0 0.0.0.0 10.10.250.2
!
!
! **VLANs**
!
vlan 10
 name Management
vlan 20
 name Server
vlan 30
 name IT
vlan 40
 name Users
vlan 50
 name Wireless
vlan 250
!
!
! **VTP**
!
vtp mode server
vtp domain LA
vtp password cisco
!
!
! **Inter-VLAN Routing**
!
interface vlan 10
 description Mgmt VLAN
 ip addr 10.10.32.1 255.255.255.128
 no shutdown
!
interface vlan 20
 description Server VLAN
 ip addr 10.10.64.1 255.255.254.0
 no shutdown
!
interface vlan 30
 description IT VLAN
 ip addr 10.10.96.1 255.255.254.0
 no shutdown
!
interface vlan 40
 description Users VLAN
 ip addr 10.10.160.1 255.255.240.0
 no shutdown
!
interface vlan 50
 description Wireless VLAN
 ip addr 10.10.192.1 255.255.224.0
 no shutdown
!
interface vlan 250
 ip addr 10.10.250.1 255.255.255.248
 no shutdown
!
interface g0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 250
 no shutdown
!
interface g0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 250
 no shutdown
!
interface g1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,250
 no shutdown
!
interface g2/0
 switchport mode access
 switchport access vlan 20
 no shutdown
!
interface g3/0
 switchport mode access
 switchport access vlan 30
 no shutdown
!  
!  
! **OSPF**  
!  
router ospf 10  
 network 10.10.0.0 0.0.255.255 area 0  
 passive-interface vlan 10  
 passive-interface vlan 20  
 passive-interface vlan 30  
 passive-interface vlan 40  
 passive-interface vlan 50  
!  
!  
! **SSH Access**  
!  
ip domain-name CIS.EDU  
crypto key generate rsa modulus 2048  
!  
username admin privilege 15 secret LabPass061  
!  
line vty 0 4  
 login local  
 transport input ssh  
!  
  
==LA_Server_20 Server VPC==  
set pcname LA_Server_20  
ip 10.10.64.254/23 10.10.64.1  
  
  
==LA_IT_30 VPC==  
set pcname LA_IT_30  
ip 10.10.96.100/23 10.10.96.1  
  
  
==LA_3650_DMZ (L2 only) device==  
hostname LA_3650_DMZ  
no ip routing  
!  
int g0/0  
 no shutdown  
int g1/0  
 no shutdown  
int g2/0  
 no shutdown  
!  
int vlan 1  
 ip address 192.168.250.10 255.255.255.0  
 no shut  
!  
ip default-gateway 192.168.250.1  
!  
!  
! **SSH Access**  
!  
ip domain-name CIS.EDU  
crypto key generate rsa modulus 2048  
!  
username admin privilege 15 secret LabPass061  
!  
line vty 0 4  
 login local  
 transport input ssh  
!  
  
==DMZ_SRV Server VPC==  
set pcname DMZ_SRV  
ip 192.168.250.254/24 192.168.250.1  
  
  
==LA_2960 (L2) (VTP Client) device==  
hostname LA_2960  
no ip routing  
!  
interface vlan 10  
 ip addr 10.10.32.10 255.255.255.128  
 no shutdown  
 ip default-gateway 10.10.32.1  
!  
!  
! **VTP**  
!  
vtp mode client
vtp domain LA
vtp password cisco
!
!
! **Inter-VLAN Routing**
!
interface g1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
!
interface g2/0
 switchport mode access
 switchport access vlan 40
 no shutdown
!
interface g3/0
 switchport mode access
 switchport access vlan 50
 no shutdown
!

====================================
LA_Users_40 VPC
====================================
set pcname LA_Users_40
ip 10.10.160.101/20 10.10.160.1


====================================
LA_Wireless_50 VPC
====================================
set pcname LA_Wireless_50
ip 10.10.192.101/19 10.10.192.1
