#Solaris Global Zone

Useful Solaris 11 to enable bonding, with LCAP in pvlan switch configs

The Global Zone (GZ) will have the bare minimum install of the OS.

Rename the network interface links to represent their function. If the links are already configured with addresses you will need to unconfigure them before renaming.

Assuming:

```` 
Old Name	Purpose	New Name
NET0	Front End 1	FE0
NET1	Management	MGMT0
NET3	Front End 2	FE1
dladm rename-link net0 fe0
 
```` 

Create an LACP bonded interface on the front end links. The fex ports will need to configured with LACP too.

```` 
dladm create-aggr -L active -T short -l fe0 -l fe1 aggr0
```` 
If the global zone requires a front end address it will need to VLAN tag the traffic so create a VLAN interface for it

```` 
dladm create-vlan -l aggr0 -v 250 vlan250
ipadm create-ip vlan250
ipadm create-addr -a local=10.10.4.14/23 vlan250
```` 
If the GZ needs a management interface it does not need to VLAN tag the traffic so you can create the address on the mgmt0 link.

```` 
ipadm create-ip mgmt0
ipadm create-addr -a=local 10.10.2.14/23 mgmt0
```` 
 

For every interface on the server enable MAC spoofing and IP address spoofing protection.

```` 
dladm set-linkprop -p protection=mac-nospoof,ip-nospoof aggr0
dladm set-linkprop -p protection=mac-nospoof,ip-nospoof fe0
dladm set-linkprop -p protection=mac-nospoof,ip-nospoof fe1
dladm set-linkprop -p protection=mac-nospoof,ip-nospoof mgmt0
```` 
 

Symmetric routing should be activated with the following commands

```` 
ipadm set-prop -p hostmodel=strong ipv4
ipadm set-prop -p hostmodel=strong ipv6

```` 
LLDP is controlled by the service

```` 
svc:/network/lldp:default
root@nme1bllab:~# lldpadm set-agentprop -p basic-tlv=all,dot1-tlv=all,dot3-tlv=all,virt-tlv=all fe0
root@nme1bllab:~# lldpadm show-agentprop fe0
AGENT        PROPERTY   PERM VALUE          DEFAULT        POSSIBLE
fe0          mode       rw   both           disable        txonly,rxonly,both,
                                                           disable
fe0          basic-tlv  rw   portdesc,      none           none,portdesc,
                             sysname,sysdesc,              sysname,sysdesc,
                             syscapab                      syscapab,mgmtaddr,
                                                           all
fe0          dot1-tlv   rw   vlanname,pvid, none           none,vlanname,pvid,
                             linkaggr,evb                  linkaggr,pfc,appln,
                                                           evb,etscfg,all
fe0          dot3-tlv   rw   max-framesize  none           none,max-framesize,
                                                           all
fe0          virt-tlv   rw   vnic           none           none,vnic,all
# display LLDP information that the server is advertising to the switch
root@nme1bllab:~# lldpadm show-agent -lv fe0
                                                 Agent: fe0
                                    Chassis ID Subtype: Local(7)
                                            Chassis ID: 00063868
                                       Port ID Subtype: MacAddress(3)
                                               Port ID: 00:25:90:4d:22:87
                                      Port Description: fe0
                                          Time to Live: 121 (seconds)
                                           System Name: nme1bllab
                                    System Description: SunOS 5.11 11.1 i86pc
                                Supported Capabilities: bridge,router,station
                                  Enabled Capabilities: --
                                    Management Address: --
                                    Maximum Frame Size: 1500
                                          Port VLAN ID: 1
                                          VLAN Name/ID: --
                                   VNIC PortID/VLAN ID: --
                               Aggregation Information: Capable, Not Aggregated
                                           PFC Willing: --
                                               PFC Cap: --
                                               PFC MBC: --
                                            PFC Enable: --
                                           PFC Pending: --
                            Application(s)(ID/Sel/Pri): --
                                           ETS Willing: --
                                    ETS Configured CBS: --
                                    ETS Configured TCS: --
                                    ETS Configured PAT: --
                                    ETS Configured BAT: --
                                    ETS Configured TSA: --
                                   ETS Recommended PAT: --
                                   ETS Recommended BAT: --
                                   ETS Recommended TSA: --
                                              EVB Mode: --
                                     EVB GID (Station): --
                               EVB ReflectiveRelay REQ: --
                            EVB ReflectiveRelay Status: --
                                      EVB GID (Bridge): --
                   EVB ReflectiveRelay Capable (RRCAP): --
                   EVB ReflectiveRelay Control (RRCTR): --
                                   EVB max Retries (R): --
                     EVB Retransmission Exponent (RTE): --
