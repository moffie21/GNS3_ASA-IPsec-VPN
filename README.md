# GNS3_ASA-IPsec-VPN
### For educational purposes

### DESCRIPTION
Built a logical topology featuring ASA and IPsec VPN configurations in GNS3 lab environment.

### GOALS
- Allow DMZ devices to ping devices in LA network.
- Successfully have each L3 device, except ISP; SF; and MI, learn single-area OSPF routes.
- Successfully SSH from a L2 or L3 device to LA_3650_DMZ and, or LA_3650 devices.

### PROTOCOLS IMPLEMENTED
- Subnetting from /16 down to /30
- VLANs
- VTP
- Trunking
- Inter-VLAN routing
- NAT
- ASA firewall
- Site-to-Site IPsec VPN
- OSPF
- Basic Security and SSH

### CHALLENGES
- Getting NAT and ACLs correct so they don't interfere with other protocols.
