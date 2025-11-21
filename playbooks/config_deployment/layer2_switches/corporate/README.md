Ansible Playbooks
############################################################################################################################

L2 switch
==============================================================================

#######################################################################################
Streamit Radio Config
For new stores, streamit radio will be plugged into port 12 of first available switch. 
Please put this port into vlan 12 for main stores or vlan112 for franchise stores.
#######################################################################################

#######################################################################################
Fridge Monitoring
For new stores, fridge monitoring will be plugged into port 13 of first available switch. 
Please put this port into vlan 42 for corp stores or vlan142 for franchise stores.
#######################################################################################

#######################################################################################
Biometrics
For new stores, fridge monitoring will be plugged into port 14 and 15 of first available switch. 
Please put this port into vlan 22 for main stores or vlan122 for franchise stores.
#######################################################################################

##################
NOTES on Variables

DDD hostname
EEE IP address AND subnet mask of switch in management vlan
FFF next hop IP address for default-gateway
GGG enable secret
HHH f@11b@ck secret
JJJ no find replace! access vlan for the port
KKK no find replace! voice vlan for the port (differs for cctv ovelay sites- vlan 51 or 32)
LLL no find replace! description- need to discuss standardising the description

##################

hostname DDD
!
!
!
vlan 11
 name Infrastructure
!
vlan 12
 name Kiosk
!
vlan 21
 name BackOffice
!
vlan 22
 name Scanners
!
vlan 31
 name Tills_POS
!
vlan 32
 name E-Tailing
!
vlan 41
 name IPT
!
vlan 42
 name 3rd_Party
!
vlan 101
 name Tyme_WAN-Breakout
!
vlan 102
 name Tyme_LAN-SSID
!
vlan 104  
 name pnp_scanners
!
vlan 105
 name pnp_devices_sui
!
vlan 106
 name pnpwifi_byod_vip
!
vlan 107
 name pnpwifi_byod_default
!
vlan 108
name pnp_3rdparty
!
vlan 109
 name pnp_staff
!
!
!
interface Vlan1
description Do Not Use
no ip address
shutdown
!
interface Vlan11
description Network_MGMT
ip address EEE
no ip redirects
no ip proxy-arp
no ip unreachables
no shutdown
!
!
ip default-gateway FFF
ip route 0.0.0.0 0.0.0.0 FFF
!
!
aaa session-id common
no service pad
service timestamps debug datetime localtime msec show-timezone
service timestamps log datetime localtime msec show-timezone
service tcp-keepalives-in
service tcp-keepalives-out
service password-encryption
service sequence-numbers
service password-recovery
no service  tcp-small-servers
no service udp-small-servers
no service config
no service compress-config
ip tcp synwait-time 7
no ip http server
no ip http secure-server
no ip finger
no ip source-route
no ip gratuitous-arps
no ip identd
ip dhcp bootp ignore

!
aaa new-model
!
boot-start-marker
boot-end-marker
!
logging buffered 16000
logging console errors
enable secret GGG
!
username f@11b@ck secret HHH
!
!
tacacs server PNPTACACS_1
 address ipv4 10.96.1.153
 key 7 051D31013879195B05221954452D0C070C
tacacs server PNPTACACS_2
 address ipv4 10.97.1.153
 key 7 051D31013879195B05221954452D0C070C

aaa group server tacacs+ PNPTACACSGROUP
 server name PNPTACACS_1
 server name PNPTACACS_2
 ip tacacs source-interface Vlan11

aaa new-model
aaa authentication login default group PNPTACACSGROUP local
aaa authentication enable default group PNPTACACSGROUP enable
aaa authorization exec default group PNPTACACSGROUP if-authenticated
aaa authorization console
aaa accounting exec default start-stop group PNPTACACSGROUP
aaa accounting commands 15 default start-stop group PNPTACACSGROUP
aaa accounting network default start-stop group PNPTACACSGROUP
aaa accounting connection default start-stop group PNPTACACSGROUP
aaa session-id common

