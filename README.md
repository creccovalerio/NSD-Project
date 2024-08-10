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
    - [R203](#r203) [[Firewall](#firewall)]
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

-`AS100` è un Autonomous System che fornisce funzionalità di accesoo ai customers: `AS200` e `AS300`.     Sono stati configurati:  
  - eBGP peering con `AS200` e `AS300`;
  - iBGP peering tra i border routers;
  - OSPF;
  - LDP/MPLS nella core network;

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
