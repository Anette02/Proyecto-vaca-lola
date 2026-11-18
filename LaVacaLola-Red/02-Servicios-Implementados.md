# 02 - Servicios Implementados

## 1. Enrutamiento Dinámico (OSPF) e Inter-VLAN

Se implementó **OSPF Área 0 (`router ospf 1`)** en los 6 routers troncales/locales (`R-Corp`, `R-Reg1`, `R-Reg2`, `R-Reg3`, `R-Planta1..3`, `R-Tienda1..3`) y en los switches multicapa (`Sw-Core-P1..P3`).

* **Enrutamiento Inter-VLAN en Plantas:** Mediante Switches Multicapa Cisco 3560 (`ip routing`) con interfaces virtuales (SVI) para VLAN 10, 20, 40 y 99, evitando cuellos de botella locales.
* **Enrutamiento Inter-VLAN en Corporativo y Tiendas:** Mediante *Router-on-a-Stick* (`encapsulation dot1Q`) en `R-Corp` y `R-Tienda1..3`.
* **Redundancia WAN:** El anillo entre `R-Reg1`, `R-Reg2` y `R-Reg3` permite convergencia automática ante caída de enlaces principales hacia Corporativo.

---

## 2. Servicio DHCP

Se configuraron pools DHCP en los routers de frontera y switches Core para asignar direccionamiento automático, excluyendo las primeras 10 IPs de cada subred para gateways y equipos fijos.

| Sede | Dispositivo Servidor DHCP | Pools Configurados | DNS Entregado |
| :--- | :--- | :--- | :--- |
| Corporativo | `R-Corp` | `CORP_ADMIN` (`10.0.10.0/24`) | `10.0.20.10` |
| Planta 1, 2, 3 | `Sw-Core-P1`, `P2`, `P3` | `VLAN10_Px`, `VLAN20_Px`, `VLAN40_Px` | `10.0.20.10` |
| Tienda 1, 2, 3 | `R-Tienda1`, `T2`, `T3` | `POS_Tx` (`10.R.130.0/24`), `CCTV_Tx` (`10.R.140.0/24`) | `10.0.20.10` |

---

## 3. Servicio DNS y Servicios Empresariales Simulados (ERP)

El servidor central **`ERP-DNS` (`10.0.20.10`)** ubicado en la VLAN 20 del Corporativo centraliza:

* **Resolución DNS interna:** Registro tipo `A` para `erp.lavacalola.com` apuntando a `10.0.20.10`.
* **Sistema ERP simulado (HTTP):** Portal web accesible desde las terminales administrativas y cajas POS de las 3 regiones para consulta de inventario y registro de ventas.
* **Célula de servicios reales (VirtualBox):** Instancia de monitoreo NOC (Zabbix/Grafana) y simulación de tráfico IoT/CCTV en Región 3.

---

## 4. Seguridad Perimetral, NAT y Control de Acceso (ACL)

* **Segmentación por VLAN:** Separación lógica de tráfico entre Administración (10), Producción/IoT (20), POS (30), CCTV (40) y Gestión (99).
* **Listas de Control de Acceso (ACL):** Políticas aplicadas en los routers de tienda y planta para bloquear el acceso desde subredes POS/Invitados hacia los servidores de Producción (`10.R.20.0/24`). *(Ver detalle en `04-Politicas-de-Seguridad.md`).*
* **NAT / Salida Externa:** Traducción de direcciones en frontera corporativa para acceso controlado hacia Internet (cuando aplique).

---

## 5. Pruebas de Funcionamiento

| Prueba | Origen | Destino | Resultado Esperado | Evidencia |
| :--- | :--- | :--- | :--- | :--- |
| Asignación DHCP | PCs de Planta y Tienda (1, 2, 3) | Router / Core Switch | IPv4, máscara, gateway y DNS `10.0.20.10` | `Evidencias/capturas/01-dhcp.png` |
| Conectividad WAN (OSPF) | `POS-T1`, `POS-T2`, `POS-T3` | `10.0.20.10` (`ERP-DNS`) | `Reply from 10.0.20.10` (0% pérdida tras ARP) | `Evidencias/capturas/02-ping-wan.png` |
| Resolución DNS y ERP | Navegador en `Admin-P1` y `POS-T2` | `http://erp.lavacalola.com` | Carga del portal ERP de La Vaca Lola | `Evidencias/capturas/03-erp-dns.png` |
| Bloqueo por ACL | `POS-T1` (`10.1.130.x`) | `Prod-IoT-P1` (`10.1.20.x`) | `Destination host unreachable` (Bloqueado) | `Evidencias/capturas/04-acl-bloqueo.png` |