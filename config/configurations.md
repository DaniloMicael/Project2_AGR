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
route-map routesA-out permit 10
match route-type local
set community 8657:1
router bgp 8657
neighbor 192.88.123.3 route-map routesA-out out
neighbor 192.88.123.3 send-community
neighbor 192.88.123.4 route-map routesA-out out
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
route-map routesA-out permit 10
match route-type local
set community 8657:2
router bgp 8657
neighbor 192.88.123.3 route-map routesA-out out
neighbor 192.88.123.3 send-community
neighbor 192.88.123.4 route-map routesA-out out
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

interface Tunnel 1
ip unnumbered FastEthernet0/0                                     
tunnel destination 192.88.124.2
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 2000
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng path-option 1 dynamic

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

interface Tunnel 1
ip unnumbered FastEthernet0/0                                     
tunnel destination 192.88.124.6
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 2000
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng path-option 1 dynamic

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

### No router CompanyALisboa e CompanyAAveiro
    
```txt
interface Tunnel 1
ip unnumbered Loopback0                                     # usar loopback ou interface? r: a mesma do id
tunnel destination 10.10.10.3                               # ip da loopback ou interface? r: de aveiro para lisboa meter destination f0/1, de lisboa para aveiro meter interface f0/0
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 2000                      # aqui a bandwidth é 2000? r: sim
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng path-option 1 dynamic
```


show mpls traffic-eng tunnels

show mpls interfaces

show mpls forwarding-table

show ip rsvp

show ip rsvp interface

show ip ospf neighbor


## CompanyALisboa#trace 192.88.124.2

```txt
Type escape sequence to abort.
Tracing the route to 192.88.124.2

  1 192.88.124.5 [MPLS: Label 17 Exp 0] 16 msec 20 msec 24 msec
  2 192.88.123.4 16 msec 24 msec 20 msec
  3 192.88.124.2 24 msec 32 msec 20 msec
```

NOTE: Ou seja, existe connectividade básica ip/mpls

## CompanyALisboa#show mpls traffic-eng tunnels  

```txt
Name: CompanyALisboa_t1                   (Tunnel1) Destination: 192.88.124.2
  Status:
    Admin: up         Oper: down   Path: not valid   Signalling: Down
    path option 1, type dynamic

  Config Parameters:
    Bandwidth: 2000     kbps (Global)  Priority: 7  7   Affinity: 0x0/0xFFFF
    Metric Type: TE (default)
    AutoRoute:  enabled   LockDown: disabled  Loadshare: 2000     bw-based
    auto-bw: disabled

  History:
    Tunnel:
      Time since created: 1 minutes, 45 seconds
    Path Option 1:
      Last Error: PCALC:: No path to destination, 192.88.125.193
```

NOTE: O tunnel está down, porque não existe caminho para o destino, why?



# ================================================ #

In this section we will try to solve the problems we have with the MPLS configuration.

# Try using network

## Porto
int f0/1
no ip ospf 1 area 0
router ospf 1
network 192.88.123.0 0.0.0.63 area 0

## Coimbra
int f0/0
no ip ospf 1 area 0
router ospf 1
network 192.88.123.0 0.0.0.63 area 0

## Lisboa
int f0/1
no ip ospf 1 area 0
int f1/0
no ip ospf 1 area 0
router ospf 1
network 192.88.123.0 0.0.0.63 area 0
network 192.88.124.4 0.0.0.3 area 0

## Aveiro
int f0/0
no ip ospf 1 area 0
int f0/1
no ip ospf 1 area 0
router ospf 1
network 192.88.123.0 0.0.0.63 area 0
network 192.88.124.0 0.0.0.3 area 0

## CompanyALisboa
int f0/0
no ip ospf 1 area 0
int f0/1
no ip ospf 1 area 0
router ospf 1
network 192.88.124.4 0.0.0.3 area 0
network 192.88.125.128 0.0.0.63 area 0

## CompanyALeiria
int f0/0
no ip ospf 1 area 0
int f0/1
no ip ospf 1 area 0
router ospf 1
network 192.88.123.0 0.0.0.63 area 0
network 192.88.125.64 0.0.0.63 area 0

## CompanyAAveiro
int f0/0
no ip ospf 1 area 0
int f0/1
no ip ospf 1 area 0
router ospf 1
network 192.88.124.0 0.0.0.3 area 0
network 192.88.125.192 0.0.0.63 area 0