aaa authentication dot1x default group ISE-RADIUS
aaa authorization network default group ISE-RADIUS
aaa accounting update newinfo periodic 71582
aaa accounting dot1x default start-stop group ISE-RADIUS
radius-server deadtime 10
radius-server dead-criteria time 5 tries 3
aaa server radius dynamic-author
 client 10.96.1.152 server-key Pennywise@123
 client 10.97.1.152 server-key Pennywise@123
!
!
device-sensor accounting
device-sensor notify all-changes

dot1x system-auth-control
source vlan to be infrastructure vlan
ip radius source-interface Vlan11
radius-server attribute 6 on-for-login-auth
radius-server attribute 8 include-in-access-req
radius-server attribute 25 access-request include
radius-server deadtime 10
radius-server dead-criteria time 5 tries 3


radius-server vsa send authentication
radius-server vsa send accounting

mac address-table notification change
mac address-table notification mac-move

radius server BREE_PSN1
    address ipv4 10.96.1.152 auth-port 1812 acct-port 1813
    automate-tester username rad1 ignore-acct-port idle-time 120
    key Pennywise@123
radius server RANDVIEW_PSN2
    address ipv4 10.97.1.152 auth-port 1812 acct-port 1813
    automate-tester username rad2 ignore-acct-port idle-time 120
    key Pennywise@123
aaa group server radius ISE-RADIUS
 !Servers will be accessed based on the order here
    server name BREE_PSN1
    server name RANDVIEW_PSN2
    deadtime 15
!
archive
 log config
  logging enable
  logging size 1000
  notify syslog contenttype plaintext
  hidekeys
!
no ip source-route
ip routing
no ip gratuitous-arps
!
!
no ip domain-lookup
ip domain-name pnp.co.za
ip name-server 10.2.35.5
ip name-server 10.2.36.5
vtp version 2
vtp domain PNP
vtp mode transparent
udld enable
!
crypto key generate rsa modulus 2048
!
!
spanning-tree mode rapid-pvst
spanning-tree portfast bpduguard default
spanning-tree extend system-id
spanning-tree pathcost method long
spanning-tree portfast default
spanning-tree portfast bpduguard default
spanning-tree etherchannel guard misconfig
spanning-tree loopguard default
!
!
udld enable
errdisable recovery cause udld
errdisable recovery cause bpduguard
errdisable recovery cause channel-misconfig
errdisable recovery cause loopback
errdisable recovery cause link-flap
errdisable recovery cause security-violation
errdisable recovery cause dtp-flap
errdisable recovery cause sfp-config-mismatch
errdisable recovery cause gbic-invalid
errdisable recovery cause storm-control
errdisable recovery cause arp-inspection
errdisable recovery interval 1800
port-channel load-balance src-dst-ip
!
vlan internal allocation policy ascending
!
ip ssh authentication-retries 3
ip ssh time-out 60
ip ssh version 2
lldp run
cdp run
!
!
ip access-list extended VTY-ACCESS
permit tcp host 10.1.9.213 any eq 22
permit tcp host 10.96.5.2 any eq 22
permit tcp host 10.96.5.5 any eq 22
permit tcp host 10.1.9.40 any eq 22
permit tcp host 10.1.9.140 any eq 22
permit tcp host 10.225.4.80 any eq 22
permit tcp host 10.224.24.174 any eq 22
permit tcp host 10.224.25.91 any eq 22
permit tcp host 10.1.9.201 any eq 22
permit tcp host 10.1.9.223 any eq 22
permit tcp host 10.1.12.109 any eq 22
permit tcp host 10.1.12.110 any eq 22
permit tcp host 10.1.12.111 any eq 22
permit tcp host 10.1.12.112 any eq 22
permit tcp host 10.2.96.225 any eq 22
deny ip any any log

!
ip access-list standard SNMP-IN
 permit 10.96.133.18
 permit 10.2.96.131
 permit 10.96.5.2
 permit 10.96.5.5
 permit 10.1.9.0 0.0.0.255
 deny any log
