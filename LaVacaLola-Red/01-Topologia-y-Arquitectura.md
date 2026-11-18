# 01 - Topología y Arquitectura de Red

## 1. Alcance de Implementación

De las 5 regiones operativas y 3 plantas por región de **La Vaca Lola**, este proyecto implementa de forma funcional:

* **Corporativo (Sede Central):** Servidores centrales (ERP, Base de Datos, DNS), Administración y NOC.
* **3 Regiones Operativas (Región 1, Región 2 y Región 3):** cada una con **1 Planta de Producción** y **1 Tienda POS** representativa.
* **Infraestructura de Expansión Futura (conceptual):** Regiones 4 y 5, así como las Plantas 2 y 3 de cada región y sucursales adicionales, contempladas dentro del plan de direccionamiento IPv4 sin traslapes.

---

## 2. Arquitectura Propuesta y Justificación Técnica

A diferencia de la topología de referencia en estrella simple (punto único de falla), nuestra arquitectura implementa una **topología WAN Híbrida de Estrella + Anillo Regional Redundante** con **Core Multicapa (L3)** en plantas:

| Decisión de Diseño | Diferencia frente a la referencia | Justificación Técnica (FCAPS y SLA) |
| :--- | :--- | :--- |
| **Anillo redundante entre Regiones** | Se agregan enlaces directos entre Región 1, Región 2 y Región 3 además de su enlace al Corporativo. | **Fallas (F) y SLA:** Si cae un enlace WAN principal hacia el Corporativo, OSPF recalcula la ruta automáticamente a través de una región vecina, protegiendo el SLA del 99.5% en POS y 99% en ERP. |
| **Switch Multicapa (L3) como Core en Plantas** | El Switch Core realiza el enrutamiento inter-VLAN local; el router de frontera se dedica solo a WAN y filtrado. | **Desempeño (P):** Evita el cuello de botella de *Router-on-a-Stick* cuando hay tráfico intenso de CCTV, IoT y administración dentro de la misma planta. |
| **Aislamiento de Servidores y NOC en Corporativo** | Separación estricta entre VLAN 10 (Admin), VLAN 20 (ERP/BD) y VLAN 99 (Gestión/NOC). | **Seguridad (S):** Limita la superficie de ataque hacia el ERP y permite gestionar equipos exclusivamente desde la red de administración segura. |
| **Ambiente Híbrido (Packet Tracer + VirtualBox)** | Macro-topología completa en Cisco Packet Tracer + célula de observabilidad real (Corporativo y Región 3) en VirtualBox. | **Monitoreo (A/P):** Permite validar tanto el diseño completo multi-región como métricas reales (SNMP, latencia, capturas Wireshark y saturación por CCTV). |

---

## 3. Diagrama Conceptual de la Topología

> *Nota: El diagrama detallado en alta resolución se encuentra en `Evidencias/diagramas/topologia-general.png`.*

                           [ Internet / ISP ]
                                  │
                       ┌──────────▼──────────┐
                       │ Firewall / R-Corp   │
                       │   (Corporativo)     │
                       └─┬────────┬────────┬─┘
             ┌───────────┘        │        └───────────┐
             │ VLAN 10/20/99      │                    │
        [SW-Corp]                 │                    │
      (ERP, BD, NOC)              │                    │
                                  │ WAN Principal      │
             ┌────────────────────┼────────────────────┐
             │ 10.255.0.0/30      │ 10.255.0.4/30      │ 10.255.0.8/30
             ▼                    ▼                    ▼
       ┌───────────┐        ┌───────────┐        ┌───────────┐
       │ R-Región 1│◄──────►│ R-Región 2│◄──────►│ R-Región 3│
       └─┬───────┬─┘ .12/30 └─┬───────┬─┘ .16/30 └─┬───────┬─┘
         │       │            │       │            │       │
         │       └────────────┼───────┼────────────┼───────┘ (Anillo: 10.255.0.20/30)
         ▼       ▼            ▼       ▼            ▼       ▼
      Planta1  Tienda1     Planta2  Tienda2     Planta3  Tienda3
      (SW-L3)              (SW-L3)              (SW-L3)

      [Expansión Futura / Conceptual]: Región 4, Región 5, Plantas 2 y 3 por región.

---

## 4. Plan de Direccionamiento IPv4 y VLANs

Se utiliza el bloque privado `10.0.0.0/8` bajo la convención **`10.<Región>.<VLAN>.0/24`** (y `10.<Región>.<100+VLAN>.0/24` para tiendas), reservando `10.4.x.x` y `10.5.x.x` para las regiones de expansión futura.