```txt
CompanyALisboa#show mpls traffic-eng tunnels 

Name: CompanyALisboa_t1                   (Tunnel1) Destination: 192.88.124.2
  Status:
    Admin: up         Oper: down   Path: not valid   Signalling: Down
    path option 1, type dynamic

  Config Parameters:
    Bandwidth: 0        kbps (Global)  Priority: 7  7   Affinity: 0x0/0xFFFF
    Metric Type: TE (default)
    AutoRoute:  enabled   LockDown: disabled  Loadshare: 0        bw-based
    auto-bw: (86400/0) 0  Bandwidth Requested: 0       

  History:
    Tunnel:
      Time since created: 2 minutes, 3 seconds
    Path Option 1:
      Last Error: RSVP:: Could not add RSVP Path: Invalid route specification
```

NOTE: agora o erro é diferente, **Last Error: RSVP:: Could not add RSVP Path: Invalid route specification**

# Try using loopback interfaces

## Porto
int Lo0
ip address 10.10.10.1 255.255.255.255
router ospf 1
network 10.10.10.1 0.0.0.0 area 0
no mpls traffic-eng router-id FastEthernet0/1
mpls traffic-eng router-id Loopback0

## Lisboa
int Lo0
ip address 10.10.10.2 255.255.255.255
router ospf 1
network 10.10.10.2 0.0.0.0 area 0
no mpls traffic-eng router-id FastEthernet0/1
mpls traffic-eng router-id Loopback0

## Coimbra
int Lo0
ip address 10.10.10.3 255.255.255.255
router ospf 1
network 10.10.10.3 0.0.0.0 area 0
no mpls traffic-eng router-id FastEthernet0/0
mpls traffic-eng router-id Loopback0

## Aveiro
int Lo0
ip address 10.10.10.4 255.255.255.255
router ospf 1
network 10.10.10.4 0.0.0.0 area 0
no mpls traffic-eng router-id FastEthernet0/0
mpls traffic-eng router-id Loopback0

## CompanyALisboa
int Lo0
ip address 10.10.10.5 255.255.255.255
router ospf 1
network 10.10.10.5 0.0.0.0 area 0
no mpls traffic-eng router-id FastEthernet0/0
mpls traffic-eng router-id Loopback0

end

conf t
int Tunnel 1
no ip unnumbered FastEthernet0/0                                     
no tunnel destination 192.88.124.2
ip unnumbered Loopback0
tunnel destination 10.10.10.7

## CompanyALeiria
int Lo0
ip address 10.10.10.6 255.255.255.255
router ospf 1
network 10.10.10.6 0.0.0.0 area 0
no mpls traffic-eng router-id FastEthernet0/0
mpls traffic-eng router-id Loopback0

## CompanyAAveiro
int Lo0
ip address 10.10.10.7 255.255.255.255
router ospf 1
network 10.10.10.7 0.0.0.0 area 0
no mpls traffic-eng router-id FastEthernet0/0
mpls traffic-eng router-id Loopback0

end

conf t
int Tunnel 1
no ip unnumbered FastEthernet0/0                                     
no tunnel destination 192.88.124.6
ip unnumbered Loopback0
tunnel destination 10.10.10.5

# Change ipsec tunnel to loopback

## CompanyALeiria
no crypto isakmp key labcom address 192.122.12.1
crypto isakmp key labcom address 10.10.10.8
interface Tunnel 0
no ip unnumbered FastEthernet0/0
ip unnumbered Loopback0
no tunnel source 192.88.123.5
tunnel source 10.10.10.6
no tunnel destination 192.122.12.1
tunnel destination 10.10.10.8

## Londres
int Lo0
ip address 10.10.10.8 255.255.255.255
router bgp 4589
network 10.10.10.8 mask 255.255.255.255
end
conf t
no crypto isakmp key labcom address 192.88.123.5
crypto isakmp key labcom address 10.10.10.6
interface Tunnel 0
no ip unnumbered FastEthernet2/1
ip unnumbered Loopback0
no tunnel source 192.122.12.1
tunnel source 10.10.10.8
no tunnel destination 192.88.123.5
tunnel destination 10.10.10.6


## Presentation

DO:

```txt
clear ip ospf process
```

in CompanyA routers


To test IPSec do:

```txt
ping 100.100.100.2 source 192.88.125.65
```

in Leiria