!
ip access-list standard NTT-SNMP-IN
permit 10.2.96.225
permit 10.2.96.226
permit 10.2.96.227
deny any log
!
logging trap notifications
logging source-interface Vlan11
logging host 10.1.9.202
!
snmp-server enable traps 
snmp-server community SPPnP_RO RO SNMP-IN
snmp-server community SPPnP_RW RW SNMP-IN
snmp-server trap-source Vlan11
snmp-server enable traps
snmp-server enable traps snmp authentication linkdown linkup coldstart warmstart 
snmp-server enable traps cluster 
snmp-server enable traps cpu threshold 
snmp-server enable traps auth-framework sec-violation 
snmp-server enable traps envmon fan shutdown supply temperature status 
snmp-server enable traps config-copy 
snmp-server enable traps config
snmp-server view nms-view 1.3.6.1.2.1 included
snmp-server view nms-view 1.3.6.1.4.1.9 included
snmp-server group nms-group v3 priv read nms-view
snmp-server user LogikMonitor-snmpv3 nms-group v3 auth sha z71l5qoj9O1bT2 priv aes 128 B0i55gPPpQWv access SNMP-IN
snmp-server host 10.96.133.18 traps version 3 priv LogikMonitor-snmpv3
snmp-server community NTTAutomat1on RW NTT-SNMP-IN
!
!
!
banner exec %
##########################################################################
#                                                                        #
#            **** Pick 'n Pay Information Systems ****                   #
#         ******* Unauthorised access prohibited *******                 #
#  In terms of Company Information Security Management Practices,        #
#  this Computer System is for authorised users ONLY.                    #
#  Individuals using this system WITH, WITHOUT or in EXCESS              #
#  of their authority are subject to having ALL their activities on      #
#  this system monitored and recorded for examination by any             #
#  authorised person. Any material so recorded may be disclosed          #
#  as appropriate.                                                       #
#  The following could also result in disciplinary actions taken:        #
#  i.   Damaging, deleting, altering or inserting data WITHOUT authority #
#  ii.  Fraudulent use of data                                           #
#  iii. Unauthorised disclosure of data                                  #
#  ANYONE USING THIS SYSTEM CONSENTS TO THESE TERMS                      #
#                                                                        #
#                                                                        #
#            ||        ||                            /\                  #
#            ||        ||                           /__\                 #
#           ||||      ||||                         /\  /\                #
#       ..:||||||:..:||||||:..                    /__\/__\               #
#                                                                        #
#      C I S C O  S Y S T E M S                  DIMENSION               #
#                                                ---- DATA               #
#                                                                        #
#                                                                        #
#                                                                        #
##########################################################################
%
!
!
banner login %
##########################################################################
#                                                                        #
#            **** Pick 'n Pay Information Systems ****                   #
#         ******* Unauthorised access prohibited *******                 #
#  In terms of Company Information Security Management Practices,        #
#  this Computer System is for authorised users ONLY.                    #
#  Individuals using this system WITH, WITHOUT or in EXCESS              #
#  of their authority are subject to having ALL their activities on      #
#  this system monitored and recorded for examination by any             #
#  authorised person. Any material so recorded may be disclosed          #
#  as appropriate.                                                       #
#  The following could also result in disciplinary actions taken:        #
#  i.   Damaging, deleting, altering or inserting data WITHOUT authority #
#  ii.  Fraudulent use of data                                           #
#  iii. Unauthorised disclosure of data                                  #
#  ANYONE USING THIS SYSTEM CONSENTS TO THESE TERMS                      #
#                                                                        #
#                                                                        #
#            ||        ||                            /\                  #
#            ||        ||                           /__\                 #
#           ||||      ||||                         /\  /\                #
#       ..:||||||:..:||||||:..                    /__\/__\               #
#                                                                        #
#      C I S C O  S Y S T E M S                  DIMENSION               #
#                                                ---- DATA               #
#                                                                        #
#                                                                        #
#                                                                        #
##########################################################################
%
!
!
banner motd %
##########################################################################
#                                                                        #
#            **** Pick 'n Pay Information Systems ****                   #
#         ******* Unauthorised access prohibited *******                 #
#  In terms of Company Information Security Management Practices,        #
#  this Computer System is for authorised users ONLY.                    #
#  Individuals using this system WITH, WITHOUT or in EXCESS              #
#  of their authority are subject to having ALL their activities on      #
#  this system monitored and recorded for examination by any             #
#  authorised person. Any material so recorded may be disclosed          #
#  as appropriate.                                                       #
#  The following could also result in disciplinary actions taken:        #
#  i.   Damaging, deleting, altering or inserting data WITHOUT authority #
#  ii.  Fraudulent use of data                                           #
#  iii. Unauthorised disclosure of data                                  #
#  ANYONE USING THIS SYSTEM CONSENTS TO THESE TERMS                      #
#                                                                        #
#                                                                        #
#            ||        ||                            /\                  #
#            ||        ||                           /__\                 #
#           ||||      ||||                         /\  /\                #
#       ..:||||||:..:||||||:..                    /__\/__\               #
#                                                                        #
#      C I S C O  S Y S T E M S                  DIMENSION               #
#                                                ---- DATA               #
#                                                                        #
#                                                                        #
#                                                                        #
##########################################################################
%
!
!
line con 0
 session-timeout 30 
 password 7 095E430C1B0A0F
 logging synchronous