### 4.1 Corporativo (`10.0.0.0/16`)

| Zona / VLAN | VLAN | Subred | Máscara | Gateway | Uso Principal |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Administración | 10 | `10.0.10.0/24` | `255.255.255.0` | `10.0.10.1` | Equipos administrativos centrales |
| Servidores Centrales | 20 | `10.0.20.0/24` | `255.255.255.0` | `10.0.20.1` | ERP, Base de Datos, DNS, Archivos |
| Gestión y NOC | 99 | `10.0.99.0/24` | `255.255.255.0` | `10.0.99.1` | Servidor NOC, Syslog, SNMP, SSH |

### 4.2 Enlaces WAN y Troncales Regionales (`10.255.0.0/16`)

| Enlace | Rol | Subred | Máscara | IPs Asignadas |
| :--- | :--- | :--- | :--- | :--- |
| Corp ↔ Región 1 | WAN Principal | `10.255.0.0/30` | `255.255.255.252` | `.1` (Corp) / `.2` (Región 1) |
| Corp ↔ Región 2 | WAN Principal | `10.255.0.4/30` | `255.255.255.252` | `.5` (Corp) / `.6` (Región 2) |
| Corp ↔ Región 3 | WAN Principal | `10.255.0.8/30` | `255.255.255.252` | `.9` (Corp) / `.10` (Región 3) |
| Región 1 ↔ Región 2 | Respaldo (Anillo) | `10.255.0.12/30` | `255.255.255.252` | `.13` (R1) / `.14` (R2) |
| Región 2 ↔ Región 3 | Respaldo (Anillo) | `10.255.0.16/30` | `255.255.255.252` | `.17` (R2) / `.18` (R3) |
| Región 1 ↔ Región 3 | Respaldo (Anillo) | `10.255.0.20/30` | `255.255.255.252` | `.21` (R1) / `.22` (R3) |
| Región `R` ↔ Planta `R` | Enlace Regional | `10.255.<1-3>.0/30` | `255.255.255.252` | `.1` (Región) / `.2` (Planta) |
| Región `R` ↔ Tienda `R` | Enlace Regional | `10.255.<1-3>.4/30` | `255.255.255.252` | `.5` (Región) / `.6` (Tienda) |

### 4.3 Plantilla de Plantas de Producción (`10.R.VLAN.0/24`, donde `R` = 1, 2, 3)

| VLAN | Zona | Subred | Máscara | Gateway | Equipos / Servicios |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 10 | Administración | `10.R.10.0/24` | `255.255.255.0` | `10.R.10.1` | PCs de oficina, acceso a ERP |
| 20 | Producción | `10.R.20.0/24` | `255.255.255.0` | `10.R.20.1` | PLCs simulados, sensores IoT |
| 40 | CCTV | `10.R.40.0/24` | `255.255.255.0` | `10.R.40.1` | Cámaras IP y NVR |
| 50 | Voz | `10.R.50.0/24` | `255.255.255.0` | `10.R.50.1` | Telefonía IP (opcional) |
| 60 | Wi-Fi Corporativo | `10.R.60.0/24` | `255.255.255.0` | `10.R.60.1` | Dispositivos internos inalámbricos |
| 70 | Invitados | `10.R.70.0/24` | `255.255.255.0` | `10.R.70.1` | Wi-Fi visitantes (aislado hacia LAN) |
| 99 | Gestión de red | `10.R.99.0/24` | `255.255.255.0` | `10.R.99.1` | Interfaces de administración SSH/SNMP |

### 4.4 Plantilla de Tiendas (`10.R.100+VLAN.0/24`, donde `R` = 1, 2, 3)

| VLAN | Zona | Subred | Máscara | Gateway | Equipos / Servicios |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 30 | POS / Ventas | `10.R.130.0/24` | `255.255.255.0` | `10.R.130.1` | Terminales punto de venta |
| 40 | CCTV | `10.R.140.0/24` | `255.255.255.0` | `10.R.140.1` | Cámaras de tienda |
| 60 | Wi-Fi Corporativo | `10.R.160.0/24` | `255.255.255.0` | `10.R.160.1` | Escáneres / inventario móvil |
| 70 | Invitados | `10.R.170.0/24` | `255.255.255.0` | `10.R.170.1` | Wi-Fi clientes |
| 99 | Gestión de red | `10.R.199.0/24` | `255.255.255.0` | `10.R.199.1` | Administración remota del sitio |