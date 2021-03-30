# Campus-Network-Design
It consists of a West Building (WB), an East Building (EB), and a Data Center (DC), all connected through a frame relay switch. The Data Center is connected to the ISP to get to the simulated Internet (it's just a 4.2.2.0/24 network). All the IP subnets are indicated in a little legend key. Of course, you may wish to use whatever IP scheme you want.


The Steps:
West Building
1 Switches: IP addresses
Configure IP addresses for the switches (which will be in VLAN 1, subnet 172.16.1.0/24). Their default gateway will be 172.16.1.254 (I used 3560 switches for the IP phones, they don't have the default gateway command; you may swap them for the 2960 switch). Verify by ping.
2 Switches: VTP and VLANS
Configure the ports connected between the switches to trunk ports. Create VLANs 30 and 40 on one switch, name them DATA1 and DATA2 respectively. Configure the VTP domain "WB" on the switch (and optionally VTP password "1234"; you'll have to configure the password on the other switches). Assign the ports that will have PCs attached to the VLANs (check topology image). Verify that all VLANs have replicated by show vlan on the other switches.
3 Switches: STP
The top switch will be the root switch. As practice, try to figure out which port (on the other switches of course) will become blocked. Configure the top switch to be the root switch for all VLANs (1, 30 and 40). Optional: Configure ports that are/will be connected to PCs/routers to portfast. Verify by show spanning-tree on all switches that the top switch is the root.
4 Routers: Router-on-a-stick
Configure the (CME labeled) router's Fast-Ethernet port on VLAN 1, and two subinterfaces for each of VLANs 30 (subnet 172.16.30.0/24) and 40 (subnet 172.16.40.0/24); don't forget to tell them they'll be getting VLAN information from the specific VLANs.
5 Routers: Gateway Router/DHCP
Configure it's Fast-Ethernet port on the VLAN 40 subnet; don't forget to configure the port on the switch on VLAN 40. Configure DHCP pools for VLANs 30 and 40 (I'm not sure if this was in the CCNA syllabus, but it's pretty cool and easy. Otherwise, forget about this and just statically assign IPs to the PCs in their respective VLAN subnets. If you do however go with the DHCP,
you'll have to configure one extra command on the router-on-a-stick router for the VLAN 30 PCs to obtain IPs. As a challenge, I'll leave you to find out that command on your own if you don't know it).
6 Routers: Routing protocol
I used EIGRP. You may use anything else. After configuring routing protocols on both routers, verify by show ip route on the Gateway router. You can verify by ping (for example, ping from Gateway router [VLAN 40] to one of the switches [VLAN 1]) to verify inter VLAN routing. Also, ping from a PC on one VLAN to another PC on the other VLAN.
East Building
Same thing as West Building. I chose the VTP domain "EB", VLANs 35 and 45 instead, different IP subnets; refer to the topology image. Use the same routing protocol.
Data Center
This is pretty straight forward. There is no VTP here, no STP, and you don't even have to have DHCP (I did). Configure static IPs. Refer to the image. Use the same routing protocol. Configure the serial interface that will connect to the ISP with the IP address in the image (as you may have noticed, I got tired of typing the IPs haha).
ISP
Configure both serial ports with IPs. Do NOT configure a routing protocol. Instead, configure a static route to network 4.2.2.0/24 pointing to the next router (which will be our network on the Internet). This is because we don't want this ISP router to know of ANY of the IPs in the buildings/data center. This will simulate the "internet" experience; so that if it receives a ping from any device, it will not be able to reply back because it doesn't have a route path to the device's IP...this is where NAT will come into play later. At this point, go back to the Data Center router and configure a default static route to point to the ISP router. Then redistribute that route to the other routers. Don't do NAT yet.
Internet
Also, straight forward; configure static IPs to the servers and all that. But again, do NOT configure a routing protocol on the router. Instead, just configure a static route to the 68.110.171.132/30 network going through the ISP router (you could also use NAT, but I didn't).
Frame Relay
Use point-to-point; I had difficulties with the routing protocol when I used multipoint. Subnets are indicated in the image. For the frame relay switch/cloud itself, just check out the configuration from the finished configuration, it's straight forward.
At this point, most of the configuration is done. Verify connectivity between both buildings and Data Center by ping (and Web Browser on the PCs if you still have that HTTP server in the Data Center).
NAT and ACLs
Verify that any device will NOT be able to reach the internet (4.2.2.0/24), because we don't have NAT configured. Now, configure NAT on the Data Center router. My ACL was configured as follows: deny VLAN 40 subnet in WB (except routers/admin laptop), deny VLAN 35 subnet in EB (except routers/admin laptop), and permit all the other subnets (I did not permit any, I permit the specific subnets for practice). Unfortunately, I did not do any other ACL practice; I was going to create an ACL in each building to only allow admin laptops to telnet to the routers/switches