EVB Remote or Local(ROL) and Resource Wait Delay (RWD): --
                         EVB Resource Wait Delay (RWD): --
 EVB Remote or Local (ROL) and Reinit Keep Alive (RKA): --
                           EVB Reinit Keep Alive (RKA): --
                              Next Packet Transmission: 25 (seconds)
 
# display information the switch is advertising to the server
root@nme1bllab:~# lldpadm show-agent -rv fe0
                                                 Agent: fe0
                                    Chassis ID Subtype: MacAddress(4)
                                            Chassis ID: 64:e9:50:00:cd:8d
                                       Port ID Subtype: Local(7)
                                               Port ID: Eth101/1/12
                                      Port Description: Ethernet101/1/12
                                          Time to Live: 120 (seconds)
                                           System Name: as0-nme.bllab
                                    System Description: Cisco Nexus Operating System (NX-OS) SoftwareTAC support: http://www.cisco.com/tac Copyright (c) 2002-2013, Cisco Systems, Inc. All rights reserved.
                                Supported Capabilities: bridge
                                  Enabled Capabilities: bridge
                                    Management Address: 10.200.43.54
                                    Maximum Frame Size: --
                                          Port VLAN ID: 1
                                          VLAN Name/ID: --
                                   VNIC PortID/VLAN ID: --
                               Aggregation Information: --
                                           PFC Willing: --
                                               PFC Cap: --
                                               PFC MBC: --
                                            PFC Enable: --
                            Application(s)(ID/Sel/Pri): --
                                           ETS Willing: --
                                    ETS Configured CBS: --
                                    ETS Configured TCS: --
                                    ETS Configured PAT: --
                                    ETS Configured BAT: --
                                    ETS Configured TSA: --
                                   ETS Recommended PAT: --
                                   ETS Recommended BAT: --
                                   ETS Recommended TSA: --
                                              EVB Mode: --
                                     EVB GID (Station): --
                               EVB ReflectiveRelay REQ: --
                            EVB ReflectiveRelay Status: --
                                      EVB GID (Bridge): --
                   EVB ReflectiveRelay Capable (RRCAP): --
                   EVB ReflectiveRelay Control (RRCTR): --
                                   EVB max Retries (R): --
                     EVB Retransmission Exponent (RTE): --
EVB Remote or Local(ROL) and Resource Wait Delay (RWD): --
                         EVB Resource Wait Delay (RWD): --
 EVB Remote or Local (ROL) and Reinit Keep Alive (RKA): --
                           EVB Reinit Keep Alive (RKA): --
                               Information Valid Until: 112 (seconds)
 
```` 

##Solaris Zone
 

For each zone, create a virtual NIC on the front end aggregate link with the correct VLAN tag. Zones should not normally need a management network

```` 
dladm create-vnic -l aggr0 -v 950 vaggr0
 
```` 

Create the zone, specify the VNIC you created, ip-type=exclusive and configure the address using the allowed-address parameter. This ensures that only the specified address(es) can be plumbed up on the interface. 

The use of VNICs ensures that zone network traffic is forced out of the server and cannot short-circuit internally. This is required for correct PVLAN isolation.
````
zonecfg:oracle1> info
zonename: somezone
zonepath: /zones/somezone
brand: solaris
autoboot: true
bootargs: 
file-mac-profile: 
pool: 
limitpriv: 
scheduling-class: 
ip-type: exclusive
hostid: 
fs-allowed: 
net:
    address not specified
    allowed-address: 10.10.132.11
    configure-allowed-address: true
    physical: vaggr0
    defrouter: 10.10.132.1
````
