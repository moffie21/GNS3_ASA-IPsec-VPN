# GNS3: Configuring ASA and Site-to-Site IPsec VPN
## INTENTION
For educational purposes showcasing my technical networking abilities.
  
## WHAT'S THIS?
Built and configured a logical topology of a multi-city organization featuring ASA and IPsec VPN routing protocols in GNS3 lab environment.
  
## GOALS - Proof Config Implementation Worked Successfully
- Allow DMZ devices to ping devices in LA network.
- Successfully have each L3 device, except ISP; SF; and MI, learn single-area OSPF routes.
- Successfully SSH from a L2 or L3 device to LA_3650_DMZ and, or LA_3650 devices.

## TOOLS
- VM platform: VMware Workstation Pro 25H2
  - Lab environment: GNS3 v2.2.56.1
    -  Device:
        -  L3 Cisco 2911 router => Cisco IOSv 15.9(3)M2
        -  Cisco 5506 => Cisco ASAv 9.22.1.1-1
        -  L3/L2 Cisco 3650 switch => Cisco IOSv-L2
        -  L2 Cisco 2960 switch => Cisco IOSv-L2
        -  Servers and PC hosts => VPC
    - Console application: Solar-PuTTY
  
## PROTOCOLS IMPLEMENTED - What I Scripted
- Subnet one /8 organization into 5 /16 city branches (LA, SD, SF, NY, MI) down to /30 subnets
- VLANs (VLAN 10 "Management" /25, VLAN 20 "Server" /23 (LA only), VLAN 30 "IT" /23 (LA only), VLAN 40 "Users" /20, VLAN 50 "Wireless" /19, VLAN 250 /29 (LA only))
- VTP (Server/Client, domain, password)
- Trunking
- Inter-VLAN routing
- NAT (PAT), static to LA_3650_DMZ, static to LA_3650
- ASA Hub-and-Spoke, ASA Failover
- Site-to-Site IPsec VPN (LA and NY, LA and SD, SD and SF, NY and MI)
- OSPF (single-area) (not for ISP_3650_R, FW_SF_R, FW_MI_R)
- Basic Security and SSH (from the Internet to LA_3650_DMZ, LA_3650)
  
## CHALLENGES / LESSON(S) LEARNED
- Challenge: Getting NAT and all the ACLs correct so they don't interfere with other protocols.
- LL: It was fun and concise to create network objects.
- LL: Configuring NAT exemption ACLs to ensure specific traffic would route through the IPsec tunnels.
- LL: OSPF single-area only outputted for each private network to learn about its own subnets vs all private networks learning about all other private networks.
- LL: Configuring ASA Failover was easier than I initially thought.
  
## TOPOLOGY
```
 ISP_3650_R  ➔  LA_3650_WAN  ➔   LA_ASA1  ⤵  
                                              LA_3650_DMZ   ➔  DMZ_SRV  
                           (Failover) ↕     ╳  
                                               LA_3650      ➔  LA_Server_20  
                              ➔   LA_ASA2  ⤴               ➔  LA_IT_30  
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
