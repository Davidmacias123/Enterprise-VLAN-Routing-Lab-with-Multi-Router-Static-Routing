# Enterprise-VLAN-Routing-Lab-with-Multi-Router-Static-Routing
Designed and implemented a multi-router enterprise network using VLAN segmentation and router-on-a-stick. Static routing over a point-to-point link enables full bidirectional communication between hosts across multiple VLANs and routed networks.


we will be creating vlans using our switch to connect with our pcs with the vlans particular ip address. we will be creating two networks, and those networks would be able to communicate one another such as roouter on a stick, using subinterfaces and all that stuff. for our vlans so if our vlan is 10, our ip would be 192.168.10.x. the first thing that we will do is click on our switch we will type enable configure terminal and create two vlans. our two vlans would be vlan 10 and vlan 20 and then we will name our vlans. for me i chose marketing and hr. after we completed that we will assign access ports to our vlan 10 so we can connect 2 out 4 of our pcs. for our first vlan it would be vlan 10 we will type interface range fa0/1 - 2 switchport mode access, switchport acccess vlan 10, and no shutdown, and then we will exit. after that we will assign access ports to vlan 20 to connect our other two pcs. we will type interface range fa0/3 - 4 switchport mode access, switchpoort access vlan 20, no shutdown, and exit. afte that we will need to configure our trunk port to router. we will be doing this on our switch as well. we will type interface gigabitethernet0/1 thats the port called on our router. we will type switchport mode trunk, no shutdown, exit, end and write memory so it is recorded and saved. these our the commands for this part would look like

<img width="1913" height="987" alt="image" src="https://github.com/user-attachments/assets/04644d83-0363-4985-84fe-d428c497f917" />



-----


# Router configurations command put it on the router

afteer this we will start to configure our router. the commands we will type is enable, configure temrina, interface gigabitethernet0/0/0 and we will not put an ip address yet, until later, and then shutdown and then exit. now we will create our subinterfaces for vlan 10 and vlan 20 on the router!!! so fun. we will type interface gigabitethernet0/0/0.10, encapsulation dot1Q 10, ip address 192.168.10.1 255.255.255.0 and then we will exit. now we will do the exact same thing for the other subinterface. we will type interface gigabitethernet0/0/0.20 encapsulation dot1Q 20, ip address 192.168.20.1 255.255.255.0 and then we will exit. after we did this, NOW we will do dhcp exclusions and reserver gateway and some static space. our first two commands that we will type is ip dhcp excluded-address 192.168.10.1 192.168.10.10 and then we will type for our second command which would be ip dhcp excluded-address 192.168.20.1 192.168.20.10. now we will do dhcp pool for vlan 10. we will type dhcp pool vlan 10 network 192.168.10.0 255.255.255.0 default router 192.168.10.1 dns server 8.8.8.8 and then exit. after that we will have to also do dhcp pool for our vlan 20. we will tyep ip dhcp pool vlan20, network 192.168.20.0 255.255.255.0 default-router 192.168.20.1 dns-server 8.8.8.8 adn then exit, end and then write memory to save it. the reason why we need to do dhcp twice for both of our vlans so our computers can get dhcp address on their vlans that we created, that is why this is really super important. 

<img width="1917" height="977" alt="image" src="https://github.com/user-attachments/assets/3b57ad04-99f8-4bc3-9791-5527d4d7ecf3" />

<img width="387" height="331" alt="image" src="https://github.com/user-attachments/assets/87ac6168-1c4c-4fb1-990e-dc79aa9c7faf" />


## Router configurations command put it on the router for our second network
no we will do the exact same thing for our other network, these are the commands for the other network over everything #Commands for your pcs to be on that exact VLANS you created!!!

SWITCH COMMANDS

#Creating VLANS

enable
configure terminal

vlan 10
 name VLAN10
exit

vlan 20
 name VLAN20
exit


#Assign access ports to VLAN 10(PC0 +PC1)

