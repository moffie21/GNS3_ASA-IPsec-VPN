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
ISP_3650_R  ➔  LA_3650_WAN  ➔   LA_ASA1   ⤵  
                                             LA_3650_DMZ   ➔  DMZ_SRV
                         (Failover) ↕      ╳
                                             LA_3650       ➔  LA_Server_20
                             ➔   LA_ASA2   ⤴              ➔  LA_IT_30
                                                           ➔  LA_2960         ➔  LA_Users_40
                                                                               ➔  LA_Wireless_50
.
            ➔     SD_ASA    ➔   SD_3650   ➔   SD_2960    ➔  SD_User_40
                                                           ➔  SD_Wireless_50
.
            ➔     FW_SF_R   ➔  SF_2960_1  ➔  SF_2960_2   ➔  SF_User_40
                                                           ➔  SF_Wireless_50
.
            ➔     NY_ASA    ➔   NY_3650   ➔   NY_2960    ➔  NY_User_40
                                                           ➔  NY_Wireless_50
.
            ➔     FW_MI_R   ➔  MI_2960_1  ➔  MI_2960_2   ➔  MI_User_40
                                                           ➔  MI_Wireless_50