line vty 0 4
 session-timeout 30
 exec-timeout 30
 access-class VTY-ACCESS in
 password 7 095E430C1B0A0F
 logging synchronous
 transport input ssh
 transport output telnet ssh

line vty 5 15
 session-timeout 30
 exec-timeout 30
 access-class VTY-ACCESS in
 password 7 095E430C1B0A0F
 logging synchronous
 transport input ssh
 transport output telnet ssh
!
clock timezone SAST 2 0
ntp source Vlan11
ntp update-calendar
ntp server 10.1.1.8 prefer
!
no ip http server
no ip http secure-server

!
!
lldp run
cdp run
!
==========================================================================================
ACCESS INTERFACE CONFIGURATION
==========================================================================================
interface range X-Y
 switchport mode access
 switchport access vlan JJJ
 switchport voice vlan KKK
 description LLL
 mls qos trust cos
 auto qos trust
 storm-control broadcast level 10.00
 storm-control multicast level 10.00
 snmp trap mac-notification change added
 snmp trap mac-notification change removed
!MAB Radius config for new stores and stores that have been cutover  
 authentication port-control auto
 authentication event fail action next-method
 authentication event server dead action authorize vlan JJJ
 authentication event server dead action authorize voice
 authentication event server alive action reinitialize
 authentication host-mode multi-domain
 authentication order mab dot1x
 authentication priority dot1x mab
 authentication port-control auto
 authentication violation restrict
 mab
 dot1x pae authenticator
 dot1x timeout tx-period 10
 dot1x pae authenticator
 dot1x timeout tx-period 10
 authentication periodic
 authentication timer reauthenticate server

==============================================================================


#TRUNKS:
description !T_Link_to_<REMOTE_HOST_INTERFACE>
 switchport
 no spanning portfast
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan <VLAN _PERMIT>
 mls qos trust cos
 auto qos trust
 udld port
 switchport nonegotiate
!
#ACCESS-POINTS:
 description !AP_Link_to_<REMOTE_HOST_INTERFACE>
 switchport trunk native vlan 11
 switchport mode trunk
 mls qos trust cos
 auto qos trust
 spanning-tree portfast trunk
!
#INTERFACES-PORT-CHANNELS:
interface range GigabitEthernet xxx
 description !P_Link_to_<REMOTE_HOST_INTERFACE>
 switchport
 no spanning portfast
 switchport nonegotiate
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan <VLAN _PERMIT>
 mls qos trust cos
 auto qos trust
 udld port
 channel-group <ID> mode active
!
#PORT-CHANNELS:
interface Port-channel <ID>
 description !P_Link_to_<REMOTE_HOST_INTERFACE>
 switchport
 switchport nonegotiate
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan <VLAN _PERMIT> 
!
!
! 
!
snmp-server contact <SITE_CONTACT>
snmp-server location <DETAILED_LOCATION_DESRIPTION>
!
interface FastEthernet/GigabitEthernetX/Y/Z - X
spanning-tree guard root -> only on down facing ports to other switches
!

==============================================================================