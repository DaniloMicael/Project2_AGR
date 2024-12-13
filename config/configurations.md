# Second Project AGR

## Network Configuration

### PC1
```txt
ip 100.100.100.1/24 100.100.100.2
save
```

### PC2
```txt
ip 192.122.12.2/25 192.122.12.1
save
```

### PC3
```txt
ip 192.88.125.130/26 192.88.125.129
save
```

### PC4
```txt
ip 192.88.125.66/26 192.88.125.65
save
```

### PC5
```txt
ip 192.88.125.194/26 192.88.125.193
save
```

### PC6
```txt
ip 192.90.12.2/24 192.90.12.1
save
```

### Londres

```txt
# ----- Basic Config -----

conf t
int f0/0
no shut
ip address 100.100.100.2 255.255.255.0
int f0/1
no shut
ip address 192.1.1.137 255.255.255.252
int f1/0
no shut
ip address 192.1.1.133 255.255.255.252
int f2/0
no shut
no switchport
ip address 192.1.1.141 255.255.255.252
int f2/1
no shut
no switchport
ip address 192.122.12.1 255.255.255.128

# ----- BGP -----

router bgp 4589
neighbor 192.1.1.138 remote-as 4891
neighbor 192.1.1.134 remote-as 8657
neighbor 192.1.1.142 remote-as 8657

network 100.100.100.0 mask 255.255.255.0
network 192.122.12.0 mask 255.255.255.128

end

# ----- IPsec -----

conf t

crypto isakmp policy 30
authentication pre-share
crypto isakmp key labcom address 192.88.123.5
crypto ipsec transform-set authT ah-sha-hmac
crypto ipsec transform-set cipherT esp-des
crypto ipsec transform-set auth_ciphT ah-sha-hmac esp-des
crypto ipsec profile ARipsec
set transform-set auth_ciphT authT cipherT

interface Tunnel 0
ip unnumbered FastEthernet2/1
tunnel source 192.122.12.1
tunnel destination 192.88.123.5
tunnel mode ipsec ipv4
tunnel protection ipsec profile ARipsec
exit

ip route 192.88.125.65 255.255.255.255 Tunnel0

end
write
```

### Vigo

```txt
# ----- Basic Config -----

conf t

ip as-path access-list 1 permit ^$
route-map routes-out
match as-path 1

int f0/0
no shut
ip address 192.1.1.138 255.255.255.252
int f0/1
no shut
ip address 192.90.12.1 255.255.255.0
int f1/0
no shut
ip address 192.1.1.145 255.255.255.252

# ----- BGP -----

router bgp 4891
neighbor 192.1.1.137 remote-as 4589
neighbor 192.1.1.137 route-map routes-out out
neighbor 192.1.1.146 remote-as 8657
neighbor 192.1.1.146 route-map routes-out out

network 192.90.12.0 mask 255.255.255.0

end
write
```

### Porto
```txt
# ----- Basic Config -----

conf t

ip as-path access-list 1 permit ^$
route-map routes-out
match as-path 1

int f0/0
no shut
ip address 192.1.1.134 255.255.255.252
int f0/1
no shut
ip ospf 1 area 0
ip address 192.88.123.1 255.255.255.192
int f1/0
no shut
ip address 192.1.1.146 255.255.255.252

# ----- BGP -----

router bgp 8657
neighbor 192.1.1.133 remote-as 4589
neighbor 192.1.1.133 route-map routes-out out
neighbor 192.1.1.145 remote-as 4891
neighbor 192.1.1.145 route-map routes-out out
network 192.88.123.0 mask 255.255.255.192

neighbor 192.88.123.2 remote-as 8657
neighbor 192.88.123.3 remote-as 8657
neighbor 192.88.123.4 remote-as 8657
neighbor 192.88.123.2 next-hop-self
neighbor 192.88.123.3 next-hop-self
neighbor 192.88.123.4 next-hop-self

redistribute ospf 1

router ospf 1
redistribute bgp 8657 subnets

end

# ----- 1º Bullet Point (Communities) -----

conf t
ip bgp-community new-format
route-map routes-out permit 10
match route-type local
set community 8657:1
router bgp 8657
neighbor 192.88.123.3 route-map routes-out out
neighbor 192.88.123.3 send-community
neighbor 192.88.123.4 route-map routes-out out
neighbor 192.88.123.4 send-community

end

# ----- MPLS config -----

conf t
ip cef
mpls ip
mpls traffic-eng tunnels
int f0/1
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000

router ospf 1
mpls traffic-eng area 0
mpls traffic-eng router-id FastEthernet0/1

end
write
```

