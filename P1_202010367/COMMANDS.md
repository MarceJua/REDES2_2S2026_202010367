# Bitácora de Comandos - Proyecto 1

## Fase 2 - Configuración de Capa 2 (Switching y Redundancia)

### 1. Configuración de VLANs y VTP

**En MS1 (VTP Server y Creación de VLANs):**

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
vlan 99
 name VLAN_ADMIN_EdificioADMIN_202010367
exit
```

**En el resto de Switches (VTP Clients):**
(Aplicado en MS2, MS3, MS4, MS5, MS6, MS7, MS8, MS9 y SW1 a SW4)

```cisco
enable
configure terminal
vtp domain ChapinRed
vtp mode client
```

### 2. Spanning Tree Protocol (STP)

(Aplicado globalmente en todos los switches de capa 2 y multicapa)

```cisco
enable
configure terminal
spanning-tree mode rapid-pvst
```

### 3. Enlaces Troncales y EtherChannel (Redundancia)

**Conexiones de Fibra Óptica (Red MAN):**
(Ejemplo aplicado en MS1, MS2, MS6 y MS7 en los puertos Gig1/1/1 a 3)

```cisco
interface range GigabitEthernet 1/1/1 - 3
 switchport mode trunk
exit
```

**LACP - Edificio Izquierdo (5 Enlaces):**
(Aplicado en MS9 y MS8)

```cisco
interface range FastEthernet 0/1 - 5
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode active
exit
```

**PAgP - Edificio Derecho (3 Enlaces):**
(Ejemplo del MS3 hacia el MS4)

```cisco
interface range FastEthernet 0/4 - 6
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol pagp
 channel-group 2 mode desirable
exit
```

### 4. Asignación de Puertos de Acceso (Hacia Usuarios Finales)

**En SW1 (Edificio Izquierdo - VLAN 10 Naranja):**

```cisco
enable
configure terminal
interface range FastEthernet 0/20 - 21
 switchport mode access
 switchport access vlan 10
exit
```

**En SW2 (Edificio Izquierdo - VLAN 20 Verde):**

```cisco
enable
configure terminal
interface range FastEthernet 0/20 - 21
 switchport mode access
 switchport access vlan 20
exit
```

**En SW3 (Edificio Derecho - VLAN 30 Naranja):**

```cisco
enable
configure terminal
interface range FastEthernet 0/20 - 21
 switchport mode access
 switchport access vlan 30
exit
```

**En SW4 (Edificio Derecho - VLAN 40 Verde):**

```cisco
enable
configure terminal
interface range FastEthernet 0/20 - 21
 switchport mode access
 switchport access vlan 40
exit
```
