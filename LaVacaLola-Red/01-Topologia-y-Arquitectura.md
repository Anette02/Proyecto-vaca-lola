# 01 - Topología y Arquitectura de Red

## 1. Corporativo y Enlaces WAN

| Sede / Enlace | VLAN / Tipo | Subred | Máscara | Gateway / IPs |
| :--- | :--- | :--- | :--- | :--- |
| Corporativo - Admin | VLAN 10 | `10.0.10.0/24` | `255.255.255.0` | `10.0.10.1` |
| Corporativo - Servidores (ERP/DNS/NOC) | VLAN 20 | `10.0.20.0/24` | `255.255.255.0` | `10.0.20.1` |
| Corporativo - Gestión | VLAN 99 | `10.0.99.0/24` | `255.255.255.0` | `10.0.99.1` |
| WAN Corp ↔ Región 1 | Punto a punto | `10.255.0.0/30` | `255.255.255.252` | `.1` (Corp) / `.2` (Región 1) |
| WAN Corp ↔ Región 2 | Punto a punto | `10.255.0.4/30` | `255.255.255.252` | `.5` (Corp) / `.6` (Región 2) |
| WAN Corp ↔ Región 3 | Punto a punto | `10.255.0.8/30` | `255.255.255.252` | `.9` (Corp) / `.10` (Región 3) |
| WAN Región `R` ↔ Planta `R` | Punto a punto | `10.255.<1-3>.0/30` | `255.255.255.252` | `.1` (Región) / `.2` (Planta) |
| WAN Región `R` ↔ Tienda `R` | Punto a punto | `10.255.<1-3>.4/30` | `255.255.255.252` | `.5` (Región) / `.6` (Tienda) |

## 2. Plantilla de VLANs y Subredes por Región (`R` = 1, 2 o 3)

### Plantas de Producción (`10.R.VLAN.0/24`)

| VLAN | Uso / Zona | Subred | Máscara | Gateway |
| :--- | :--- | :--- | :--- | :--- |
| 10 | Administración | `10.R.10.0/24` | `255.255.255.0` | `10.R.10.1` |
| 20 | Producción / IoT | `10.R.20.0/24` | `255.255.255.0` | `10.R.20.1` |
| 40 | CCTV | `10.R.40.0/24` | `255.255.255.0` | `10.R.40.1` |
| 50 | Voz (opcional) | `10.R.50.0/24` | `255.255.255.0` | `10.R.50.1` |
| 60 | Wi-Fi Corporativo | `10.R.60.0/24` | `255.255.255.0` | `10.R.60.1` |
| 70 | Invitados | `10.R.70.0/24` | `255.255.255.0` | `10.R.70.1` |
| 99 | Gestión de red | `10.R.99.0/24` | `255.255.255.0` | `10.R.99.1` |

### Tiendas (`10.R.100+VLAN.0/24`)

| VLAN | Uso / Zona | Subred | Máscara | Gateway |
| :--- | :--- | :--- | :--- | :--- |
| 30 | POS / Ventas | `10.R.130.0/24` | `255.255.255.0` | `10.R.130.1` |
| 40 | CCTV | `10.R.140.0/24` | `255.255.255.0` | `10.R.140.1` |
| 60 | Wi-Fi Corporativo | `10.R.160.0/24` | `255.255.255.0` | `10.R.160.1` |
| 70 | Invitados | `10.R.170.0/24` | `255.255.255.0` | `10.R.170.1` |
| 99 | Gestión de red | `10.R.199.0/24` | `255.255.255.0` | `10.R.199.1` |