### Lisboa
```txt
# ----- Basic Config -----

conf t

ip as-path access-list 1 permit ^$
route-map routes-out
match as-path 1

int f0/0
no shut
ip address 192.1.1.142 255.255.255.252
int f0/1
no shut
ip ospf 1 area 0
ip address 192.88.123.2 255.255.255.192
int f1/0
no shut
ip ospf 1 area 0
ip address 192.88.124.5 255.255.255.252

# ----- BGP -----

router bgp 8657
neighbor 192.1.1.141 remote-as 4589
neighbor 192.1.1.141 route-map routes-out out

neighbor 192.88.123.1 remote-as 8657
neighbor 192.88.123.3 remote-as 8657
neighbor 192.88.123.4 remote-as 8657
neighbor 192.88.123.1 next-hop-self
neighbor 192.88.123.3 next-hop-self
neighbor 192.88.123.4 next-hop-self
network 192.88.123.0 mask 255.255.255.192
network 192.88.124.4 mask 255.255.255.252

redistribute ospf 1

router ospf 1
redistribute bgp 8657 subnets

end

# ----- 1º Bullet Point (Communities) -----

conf t
ip bgp-community new-format
route-map routes-out permit 10
match route-type local
set community 8657:2
router bgp 8657
neighbor 192.88.123.3 route-map routes-out out
neighbor 192.88.123.3 send-community
neighbor 192.88.123.4 route-map routes-out out
neighbor 192.88.123.4 send-community

end

# ----- MPLS config -----

conf t
ip cef
mpls ip
mpls traffic-eng tunnels    
int f0/1
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000
int f1/0
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000

router ospf 1
mpls traffic-eng area 0
mpls traffic-eng router-id FastEthernet0/1
exit

interface Tunnel 1
ip unnumbered FastEthernet0/1                                    
tunnel destination 192.88.123.4
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 2000
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng path-option 1 dynamic

end
write
```

### Coimbra
```txt
# ----- Basic Config -----

conf t
int f0/0
no shut
ip ospf 1 area 0
ip address 192.88.123.3 255.255.255.192

# ----- BGP -----

router bgp 8657
neighbor 192.88.123.1 remote-as 8657
neighbor 192.88.123.2 remote-as 8657
neighbor 192.88.123.4 remote-as 8657

end

# ----- 1º Bullet Point (Communities) -----

conf t
ip bgp-community new-format
ip community-list 1 permit 8657:1
ip community-list 2 permit 8657:2
route-map routes-in permit 10
match community 1
set local-preference 100
route-map routes-in permit 20
match community 2
set local-preference 200
router bgp 8657
neighbor 192.88.123.1 route-map routes-in in
neighbor 192.88.123.2 route-map routes-in in

end

# ----- MPLS config -----

conf t
ip cef
mpls ip
mpls traffic-eng tunnels
int f0/0
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000

router ospf 1
mpls traffic-eng area 0
mpls traffic-eng router-id FastEthernet0/0

end
write
```

### Aveiro
```txt
# ----- Basic Config -----

conf t
int f0/0
no shut
ip ospf 1 area 0
ip address 192.88.123.4 255.255.255.192
int f0/1
no shut
ip ospf 1 area 0
ip address 192.88.124.1 255.255.255.252

# ----- BGP -----

router bgp 8657
neighbor 192.88.123.1 remote-as 8657
neighbor 192.88.123.2 remote-as 8657
neighbor 192.88.123.3 remote-as 8657

end

# ----- 1º Bullet Point (Communities) -----

conf t
ip bgp-community new-format
ip community-list 1 permit 8657:1
ip community-list 2 permit 8657:2
route-map routes-in permit 10
match community 1
set local-preference 200
route-map routes-in permit 20
match community 2
set local-preference 100
router bgp 8657
neighbor 192.88.123.1 route-map routes-in in
neighbor 192.88.123.2 route-map routes-in in

end

# ----- MPLS config -----

conf t
ip cef
mpls ip
mpls traffic-eng tunnels
int f0/0
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000
int f0/1
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000

router ospf 1
mpls traffic-eng area 0
mpls traffic-eng router-id FastEthernet0/0
exit

interface Tunnel 1
ip unnumbered FastEthernet0/0                                     
tunnel destination 192.88.123.2
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 2000
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng path-option 1 dynamic

end
write
```

