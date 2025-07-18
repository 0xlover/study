# Subnetting
## Samples
* 0.0.0.0/0 the intere internet
* 16.0.0.0/8 which is 16.0.0.0 to 16.255.255.255 which is a class A network
* 176.16.0.0/16 which is 176.16.0.0 to 176.16.255.255 which is a class B network
* 192.168.0.0/24 which is 192.168.0.0 to 192.168.0.255 which which is a class C network
* 127.0.0.0/30 which is 127.0.0.0 to 127.0.0.3 which is another class C network
* 127.0.0.1/32 which is just 127.0.0.1 a single ip address and a network C class network
* 3.0.0.0/3 which is massive and goes from 0.0.0.1 to 63.255.255.255 note that 256/2=128/2=6 which is why /3 has those 64 available

## Deviding by two
10.16.0.0/16 has to be devided in two smaller /17 networks
* one would be 10.16.0.0/17 which goes from 10.16.0.0 to 10.16.127.255
* the other will be 10.16.128.0/17 which foes from 10.16.128.0 to 10.16.255.255
this two are class C networks

## Again
10.16.0.0/17 has to be devided in two smaller /18 networks
* one would be 10.16.0.0/18 which goes from 10.16.0.0 to 10.16.63.255
* the other will be 10.16.64.0/18 which goes from 10.16.64.0 to 10.16.127.255
this two are class D networks

## One more
10.16.128.0/17 has to be devided in two smaller /18 networks
* one would be 10.16.128.0/18 which goes from 10.16.128.0 to 10.16.128.191
* the other will be 10.16.192.0/18 which goes froom 10.16.192.0 to 10.16.255.255
this two are class D networks

## Real world
192.168.0.0/24 is a class C network
* Network ID being 192.168.0.0
* Usuable ip addresses range is 192.168.0.1 to 192.168.0.254 because
* Broadcast ip address is reserved and will be 192.168.0.255
* Number of hosts or network size is 256
* Number of usuable hosts is 254
* Subnet mask converted from cidr notation(shorthand prefix notation) is 255.255.255.0
* The same in binary is 11111111.11111111.11111111.00000000
* Happens to be a private ip used for example behind a gateway nat