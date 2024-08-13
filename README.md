# NSD-Project

### Autori
* **Valerio Crecco - matricola: 0320452**
* **Ludovico De Santis - matricola: 0320460**

### Topologia
<p align="center">

</p>

### Indice
- [AS100](#as100)
    - [R101](#r101)
    - [R102](#r102)
    - [R103](#r103)
- [AS200](#as200)
    - [R201](#r201)
    - [R202](#r202)
    - [R203](#r203)
- [Client-200](#client-200) [[AppArmor](#apparmor)]
- [AS300](#as300)
    - [R301](#r301)
    - [R302](#r302)
    - [GW300](#gw300)
- [DC Network](#dc-network)
    - [Spine 1](#spine-1) | [Spine 2](#spine-2)
    - [Leaf 1](#leaf-1) | [Leaf 2](#leaf-2)
    - [A1](#a1) | [B1](#b1) | [A2](#a2) | [B2](#b2)
- [AS400](#as400)
    - [R401](#r401)
    - [R402](#r402)
    - [Client-400](#client-400)
- [OpenVPN](#openvpn)

### AS100

>`AS100` è un AS che fornisce funzionalità di accesso ai customers: `AS200` e `AS300`.     Sono stati configurati:  
 > - eBGP peering con `AS200` e `AS300`;
 > - iBGP peering tra i border routers;
 > - OSPF;
 > - LDP/MPLS nella core network;

 - #### R101
  
  Sono state configurate le **interfacce** `eth0`, `eth1` e di loopback `lo`, abilitando **MPLS** per `eth1` e `lo`:
  
  ```shell
interface eth0
 ip address 10.12.11.2/30

interface eth1
 ip address 10.1.12.1/30
 mpls enable

interface lo
 ip address 1.1.0.1/16
 ip address 1.255.0.1/32
 mpls enable
exit
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
router ospf
 ospf router-id 1.255.0.1
 network 1.255.0.1/32 area 0
 network 10.1.12.0/30 area 0
exit
  ```
  Configurazione del protocollo **BGP**:
  
  ```shell
router bgp 100
 bgp router-id 1.255.0.1
 neighbor 1.255.0.3 remote-as 100
 neighbor 1.255.0.3 update-source 1.255.0.1
 neighbor 10.12.11.1 remote-as 200
 address-family ipv4 unicast
  network 1.1.0.0/16
  neighbor 1.255.0.3 next-hop-self
 exit-address-family
exit
  ```
  Configurazione di **MPLS/LDP**:
  
  Sono state abilitate le interfacce che possono accettare pacchetti MPLS, modificando il file `/etc/sysctl.conf`: 
  
```shell
echo 'net.mpls.conf.lo.input = 1' >> /etc/sysctl.conf
echo 'net.mpls.conf.eth1.input = 1' >> /etc/sysctl.conf
echo 'net.mpls.platform_labels = 100000' >> /etc/sysctl.conf
sysctl -p
  ```
  
  ```shell
mpls ldp
 router-id 1.255.0.1
 ordered-control
 address-family ipv4
  discovery transport-address 1.255.0.1
  interface eth1
  interface lo
 exit-address-family
exit
  ```

- #### R102
  
Sono state configurate le **interfacce** `eth0`, `eth1` e di loopback `lo`, abilitando **MPLS** per tutte queste:
  
  ```shell
interface eth0
 ip address 10.1.12.2/30
 mpls enable

interface eth1
 ip address 10.1.23.1/30
 mpls enable

interface lo
 ip address 1.2.0.1/16
 ip address 1.255.0.2/32
 mpls enable
exit
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
router ospf
 ospf router-id 1.255.0.2
 network 1.255.0.2/32 area 0
 network 10.1.12.0/30 area 0
 network 10.1.23.0/30 area 0
exit
  ```
  Configurazione di **MPLS/LDP**:
  
  Sono state abilitate le interfacce che possono accettare pacchetti MPLS, modificando il file `/etc/sysctl.conf`: 
  
```shell
echo 'net.mpls.conf.lo.input = 1' >> /etc/sysctl.conf
echo 'net.mpls.conf.eth0.input = 1' >> /etc/sysctl.conf
echo 'net.mpls.conf.eth1.input = 1' >> /etc/sysctl.conf
echo 'net.mpls.platform_labels = 100000' >> /etc/sysctl.conf
sysctl -p
  ```
  ```shell
mpls ldp
 router-id 1.255.0.2
 ordered-control
 address-family ipv4
  discovery transport-address 1.255.0.2
  interface eth0
  interface eth1
  interface lo
 exit-address-family
exit
  ```
- #### R103
  
Sono state configurate le **interfacce** `eth0`, `eth1` e di loopback `lo`, abilitando **MPLS** per `eth1` e `lo`:
  
  ```shell
interface eth0
 ip address 10.13.31.1/30

interface eth1
 ip address 10.1.23.2/30
 mpls enable

interface lo
 ip address 1.3.0.1/16
 ip address 1.255.0.3/32
 mpls enable
exit
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
router ospf
 ospf router-id 1.255.0.3
 network 1.255.0.3/32 area 0
 network 10.1.23.0/30 area 0
exit
  ```
  Configurazione del protocollo **BGP**:
  
  ```shell
router bgp 100
 bgp router-id 1.255.0.3
 neighbor 1.255.0.1 remote-as 100
 neighbor 1.255.0.1 update-source 1.255.0.3
 neighbor 10.13.31.2 remote-as 300
 address-family ipv4 unicast
  network 1.3.0.0/16
  neighbor 1.255.0.1 next-hop-self
 exit-address-family
exit
  ```
  Configurazione di **MPLS/LDP**:
   Sono state abilitate le interfacce che possono accettare pacchetti MPLS, modificando il file `/etc/sysctl.conf`: 
  
```shell
echo 'net.mpls.conf.lo.input = 1' >> /etc/sysctl.conf
echo 'net.mpls.conf.eth1.input = 1' >> /etc/sysctl.conf
echo 'net.mpls.platform_labels = 100000' >> /etc/sysctl.conf
sysctl -p
  ```
  ```shell
mpls ldp
 router-id 1.255.0.3
 ordered-control
 address-family ipv4
  discovery transport-address 1.255.0.3
  interface eth1
  interface lo
 exit-address-family
exit
  ```
### AS200

> `AS200` è un AS connesso a `AS100`, che fornisce servizi di transito. Sono stati configurati:
> - eBGP peering con `AS100`;
> - iBGP peering;
> - OSPF;
> - `R203` non è BGP speaker e si ha:
>   - default route verso R202;
>   - indirizzo IP pubblico appartente ad un insieme di indirizzi IP di `AS200`;
>   - Access Gateway per la LAN presente, e per il quale si è configurato:
>     - dynamic NAT;
>     - un semplice firewall per consentire solo le connessioni stabilite dalla LAN;
> - `Client-200` è stato configurato per usare un *Mandatory Access Control (AppArmor)*.
> - `Client-200` è un *OpenVPN client*.

- #### R201
  
Sono state configurate le **interfacce** `eth0`, `eth1` e di loopback `lo`:
  
  ```shell
interface eth0
 ip address 10.12.11.1/30

interface eth1
 ip address 10.2.12.2/30

interface lo
 ip address 2.1.0.1/16
 ip address 2.255.0.1/32
exit
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
router ospf
 ospf router-id 2.255.0.1
 network 2.1.0.0/16 area 0
 network 2.255.0.1/32 area 0
 network 10.2.12.0/30 area 0
exit
  ```
  Configurazione del protocollo **BGP**:
  
  ```shell
router bgp 200
 bgp router-id 2.255.0.1
 neighbor 2.255.0.2 remote-as 200
 neighbor 2.255.0.2 update-source 2.255.0.1
 neighbor 10.12.11.2 remote-as 100
 address-family ipv4 unicast
  network 2.1.0.0/16
  neighbor 2.255.0.2 next-hop-self
 exit-address-family
exit
  ```
- #### R202
  
Sono state configurate le **interfacce** `eth0`, `eth1` e di loopback `lo`:
  
  ```shell
interface eth0
 ip address 2.2.23.1/30

interface eth1
 ip address 10.2.12.1/30

interface lo
 ip address 2.2.0.1/16
 ip address 2.255.0.2/32
exit
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
router ospf
 ospf router-id 2.255.0.2
 network 2.2.0.0/16 area 0
 network 2.255.0.2/32 area 0
 network 2.2.23.0/30 area 0
 network 10.2.12.0/30 area 0
exit
  ```
  Configurazione del protocollo **BGP**:
  
  ```shell
router bgp 200
 bgp router-id 2.255.0.2
 neighbor 2.255.0.1 remote-as 200
 neighbor 2.255.0.1 update-source 2.255.0.2
 neighbor 2.2.23.2 remote-as 200
 neighbor 2.2.23.2 update-source 2.255.0.2

 address-family ipv4 unicast
  network 2.2.0.0/16
  network 2.2.23.0/30
  neighbor 2.255.0.1 next-hop-self
  neighbor 2.2.23.2 next-hop-self
 exit-address-family
exit
  ```

- #### R203
  
Sono state configurate le **interfacce** `eth0`, `eth1` e la default route verso `R202`, successivamente è stato abilitato il forwarding degli indirizzi IP ed infine sono stati configurati la NAT ed il firewall in moda da accettare:
- pacchetti che arrivano dall'interfaccia di rete locale e sono destinati alla rete esterna;
- pacchetti che appartengono già a connessioni stabilite;


```shell
 ip addr add 2.2.23.2/30 dev eth0
 ip addr add 192.168.200.1/24 dev eth1
 ip route add default via 2.2.23.1

 sysctl -w net.ipv4.ip_forward=1

 iptables -F

 iptables -P FORWARD DROP
 iptables -P INPUT DROP
 iptables -P OUTPUT ACCEPT
 
 iptables -A FORWARD -m state --state ESTABLISHED -j ACCEPT
 iptables -A FORWARD -i eth1 -o eth0 -s 192.168.200.0/24 -j ACCEPT

 iptables -A POSTROUTING -t nat -o eth0 -j MASQUERADE
  ```
- #### Client-200
    
Il client è stato realizzato utilizzando una macchina virtuale contenente *Lubuntu 22.04.3*. Prima di tutto, sul terminale è stato configurato l'indirizzo dell'interfaccia `enp0s8` e della default route verso il router `R203`:
    
    ```shell
    sudo ip addr add 192.168.200.2/24 dev enp0s8
    sudo ip route add default via 192.168.200.1
    ```

- #### AppArmor

to be implemented!

### AS300

> `AS300` è un customer AS connesso a `AS100`. Ha anche delle lateral peering relationship con `AS400`. Sono stati configurati:
> - eBGP peering con `AS400` e `AS100`;
> - iBGP peering;
> - OSPF;
> - `GW300` non è un BGP speaker, e si ha:
>   - default route verso `R302`;
>   - indirizzo IP pubblico appartenente ad un insieme di indirizzi IP di `AS300`;
>   - rappresenta l'Access Gateway per la DC network ad esso collegata, e presenta:
>     - una dynamic NAT;
>     - è un *OpenVPN server*;

- #### R301
  
Sono state configurate le **interfacce** `eth0`, `eth1` e di loopback `lo`:
  
  ```shell
interface eth0
 ip address 10.13.31.2/30

interface eth1
 ip address 10.3.12.1/30

interface lo
 ip address 3.1.0.1/16
 ip address 3.255.0.1/32
exit
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
router ospf
 ospf router-id 3.255.0.1
 network 3.1.0.0/16 area 0
 network 3.255.0.1/32 area 0
 network 10.3.12.0/30 area 0
exit
  ```
  Configurazione del protocollo **BGP**:
  
  ```shell
router bgp 300
 bgp router-id 3.255.0.1
 neighbor 3.2.0.1 remote-as 300
 neighbor 3.2.0.1 update-source 3.1.0.1
 neighbor 3.255.0.2 remote-as 300
 neighbor 3.255.0.2 update-source 3.255.0.1
 neighbor 10.13.31.1 remote-as 100
 address-family ipv4 unicast
  network 3.1.0.0/16
  neighbor 3.2.0.1 next-hop-self
  neighbor 3.255.0.2 next-hop-self
 exit-address-family
exit
  ```

- #### R302
  
Sono state configurate le **interfacce** `eth0`, `eth1`, `eth2 e di loopback `lo`:
  
  ```shell
interface eth0
 ip address 10.3.12.2/30

interface eth1
 ip address 10.34.21.1/30

interface eth2
 ip address 3.3.23.1/30

interface lo
 ip address 3.2.0.1/16
 ip address 3.255.0.2/32
exit
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
router ospf
 ospf router-id 3.255.0.2
 network 3.2.0.0/16 area 0
 network 3.255.0.2/32 area 0
 network 10.3.12.0/30 area 0
exit
  ```
  Configurazione del protocollo **BGP**:
  
  ```shell
router bgp 300
 bgp router-id 3.255.0.2
 neighbor 3.255.0.1 remote-as 300
 neighbor 3.255.0.1 update-source 3.255.0.2
 neighbor 3.255.0.1 next-hop-self
 neighbor 3.3.23.2 remote-as 300
 neighbor 3.3.23.2 update-source 3.2.0.1
 neighbor 3.3.23.2 next-hop-self
 neighbor 10.34.21.2 remote-as 400
 address-family ipv4 unicast
  network 3.2.0.0/16
  network 3.3.23.0/30
 exit-address-family
exit
  ```
- #### GW300
  > GW300 non è un BGP speaker. Per questo si ha:
  > - default route verso R302;
  > - indirizzi IP pubblico;
  > - fa da Access Gateway per la rete DC:
  >   - una dynamic NAT;
  >   - è un *OpenVPN server*; 

E' stata configurate l' **interfaccia** `eth2`, ed aggiunto una `default route` verso R302. Successivamente è stato abilitato il forwarding degli indirizzi IP come segue:
  ```shell
  ip addr add 3.3.23.2/30 dev eth2
  ip route add default via 3.3.23.1
  
  sysctl -w net.ipv4.ip_forward=1
  ```
Sono stato associate le vlan con id `100` e `200` alle interfacce virtuali `eth0.100` e `eth0.200` per consentire la connettività da e verso l'esterno del datacenter. Si aggiunge l'indirizzo del gateway associato alle due interfacce virtuali e si aggiungono due rotte verso la foglia L1 per raggiungere separatamente i due tenant:
  ```shell
  ip link add link eth1 name eth1.100 type vlan id 100
  ip link add link eth1 name eth1.200 type vlan id 200
 
  ip addr add 10.1.3.2/30 dev eth1.100
  ip addr add 10.1.3.2/30 dev eth1.200

  ip link set eth1.100 up
  ip link set eth1.200 up

  ip route add 192.168.0.0/24 via 10.1.3.1 dev eth1.100
  ip route add 192.168.1.0/24 via 10.1.3.1 dev eth1.200
  ```

E' stata poi configurata la NAT verso l'interfaccia `eth2`:
  ```shell  
  iptables -t nat -A POSTROUTING -o eth2 -j MASQUERADE
  ```

Si configura il firewall per bloccare il traffico da 192.168.0.0/24 to 192.168.1.0/24
  ```shell   
  iptables -F FORWARD
  iptables -P FORWARD ACCEPT
  iptables -A FORWARD -m state --state ESTABLISHED -j ACCEPT
  iptables -A FORWARD -s 192.168.0.0/24 -d 192.168.1.0/24 -j DROP
  iptables -A FORWARD -s 192.168.1.0/24 -d 192.168.0.0/24 -j DROP
  ```

### DC Network
> La DC Network è un data center `leaf-spine` con la presenza di due `leaves` e due `spines`. Nel rete cloud vi sono due `tenants` (A and B), ciascuno dei quali fa da host per due macchine virtuali connesse a `leaf1` e a `leaf2`. Sono stati configurati:
> - VXLAN/EVPN forwarding nella DC network per fornire L2VPNs tra i macchine;
> - In L1, abilitazione della connessione verso la rete esterna. Ovvero, le macchine tenants devono poter raggiungere la rete esterna attraverso il link presente tra L1 e R303, utilizzando l'incapsulamento in dei tunnel OpenVPN quando necessario.

- #### Spine 1
  Sono state configurate le **interfacce** `swp1`, `swp2` e di loopback `lo`:
  ```shell  
  net add interface swp1 ip add 10.1.1.2/30
  net add interface swp2 ip add 10.2.1.2/30
  net add loopback lo ip add 3.3.3.3/32
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
  net add ospf router-id 3.3.3.3
  net add ospf network 0.0.0.0/0 area 0
  ```
  Configurazione del protocollo **MP-eBGP**:
  
  ```shell
  net add bgp autonomous-system 65000
  net add bgp router-id 3.3.3.3
  net add bgp neighbor swp1 remote-as external
  net add bgp neighbor swp2 remote-as external
  net add bgp evpn neighbor swp1 activate
  net add bgp evpn neighbor swp2 activate
  ```

- #### Spine 2
  Sono state configurate le **interfacce** `swp1`, `swp2` e di loopback `lo`:
  ```shell  
  net add interface swp1 ip add 10.1.2.2/30
  net add interface swp2 ip add 10.2.2.2/30
  net add loopback lo ip add 4.4.4.4/32
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
  net add ospf router-id 4.4.4.4
  net add ospf network 0.0.0.0/0 area 0
  ```
  Configurazione del protocollo **MP-eBGP**:
  
  ```shell
  net add bgp autonomous-system 65000
  net add bgp router-id 4.4.4.4
  net add bgp neighbor swp1 remote-as external
  net add bgp neighbor swp2 remote-as external
  net add bgp evpn neighbor swp1 activate
  net add bgp evpn neighbor swp2 activate
  ```

  
- #### Leaf 1
  to be implemented...

- #### Leaf 2
  to be implemented...

- #### A1
  E' stata configurate l' **interfaccia** `eth0`, ed aggiunto una `default route` come segue:
   ```shell  
     ip addr add 192.168.0.1/24 dev eth0
     ip route add default via 192.168.0.254
  ```
- ####  B1
  E' stata configurate l' **interfaccia** `eth0`, ed aggiunto una `default route` come segue:
   ```shell  
     ip addr add 192.168.1.1/24 dev eth0
     ip route add default via 192.168.1.254
  ```
  
- #### A2
  E' stata configurate l' **interfaccia** `eth0`, ed aggiunto una `default route` come segue:
   ```shell  
     ip addr add 192.168.0.2/24 dev eth0
     ip route add default via 192.168.0.254
  ```
  
- #### B2
  E' stata configurate l' **interfaccia** `eth0`, ed aggiunto una `default route` come segue:
   ```shell  
     ip addr add 192.168.1.2/24 dev eth0
     ip route add default via 192.168.1.254
  ```

### AS400

> `AS400` ha una relazione di peering laterale con `AS300`. Si è configurato:
> - eBGP peering with `AS400` and `AS100`;
> - `R402` non è un BGP speaker, ed ha:
>   - default route verso `R401`;
>   - indirizzo IP pubblico;
>   - Access Gateway per la LAN presente, basato su:
>     - dynamic NAT;
>     - è un *OpenVPN client*.

- #### R401
  
Sono state configurate le **interfacce** `eth0`, `eth1` e di loopback `lo`:
  
  ```shell
interface eth0
 ip address 4.4.12.2/30

interface eth1
 ip address 10.34.21.2/30

interface lo
 ip address 4.1.0.1/16
 ip address 4.255.0.1/32
exit
  ```
  Configurazione del protocollo **OSPF**:
  
  ```shell
router ospf
 ospf router-id 4.255.0.1
 network 4.1.0.0/16 area 0
 network 4.255.0.1/32 area 0
 network 4.4.12.0/30 area 0
exit
  ```
  Configurazione del protocollo **BGP**:
  
  ```shell
router bgp 400
 bgp router-id 4.255.0.1
 neighbor 4.4.12.1 remote-as 400
 neighbor 4.4.12.1 update-source 4.255.0.1
 neighbor 10.34.21.1 remote-as 300
 address-family ipv4 unicast
  network 4.1.0.0/16
  network 4.4.12.0/30
  neighbor 4.4.12.1 next-hop-self
 exit-address-family
exit
  ```

- #### R402
  
Sono state configurate le **interfacce** `eth0`, `eth1` e la default route verso `R401`. Successivamente è stato abilitato il forwarding degli indirizzi IP ed infine è stata configurata la NAT come segue:

  ```shell
ip addr add 4.4.12.1/30 dev eth0
ip addr add 192.168.40.1/24 dev eth1
ip route add default via 4.4.12.2

sysctl -w net.ipv4.ip_forward=1

iptables -A POSTROUTING -t nat -o eth0 -j MASQUERADE

  ```
 - #### Client-400
    
Configurazione dell'**interfaccia** `eth0` e della default route verso `R402`:
    
  ```shell
ip addr add 192.168.40.2/24 dev eth0
ip route add default via 192.168.40.1
  ```
