# Proyecto 1: Chapin Red

**Autor:** Marcelo André Juarez Alfaro - 202010367

## 🟢 FASE 1: Diseño de Red y Topología Física

### 1.1 Esquema de Direccionamiento para VLANs (VLSM)

Red base asignada: `192.188.67.0/24`. Se dividió en 5 subredes `/27` para alojar hasta 30 hosts cada una. La primera IP utilizable de cada subred se reserva para el Default Gateway (SVI).

| VLAN        | Departamento | Edificio       | Dirección de Red | Máscara         | Rango Utilizable (Hosts) | Broadcast      | Gateway (SVI)  |
| :---------- | :----------- | :------------- | :--------------- | :-------------- | :----------------------- | :------------- | :------------- |
| **VLAN 10** | Naranja      | Izquierdo      | 192.188.67.0     | 255.255.255.224 | 192.188.67.1 - .30       | 192.188.67.31  | 192.188.67.1   |
| **VLAN 20** | Verde        | Izquierdo      | 192.188.67.32    | 255.255.255.224 | 192.188.67.33 - .62      | 192.188.67.63  | 192.188.67.33  |
| **VLAN 30** | Naranja      | Derecho        | 192.188.67.64    | 255.255.255.224 | 192.188.67.65 - .94      | 192.188.67.95  | 192.188.67.65  |
| **VLAN 40** | Verde        | Derecho        | 192.188.67.96    | 255.255.255.224 | 192.188.67.97 - .126     | 192.188.67.127 | 192.188.67.97  |
| **VLAN 99** | ADMIN        | Administración | 192.188.67.128   | 255.255.255.224 | 192.188.67.129 - .158    | 192.188.67.159 | 192.188.67.129 |

> _Evidencia:_ [📄 Ver Reporte de cálculo VLSM en PDF](./src/reporte_VLSM_192.188.67.0_160826033520.pdf)

### 1.2 Esquema de Direccionamiento para Enlaces (FLSM)

Red base asignada: `10.4.67.0/24`. Se dividió en subredes `/30` para optimizar los enlaces punto a punto entre equipos de Capa 3.

| ID Enlace    | Conexión (Origen -> Destino) | Dirección de Red | Máscara         | IP Extremo A | IP Extremo B |
| :----------- | :--------------------------- | :--------------- | :-------------- | :----------- | :----------- |
| **Enlace 1** | Router 1 -> MSW Core Izq     | 10.4.67.0        | 255.255.255.252 | 10.4.67.1    | 10.4.67.2    |
| **Enlace 2** | MSW Core Izq -> MSW Admin    | 10.4.67.4        | 255.255.255.252 | 10.4.67.5    | 10.4.67.6    |
| **Enlace 3** | MSW Core Izq -> MSW Core Der | 10.4.67.8        | 255.255.255.252 | 10.4.67.9    | 10.4.67.10   |

_(Nota: Esta tabla se expandirá conforme se asignen las IPs a los enlaces inter-edificios de fibra óptica y enrutamiento)._

> _Evidencia:_ [📄 Ver Reporte de cálculo FLSM en PDF](./src/reporte_FLSM_10.4.67.0_160826033200.pdf)

## 🔵 FASE 2: Capa 2, VLANs y Redundancia (LACP/PAgP)

Se configuró el protocolo VTP para la propagación de VLANs desde el MS1 hacia el resto de la topología. Además, se implementó tolerancia a fallos mediante EtherChannel (LACP y PAgP) y Spanning Tree (Rapid-PVST).

> _Evidencia de propagación:_ Captura de `show vlan brief` comprobando la recepción de VLANs en los switches de acceso.
> _[INSERTAR CAPTURA DE PANTALLA DE SH VLAN BRIEF AQUÍ]_