interface range fa0/1 - 2
OR
interface fa/1 (going to do this for every single port if the range doesn't work)
 switchport mode access
 switchport access vlan 10
 no shutdown
exit


#Assign Access Ports to VLAN 20 (PC3 + PC2)

interface range fa0/3 - 4
 switchport mode access
 switchport access vlan 20
 no shutdown
exit



#4) Configure Trunk Port to Router 
#YOU DO THIS ON THE SWITCH!!!!!!

interface gig0/1
 switchport mode trunk
 no shutdown
exit
end
write memory

a screenshot of what we just did on the switch the switch commands on the CLI:

<img width="1918" height="966" alt="image" src="https://github.com/user-attachments/assets/5754d13e-e948-4c72-a08d-04ae908128ae" />


#ROUTER CONFIGURATIONS COMMANDS PUT IT ON THE ROUTER

enable
configure terminal

interface gig0/0/0
 no ip address
 no shutdown
exit

#2) Create Subinterfaces for VLAN 10 and VLAN 20 ON THE ROUTER!!!


interface gig0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.100.1 255.255.255.0
exit

interface gig0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.200.1 255.255.255.0
exit

#3) DHCP Exclusions (Reserve gateway + some static space)

ip dhcp excluded-address 192.168.100.1 192.168.100.10
ip dhcp excluded-address 192.168.200.1 192.168.200.10


#4) DHCP Pool for VLAN 10

ip dhcp pool VLAN10
 network 192.168.100.0 255.255.255.0
 default-router 192.168.100.1
 dns-server 8.8.8.8
exit

#5) DHCP Pool for VLAN 20

ip dhcp pool VLAN20
 network 192.168.200.0 255.255.255.0
 default-router 192.168.200.1
 dns-server 8.8.8.8
exit
end
write memory

screenshots of our router configurations of what we just did: and our diagram of that part of the network:

<img width="1918" height="971" alt="image" src="https://github.com/user-attachments/assets/8c3d8e60-bfe9-42e7-969f-2e9be2fd6e7d" />

<img width="1915" height="972" alt="image" src="https://github.com/user-attachments/assets/1f872a88-aa0f-43e7-85d7-0a8783accd06" />



<img width="332" height="257" alt="image" src="https://github.com/user-attachments/assets/a362c7a1-02f5-4618-adc0-a4979bfdd28a" />


----

now we active dhcp on our pc on our first network:


<img width="1915" height="1017" alt="image" src="https://github.com/user-attachments/assets/3ca3803c-d75d-4b1b-9505-ef629b852a46" />



<img width="1908" height="1011" alt="image" src="https://github.com/user-attachments/assets/0d1d2a69-2b66-4d42-a845-c4419a3c6035" />




----
now we activate dhcp on our pcs for our second network:

<img width="1917" height="1002" alt="image" src="https://github.com/user-attachments/assets/792b927e-f024-464b-8452-43e1fd010d8a" />
<img width="1918" height="978" alt="image" src="https://github.com/user-attachments/assets/c7f4fb32-7224-494e-9e1e-29b938cd87fe" />








-----


# after you completed that now we will connect both our router together in order for our pcs to talk to one another on the two networks


<img width="785" height="388" alt="image" src="https://github.com/user-attachments/assets/eadb0f6d-5dbc-415e-9f76-fb251af3da2c" />




----

Now we will need to go to our first router on our first network adn weill add an ip on the interface gigabitethernet0/0/0. the ip address that i chose is 192.168.30.1 255.255.255.0, and now we will make a second ip that would be so the router to router are able to link from one naother we will type this on our interface gigabitethernet0/0/1 ip address 10.0.0.1 255.255.255.252 and no shutdown and exit. now we will static routes to our router 2 LAN. the commands that we use were ip route 192.168.100.0 255.255.255.0 10.0.0.2
ip route 192.168.200.0 255.255.255.0 10.0.0.2
so we can communicate to the other network vlans thast why we putting those commands, and then end and then write memory. 


<img width="1908" height="981" alt="image" src="https://github.com/user-attachments/assets/90e7470c-b65b-4164-8a65-89e67b0abbb3" />

-------

# now we will do the exact same thing on the other router of the second network
we will enable configure terminal interface gigabitethernet0/0/0 the ip address that we will use is 192.168.40.1 255.255.255.0 and no shutdown and exit. then we will router to router link with our first network. we will type interface gigabitethernet0/0/1 ip address 10.0.0.2 255.255.255.252 no shutdown and exit. now we will static route to our router 1 our first network LAN. we will type these two commands
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1
and then end and then write memory.

it should look like this.


<img width="1917" height="981" alt="image" src="https://github.com/user-attachments/assets/a7999a33-f2f4-4675-a316-37af9b9db9b6" />


----

this is how our packet tracer looks now

<img width="1918" height="1022" alt="image" src="https://github.com/user-attachments/assets/cae2ac0d-f13c-4907-9c4e-255d452e60b2" />



now we will ping our pcs one another:

<img width="1906" height="971" alt="image" src="https://github.com/user-attachments/assets/87d457d6-e828-46d2-b92f-af74e2fc513f" />



<img width="1918" height="1017" alt="image" src="https://github.com/user-attachments/assets/e8b66e5a-284f-41d5-83a5-f3d55129230b" />




that will conclude our project.
