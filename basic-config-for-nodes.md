# OSPF Multiarea with Route redistribute Lab

## Basic configuration for nodes used in the lab.
This lab demonstrate the INE youtube live stream video where Keith Bogart provides the design of the OSPF lab. Lab spends time on the Understanding OSPF LSAs.

### Diagram

OSPF diagram is created using tool draw.io. Topology diagram shows two OSPF areas, Area 0 and Area 51 as well as route redistribution to pass traffic between OSPF and EIGRP.

Lab has two CSR routers, two routers are part of area 51 and three layer 3 switches running OSPF.

[Lab Topology](./images/INE_OSPF_MultiArea_MutliRouting_Protocols.drawio.pdf)

### Configuration

#### Border Area - First CSR Router - CSR1

```plaintext
!
enable
!
configure terminal
!
hostname CSR1
ip domain name ptn.internal
crypto key generate rsa modulus 2048
username admin privilege 15 secret Admin@123
line vty 0 4
 login local
 transport input ssh
 logging synchronous
!
exit
!
service password-encryption
ip ssh version 2
!
!Create OSPF router-id
router ospf 1
router-id 255.255.0.1
!   
interface GigabitEthernet0/0
 description uplink_to_R1
 ip address 11.11.11.11 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description uplink_to_SW2
 ip address 1.2.3.11 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/2
 description uplink_to_SW2
 ip address 21.21.21.11 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/3
 description uplink_to_SW2
 ip address 192.168.1.11 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/4
 description uplink_to_eve-ng_en0
 ip address 172.16.105.15 255.255.255.0
 no shutdown
!
end
!
write memory
!

```
#### Border Area -  Second CSR Router - CSR2

```plaintext
!
enable
!
configure terminal
!
hostname CSR2
ip domain name ptn.internal
crypto key generate rsa modulus 2048
username admin privilege 15 secret Admin@123
line vty 0 4
 login local
 transport input ssh
 logging synchronous
!
exit
!
service password-encryption
ip ssh version 2
!   
interface GigabitEthernet0/0
 description uplink_to_R1
 ip address 122.122.122.22 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description uplink_to_R2
 ip address 22.22.22.22 255.255.255.0
 no shutdown
!
end
!
write memory
!

```

#### Area 51 Router 1 - R1

```plaintext
!
enable
!
configure terminal
!
hostname Area51-R1
ip domain name ptn.internal
crypto key generate rsa modulus 2048
username admin privilege 15 secret Admin@123
line vty 0 4
 login local
 transport input ssh
 logging synchronous
!
exit
!
service password-encryption
ip ssh version 2
~
interface GigabitEthernet0/0
 description uplink_to_R2
 ip address 112.112.112.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description uplink_to_CSR2
 ip address 122.122.122.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/2
 description uplink_to_CSR1
 ip address 11.11.11.1 255.255.255.0
 no shutdown
!
end
!
write memory
!
```

#### Area 51 Router 2 - R2

```plaintext
!
enable
!
configure terminal
!
hostname Area51-R2
ip domain name ptn.internal
crypto key generate rsa modulus 2048
username admin privilege 15 secret Admin@123
line vty 0 4
 login local
 transport input ssh
 logging synchronous
!
exit
!
service password-encryption
ip ssh version 2
~
interface GigabitEthernet0/0
 description uplink_to_R1
 ip address 112.112.112.2 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description uplink_to_CSR1
 ip address 21.21.21.2 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/2
 description uplink_to_CSR2
 ip address 22.22.22.2 255.255.255.0
 no shutdown
!
end
!
write memory
!
```

#### Area 0 - Switch 3 - SW3

```plaintext
!
enable
!
configure terminal
!
hostname Area0-SW3
ip domain name ptn.internal
crypto key generate rsa modulus 2048
username admin privilege 15 secret Admin@123
line vty 0 4
 login local
 transport input ssh
 logging synchronous
!
exit
!
service password-encryption
ip ssh version 2
!
interface GigabitEthernet1/0
 no switchport
 description uplink_to_CSR1
 ip address 192.168.1.3 255.255.255.0
 no shutdown
!
interface GigabitEthernet1/2
 no switchport
 description uplink_to_AREA0-SW2
 ip address 3.3.3.3 255.255.255.0
 no shutdown
 end
!
write memory
!

```

#### Area 0 - Switch 2 - SW2

```plaintext
!
enable
!
configure terminal
!
hostname Area0-SW2
ip domain name ptn.internal
crypto key generate rsa modulus 2048
username admin privilege 15 secret Admin@123
line vty 0 4
 login local
 transport input ssh
 logging synchronous
!
exit
!
service password-encryption
ip ssh version 2
!
interface GigabitEthernet0/0
 no switchport
 description uplink_to_CSR1
 ip address 1.2.3.2 255.255.255.0
 no shutdown
!
interface GigabitEthernet1/3
 no switchport
 description uplink_to_Area0-SW3
 ip address 3.3.3.2 255.255.255.0
 no shutdown
write memory
!
interface GigabitEthernet1/1
 no switchport
 description uplink_to_EIGRP-SW1
 ip address 178.18.1.2 255.255.255.0
 no shutdown
!
end
!
write memory
!

```

#### EIGRP - Switch 1 - SW1

```plaintext
!
enable
!
configure terminal
!
hostname EIGRP-SW1
ip domain name ptn.internal
crypto key generate rsa modulus 2048
username admin privilege 15 secret Admin@123
line vty 0 4
 login local
 transport input ssh
 logging synchronous
!
exit
!
service password-encryption
ip ssh version 2
!
interface GigabitEthernet1/1
 no switchport
 description uplink_to_AREA0-SW2
 ip address 1.2.3.2 255.255.255.0
 no shutdown
!
end
!
write memory
!

```