### CompanyALisboa
```txt
# ----- Basic Config -----

conf t
int f0/0
no shut
ip ospf 1 area 0
ip address 192.88.124.6 255.255.255.252
int f0/1
no shut
ip ospf 1 area 0
ip address 192.88.125.129 255.255.255.192

end

# ----- MPLS config -----

conf t
ip cef
mpls ip
mpls traffic-eng tunnels
int f0/0
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000
int f0/1
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000

router ospf 1
mpls traffic-eng area 0
mpls traffic-eng router-id FastEthernet0/0

end
write
```

### CompanyALeiria
```txt
# ----- Basic Config -----

conf t
int f0/0
no shut
ip ospf 1 area 0
ip address 192.88.123.5 255.255.255.192
int f0/1
no shut
ip ospf 1 area 0
ip address 192.88.125.65 255.255.255.192

end

# ----- IPsec -----

conf t

crypto isakmp policy 30
authentication pre-share
crypto isakmp key labcom address 192.122.12.1
crypto ipsec transform-set authT ah-sha-hmac
crypto ipsec transform-set cipherT esp-des
crypto ipsec transform-set auth_ciphT ah-sha-hmac esp-des
crypto ipsec profile ARipsec
set transform-set auth_ciphT authT cipherT

interface Tunnel 0
ip unnumbered FastEthernet0/0
tunnel source 192.88.123.5
tunnel destination 192.122.12.1
tunnel mode ipsec ipv4
tunnel protection ipsec profile ARipsec
exit

ip route 100.100.100.2 255.255.255.255 Tunnel0

end

# ----- MPLS config -----

conf t
ip cef
mpls ip
mpls traffic-eng tunnels
int f0/0
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000
int f0/1
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000

router ospf 1
mpls traffic-eng area 0
mpls traffic-eng router-id FastEthernet0/0

end
write
```

### CompanyAAveiro
```txt
# ----- Basic Config -----

conf t
int f0/0
no shut
ip ospf 1 area 0
ip address 192.88.124.2 255.255.255.252
int f0/1
no shut
ip ospf 1 area 0
ip address 192.88.125.193 255.255.255.192

end

# ----- MPLS config -----

conf t
ip cef
mpls ip
mpls traffic-eng tunnels
int f0/0
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000
int f0/1
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000

router ospf 1
mpls traffic-eng area 0
mpls traffic-eng router-id FastEthernet0/0

end
write
```


## MPLS Doubts

### Em todos os routers

```txt
# ----- MPLS config -----

conf t
ip cef
mpls ip
mpls traffic-eng tunnels
int f0/0
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000
int f0/1
mpls ip
mpls traffic-eng tunnels
ip rsvp bandwidth 2000 2000
```

- em que interfaces tenho de fazer isto? R: podemos meter em todos os routers, todas as interfaces, menos as que vao pra outro as

```txt
router ospf 1
mpls traffic-eng area 0
mpls traffic-eng router-id Loopback 0
```

- preciso de um loopback em todos os routers? ou posso usar uma interface qualquer? R: qualquer uma fisica

### No router Lisboa e Aveiro
    
```txt
interface Tunnel 1
ip unnumbered Loopback0                                     # usar loopback ou interface? r: a mesma do id
tunnel destination 10.10.10.3                               # ip da loopback ou interface? r: de aveiro para lisboa meter destination f0/1, de lisboa para aveiro meter interface f0/0
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 2000                      # aqui a bandwidth é 2000? r: sim
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng path-option 1 dynamic
```