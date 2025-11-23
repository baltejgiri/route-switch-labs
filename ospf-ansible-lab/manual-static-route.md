Static Routes added to each network node in the lab.
# Router CSR1


```plaintext
!
configure terminal
!
ip route 0.0.0.0 0.0.0.0 172.16.105.72
ip route 1.2.3.0 255.255.255.0 GigabitEthernet0/1 111
ip route 3.3.3.0 255.255.255.0 192.168.1.3 111
ip route 3.3.3.0 255.255.255.0 1.2.3.2 111
ip route 11.11.11.0 255.255.255.0 GigabitEthernet0/0 111
ip route 21.21.21.0 255.255.255.0 GigabitEthernet0/2 111
ip route 22.22.22.0 255.255.255.0 21.21.21.2 111
ip route 22.22.22.0 255.255.255.0 11.11.11.1 111
ip route 112.112.112.0 255.255.255.0 11.11.11.1 111
ip route 112.112.112.0 255.255.255.0 21.21.21.2 111
ip route 122.122.122.0 255.255.255.0 11.11.11.1 111
ip route 122.122.122.0 255.255.255.0 21.21.21.2 111
ip route 172.18.1.0 255.255.255.0 192.168.1.3 111
ip route 172.18.1.0 255.255.255.0 1.2.3.2 111
ip route 192.168.1.0 255.255.255.0 GigabitEthernet0/3 111
!
end
!
show ip route static
!
!
write memory
!
```

### Router CSR2

Static route enteries are added on the router CSR2. Administrative value of all networks is set 1 digit higher than the OSPF administrative value (by default 110) expect network 172.16.105.0/24 is kept with AD value of 1.

```plaintext
ip route 11.11.11.0 255.255.255.0 122.122.122.1 111
ip route 11.11.11.0 255.255.255.0 22.22.22.2 111
ip route 21.21.21.0 255.255.255.0 122.122.122.1 111
ip route 21.21.21.0 255.255.255.0 22.22.22.2 111
ip route 112.112.112.0 255.255.255.0 122.122.122.1 111
ip route 112.112.112.0 255.255.255.0 22.22.22.2 111
ip route 172.16.105.0 255.255.255.0 22.22.22.2
ip route 172.16.105.0 255.255.255.0 122.122.122.1
```

### Router 1
