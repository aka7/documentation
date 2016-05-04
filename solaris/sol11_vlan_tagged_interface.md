'Solaris 11 interface config

''Assuming that the physical interfaces net3 and net7 are patched and their ports support a given vlan (1205 in this case) we can do the following:

''Free the interfaces from any original configuration
ipadm delete-ip net3
ipadm delete-ip net7

''Rename the interfaces if necessary
dladm rename-link net3 int0
dladm rename-link net7 int1

''Create the link aggregation:    
dladm create-aggr -m trunk -P L4 -L active -T long -l int0 -l int1 aggr0

''Create a vlan interface using the aggregation:
dladm create-vlan -l aggr0 -v 254 aggr0_v254

''Enable the new interface
ipadm create-ip aggr0_v254

''Assign an IP address
ipadm create-addr -a 10.234.76.12/23 aggr0_v254

''adding static route:
route -p add -net 10.234.12.0/23 -gateway 10.234.76.1
