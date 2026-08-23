# Universidad de San Carlos de Guatemala

## Facultad de Ingeniería - Escuela de Ciencias y Sistemas

### Redes de Computadoras 2

**Proyecto 1: Chapin Red**

**Nombre:** Marcelo André Juárez Alfaro  
**Carné:** 202010367

---

## 📖 Introducción

El presente documento detalla el diseño, configuración e implementación de la red corporativa "Chapin Red". La infraestructura conecta cuatro edificios mediante una red MAN utilizando una arquitectura jerárquica, segmentación por VLANs, redundancia de enlaces (LACP/PAgP), enrutamiento dinámico (EIGRP) y políticas estrictas de seguridad mediante Listas de Control de Acceso (ACLs).

---

## 🌍 Topología General del Proyecto

El presente documento detalla el diseño, configuración e implementación de la red corporativa "Chapin Red". La infraestructura conecta cuatro edificios mediante una red MAN utilizando una arquitectura jerárquica de tres capas (Core, Distribución y Acceso).

Se implementó segmentación por VLANs, redundancia de enlaces con LACP y PAgP, enrutamiento dinámico mediante EIGRP y políticas estrictas de seguridad interdepartamental mediante Listas de Control de Acceso (ACLs).

<img width="1720" height="949" alt="topologia_general" src="https://github.com/user-attachments/assets/f381e220-b744-4de8-988e-968d60b503b3" />

_Figura 1: Vista física y lógica de la topología Chapin Red interconectando los cuatro edificios._

---

## 🟢 FASE 1: Diseño de Red y Direccionamiento IP

### 1.1 Esquema de Direccionamiento para VLANs (VLSM)

Red base asignada: `192.188.67.0/24`. Se dividió en 5 subredes `/27` para alojar hasta 30 hosts cada una. La primera IP utilizable de cada subred se reserva para el Default Gateway (SVI).

| VLAN        | Departamento | Edificio       | Dirección de Red | Máscara         | Rango Utilizable      | Gateway (SVI)  |
| :---------- | :----------- | :------------- | :--------------- | :-------------- | :-------------------- | :------------- |
| **VLAN 10** | Naranja      | Izquierdo      | 192.188.67.0     | 255.255.255.224 | 192.188.67.1 - .30    | 192.188.67.1   |
| **VLAN 20** | Verde        | Izquierdo      | 192.188.67.32    | 255.255.255.224 | 192.188.67.33 - .62   | 192.188.67.33  |
| **VLAN 30** | Naranja      | Derecho        | 192.188.67.64    | 255.255.255.224 | 192.188.67.65 - .94   | 192.188.67.65  |
| **VLAN 40** | Verde        | Derecho        | 192.188.67.96    | 255.255.255.224 | 192.188.67.97 - .126  | 192.188.67.97  |
| **VLAN 99** | ADMIN        | Administración | 192.188.67.128   | 255.255.255.224 | 192.188.67.129 - .158 | 192.188.67.129 |

> 📄 **Evidencia:** [Enlace al PDF del cálculo VLSM](./src/reporte_VLSM_192.188.67.0_160826033520.pdf)

### 1.2 Esquema de Direccionamiento para Enlaces (FLSM)

Red base asignada: `10.4.67.0/24`. Se dividió en subredes `/30` para optimizar los enlaces punto a punto de fibra óptica y servidores DHCP.

| ID Enlace    | Conexión (Origen -> Destino) | Dirección de Red | IP Extremo A | IP Extremo B |
| :----------- | :--------------------------- | :--------------- | :----------- | :----------- |
| **Enlace 1** | MS1 (Core) -> MS7 (Izq)      | 10.4.67.4/30     | 10.4.67.5    | 10.4.67.6    |
| **Enlace 2** | MS1 (Core) -> MS2 (Der)      | 10.4.67.8/30     | 10.4.67.9    | 10.4.67.10   |
| **Enlace 3** | MS7 (Izq) -> MS6 (Admin)     | 10.4.67.20/30    | 10.4.67.22   | 10.4.67.21   |
| **Enlace 4** | MS2 (Der) -> MS6 (Admin)     | 10.4.67.24/30    | 10.4.67.26   | 10.4.67.25   |
| **DHCP 1**   | MS1 -> DHCP1 (Server Izq)    | 10.4.67.12/30    | 10.4.67.13   | 10.4.67.14   |
| **DHCP 2**   | MS1 -> DHCP2 (Server Der)    | 10.4.67.16/30    | 10.4.67.17   | 10.4.67.18   |

> 📄 **Evidencia:** [Enlace al PDF del cálculo FLSM](./src/reporte_FLSM_10.4.67.0_160826033200.pdf)

---

