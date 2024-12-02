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
write
```

### Vigo

```txt
# ----- Basic Config -----

conf t
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
neighbor 192.1.1.146 remote-as 8657

network 192.90.12.0 mask 255.255.255.0

end
write
```

### Porto
```txt
# ----- Basic Config -----

conf t
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
neighbor 192.1.1.145 remote-as 4891

redistribute ospf 1

router ospf 1
redistribute bgp 8657 subnets

end
write
```

### Lisboa
```txt
# ----- Basic Config -----

conf t
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
neighbor 192.1.1.140 remote-as 4589

redistribute ospf 1

router ospf 1
redistribute bgp 8657 subnets

end
write
```

### Coimbra
```txt
conf t
int f0/0
no shut
ip ospf 1 area 0
ip address 192.88.123.3 255.255.255.192

end
write
```

### Aveiro
```txt
conf t
int f0/0
no shut
ip ospf 1 area 0
ip address 192.88.123.4 255.255.255.192
int f0/1
no shut
ip ospf 1 area 0
ip address 192.88.124.1 255.255.255.252

end
write
```

### CompanyALisboa
```txt
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
write
```

### CompanyALeiria
```txt
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
write
```

### CompanyAAveiro
```txt
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
write
```
