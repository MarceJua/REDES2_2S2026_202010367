# Proyecto 1: Chapin Red

**Autor:** Marcelo André Juárez Alfaro - 202010367
**Curso:** Redes de Computadoras 2

---

## 💻 BITÁCORA TÉCNICA: Comandos de Configuración

### Fase 2 - Capa 2 (Switching y Redundancia)

#### 1. Configuración VTP y Creación de VLANs

**En MS1 (VTP Server):**

```cisco
enable
configure terminal
vtp domain ChapinRed
vtp mode server
vlan 10
 name VLAN_Naranja_EdificioIZQ_202010367
vlan 20
 name VLAN_Verde_EdificioIZQ_202010367
vlan 30
 name VLAN_Naranja_EdificioDER_202010367
vlan 40
 name VLAN_Verde_EdificioDER_202010367
exit
```

**En MS2, MS3, MS4, MS5, MS7, MS8, MS9, SW1, SW2, SW3, SW4 (VTP Clients):**

```
enable
configure terminal
vtp domain ChapinRed
vtp mode client
```

#### 2. Spanning Tree Protocol (En todos los switches)

```
enable
configure terminal
spanning-tree mode rapid-pvst
```

#### 3. Agregación de Enlaces (EtherChannel)

**LACP (Edificio Izquierdo: MS7, MS8, MS9):**

```
enable
configure terminal
interface range GigabitEthernet 1/0/1 - 5
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode active
exit
```

**PAgP (Edificio Derecho: MS2, MS3, MS4, MS5):**

```
enable
configure terminal
interface range GigabitEthernet 1/0/1 - 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol pagp
 channel-group 2 mode desirable
exit
```

#### 4. Puertos de Acceso y Portfast (Switches Finales)

_(Ejemplo en SW1 - VLAN 10 y 20)_

```
enable
configure terminal
interface FastEthernet 0/20
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit
interface FastEthernet 0/21
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit
```

### Fase 3 - Capa 3 (Enrutamiento y DHCP)

#### 1. Puertas de Enlace (SVI) y DHCP Relay

**En MS7 (Core Izquierdo):**

```
enable
configure terminal
ip routing
interface vlan 10
 ip address 192.188.67.1 255.255.255.224
 ip helper-address 10.4.67.14
 no shutdown
exit
interface vlan 20
 ip address 192.188.67.33 255.255.255.224
 ip helper-address 10.4.67.14
 no shutdown
exit
```

**En MS2 (Core Derecho):**

```
enable
configure terminal
ip routing
interface vlan 30
 ip address 192.188.67.65 255.255.255.224
 ip helper-address 10.4.67.18
 no shutdown
exit
interface vlan 40
 ip address 192.188.67.97 255.255.255.224
 ip helper-address 10.4.67.18
 no shutdown
exit
```

#### 2. Configuración Especial MS6 (Edificio Administración)

```
enable
configure terminal
vtp mode transparent
vlan 99
 name VLAN_ADMIN_EdificioADMIN_202010367
exit

interface GigabitEthernet 1/0/1
 switchport mode access
 switchport access vlan 99
 spanning-tree portfast
exit

ip routing
interface vlan 99
 ip address 192.188.67.129 255.255.255.224
 ip helper-address 10.4.67.14
 no shutdown
exit
```

#### 3. Enlaces Ruteados (Fibra Óptica P2P) y EIGRP 10

_(Ejemplo en MS1 - Core Servidores)_

```
enable
configure terminal
interface GigabitEthernet 1/1/1
 no switchport
 ip address 10.4.67.5 255.255.255.252
 no shutdown
exit

router eigrp 10
 network 192.188.67.0 0.0.0.31
 network 192.188.67.32 0.0.0.31
 network 10.4.67.4 0.0.0.3
 network 10.4.67.8 0.0.0.3
 network 10.4.67.12 0.0.0.3
 network 10.4.67.16 0.0.0.3
exit
```

### Fase 4 - Seguridad y Listas de Control de Acceso (ACLs)

#### 1. ACLs en MS7 (Edificio Izquierdo)

```
enable
configure terminal

! ACL para VLAN 10 (Naranja)
ip access-list extended ACL_NARANJA_IN
 permit icmp 192.188.67.0 0.0.0.31 192.188.67.128 0.0.0.31 echo-reply
 permit tcp 192.188.67.0 0.0.0.31 192.188.67.128 0.0.0.31 established
 deny ip 192.188.67.0 0.0.0.31 192.188.67.128 0.0.0.31
 deny ip 192.188.67.0 0.0.0.31 192.188.67.32 0.0.0.31
 deny ip 192.188.67.0 0.0.0.31 192.188.67.96 0.0.0.31
 permit ip any any
exit

! ACL para VLAN 20 (Verde)
ip access-list extended ACL_VERDE_IN
 permit icmp 192.188.67.32 0.0.0.31 192.188.67.128 0.0.0.31 echo-reply
 permit tcp 192.188.67.32 0.0.0.31 192.188.67.128 0.0.0.31 established
 deny ip 192.188.67.32 0.0.0.31 192.188.67.128 0.0.0.31
 deny ip 192.188.67.32 0.0.0.31 192.188.67.0 0.0.0.31
 deny ip 192.188.67.32 0.0.0.31 192.188.67.64 0.0.0.31
 permit ip any any
exit

! Aplicación en SVIs
interface vlan 10
 ip access-group ACL_NARANJA_IN in
exit
interface vlan 20
 ip access-group ACL_VERDE_IN in
exit
```

#### 2. ACLs en MS2 (Edificio Derecho)

```
enable
configure terminal

! ACL para VLAN 30 (Naranja)
ip access-list extended ACL_NARANJA_DER_IN
 permit icmp 192.188.67.64 0.0.0.31 192.188.67.128 0.0.0.31 echo-reply
 permit tcp 192.188.67.64 0.0.0.31 192.188.67.128 0.0.0.31 established
 deny ip 192.188.67.64 0.0.0.31 192.188.67.128 0.0.0.31
 deny ip 192.188.67.64 0.0.0.31 192.188.67.32 0.0.0.31
 deny ip 192.188.67.64 0.0.0.31 192.188.67.96 0.0.0.31
 permit ip any any
exit

! ACL para VLAN 40 (Verde)
ip access-list extended ACL_VERDE_DER_IN
 permit icmp 192.188.67.96 0.0.0.31 192.188.67.128 0.0.0.31 echo-reply
 permit tcp 192.188.67.96 0.0.0.31 192.188.67.128 0.0.0.31 established
 deny ip 192.188.67.96 0.0.0.31 192.188.67.128 0.0.0.31
 deny ip 192.188.67.96 0.0.0.31 192.188.67.0 0.0.0.31
 deny ip 192.188.67.96 0.0.0.31 192.188.67.64 0.0.0.31
 permit ip any any
exit

! Aplicación en SVIs
interface vlan 30
 ip access-group ACL_NARANJA_DER_IN in
exit
interface vlan 40
 ip access-group ACL_VERDE_DER_IN in
exit
```