## 🟡 FASE 2: Capa 2 - VLANs, VTP y Redundancia

### 2.1 Configuración de VTP y VLANs

Para mantener la consistencia en la base de datos de VLANs a través de la red, se configuró el protocolo VTP con los siguientes parámetros:

- **Dominio VTP:** `ChapinRed`
- **Modo VTP MS1:** `Server`
- **Modo VTP Resto de Switches:** `Client`
- **Modo VTP MS6 (Admin):** `Transparent` (Para aislar el tráfico de la VLAN 99).

<img width="879" height="486" alt="show_vlan" src="https://github.com/user-attachments/assets/9f6c7fc9-d8c2-416f-ab03-b564a14d8101" />

> _Figura 1: Comprobación de VLANs propagadas exitosamente mediante VTP._

### 2.2 Agregación de Enlaces (EtherChannel) y STP

Para garantizar la alta disponibilidad y evitar cuellos de botella, se configuraron enlaces lógicos agrupados:

- **Edificio Izquierdo:** 5 enlaces físicos agrupados utilizando el protocolo estándar **LACP** (IEEE 802.3ad) en modo `active`.
- **Edificio Derecho:** 3 enlaces físicos agrupados utilizando el protocolo propietario de Cisco **PAgP** en modo `desirable`.
- Se habilitó `spanning-tree portfast` en todos los puertos de acceso finales para transiciones rápidas al estado de reenvío.

<img width="874" height="546" alt="etherchannel" src="https://github.com/user-attachments/assets/fc9633ae-3ce2-4d3c-b9c2-4caa0d2b156f" />

> _Figura 2: Verificación del estado de los Port-Channels (LACP y PAgP)._

---

## 🟠 FASE 3: Capa 3 - Enrutamiento Dinámico y Servicios

### 3.1 Enrutamiento Dinámico (EIGRP)

Se configuró el protocolo **EIGRP** (Sistema Autónomo 10) debido a la terminación impar del número de carné. Este protocolo se encargó de enrutar el tráfico entre las SVIs y los enlaces punto a punto de la red MAN.

<img width="893" height="714" alt="eigrp_rutas" src="https://github.com/user-attachments/assets/0d0a91f9-95bf-4dcf-bd61-81961c801bc4" />

> _Figura 3: Tabla de enrutamiento mostrando las rutas aprendidas vía EIGRP (D)._

### 3.2 Servicio DHCP y DHCP Relay

Se configuraron dos servidores centralizados conectados a MS1:

- **DHCP1:** Atiende las solicitudes de las VLANs 10, 20 y 99.
- **DHCP2:** Atiende las solicitudes de las VLANs 30 y 40.
  Dado que los servidores se encuentran en una subred distinta a las computadoras, se configuró el comando `ip helper-address` en las interfaces virtuales (SVI) de los switches Core (MS7, MS2 y MS6) para retransmitir las peticiones por broadcast como paquetes unicast.

<img width="890" height="347" alt="dhcp_successful" src="https://github.com/user-attachments/assets/5949af9a-83a4-4357-8f48-2c9c24faffcc" />

> _Figura 4: Dispositivo final obteniendo configuración IPv4 exitosamente._

---

## 🔴 FASE 4: Políticas de Seguridad (ACLs)

Para cumplir con las restricciones de comunicación interdepartamental, se diseñaron Listas de Control de Acceso (ACLs) Extendidas, aplicadas como filtros de entrada (`in`) en las SVIs correspondientes.

### 4.1 Propósito y Lógica de las ACLs

1.  **Aislamiento de Departamentos:** Las reglas `deny` bloquean explícitamente el tráfico cruzado entre el departamento de Proyectos (Naranja) y Coordinación (Verde).
2.  **Comunicación Intradepartamental:** Las reglas `permit` aseguran que las VLANs del mismo color puedan comunicarse entre los diferentes edificios.
3.  **Tráfico Unidireccional (VLAN ADMIN):** Para garantizar que el departamento de Administración (VLAN 99) inicie conexiones sin ser interrumpido, pero bloqueando el acceso hacia él, se utilizaron sentencias granulares `permit icmp echo-reply` y `permit tcp established` en las VLANs Naranja y Verde. Esto permite la _respuesta_ a los paquetes de ADMIN, pero deniega el _inicio_ de sesión.

<img width="1446" height="537" alt="ping_acls" src="https://github.com/user-attachments/assets/9eac6fab-f1e2-41ac-8a67-8343bb10d1f6" />

> _Figura 5: Resultados de paquetes ICMP demostrando el bloqueo exitoso según las políticas._

---

_(Nota: El registro detallado de todos los comandos de terminal utilizados para estas configuraciones se encuentra documentado en el archivo adjunto `COMMANDS.md`)._
