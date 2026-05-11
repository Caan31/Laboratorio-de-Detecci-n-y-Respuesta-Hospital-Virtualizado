# 🔵 Laboratorio de Detección y Respuesta — Hospital Virtualizado

<div align="center">

![pfSense](https://img.shields.io/badge/Firewall-pfSense_CE-212121?style=for-the-badge&logo=pfsense&logoColor=white)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh_4.14.5-3AABE8?style=for-the-badge&logo=wazuh&logoColor=white)
![VirtualBox](https://img.shields.io/badge/Virtualización-VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Atacante-Kali_Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Windows](https://img.shields.io/badge/Endpoint-Windows_10-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Endpoint-Ubuntu_24.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

**Infraestructura de hospital simulada con firewall perimetral y SIEM.**  
Ataques reales detectados y mapeados a MITRE ATT&CK y PCI DSS.

[📄 Ver documentación completa (PDF)](#-documentación)

</div>

---

## 📋 Descripción

Este proyecto documenta el diseño, despliegue y validación de una **infraestructura virtualizada de hospital** orientada a la detección y respuesta ante incidentes de ciberseguridad.

La arquitectura combina dos pilares:
- **pfSense Community Edition** como firewall perimetral, servidor DHCP y DNS
- **Wazuh 4.14.5** como SIEM para la recolección, correlación y visualización de eventos de seguridad

El laboratorio simula un escenario realista donde un atacante externo (Kali Linux en la WAN) intenta comprometer los activos internos del hospital, y el SIEM detecta y registra toda la actividad maliciosa en tiempo real.

---

## 🏗️ Arquitectura del laboratorio

```
                        ┌─────────────────────┐
                        │    🌐 WAN            │
                        │  192.168.1.0/24      │
                        │                      │
                        │  🔴 Kali Linux       │
                        │  (Atacante)          │
                        └──────────┬───────────┘
                                   │
                        ┌──────────┴───────────┐
                        │    🛡️ pfSense CE     │
                        │    Firewall/DHCP/DNS  │
                        │    NAT · Reglas · NTP │
                        └──────────┬───────────┘
                                   │
                        ┌──────────┴───────────┐
                        │    🏥 LAN Hospital    │
                        │  192.168.10.0/24      │
                        │                      │
                        │  ┌─────────────────┐ │
                        │  │ 🔵 Wazuh SIEM   │ │
                        │  │ 192.168.10.2    │ │
                        │  │ Manager+Indexer │ │
                        │  │ + Dashboard     │ │
                        │  └─────────────────┘ │
                        │                      │
                        │  ┌────────┐ ┌──────┐ │
                        │  │🖥️ Win10│ │🐧 Ubu│ │
                        │  │  .11   │ │  .10 │ │
                        │  │Endpoint│ │FTP   │ │
                        │  └────────┘ └──────┘ │
                        └──────────────────────┘
```

| Máquina | SO | IP | Rol |
|---------|----|----|-----|
| **pfSense** | FreeBSD (pfSense CE) | WAN: DHCP / LAN: 192.168.10.1 | Firewall perimetral, DHCP, DNS, NAT |
| **Wazuh Server** | Ubuntu Server | 192.168.10.2 | SIEM — Manager + Indexer + Dashboard |
| **Endpoint Windows** | Windows 10 Pro | 192.168.10.11 | Estación de trabajo con agente Wazuh |
| **Endpoint Ubuntu** | Ubuntu 24.04 LTS | 192.168.10.10 | Servidor FTP con agente Wazuh |
| **Kali Linux** | Kali Linux | 192.168.1.x (WAN) | Máquina atacante para validación |

---

## 📖 Fases del proyecto

El documento se organiza en cuatro capítulos que corresponden a las cuatro fases del despliegue:

### Capítulo 4 — Configuración de pfSense

- Instalación y configuración inicial del firewall
- Configuración de interfaces WAN y LAN
- Servidor DHCP para la red del hospital
- Reglas de firewall y NAT específico
- Sincronización NTP para correlación temporal de logs

### Capítulo 5 — Despliegue del SIEM (Wazuh 4.14.5)

- Instalación de Wazuh Manager, Indexer y Dashboard
- Configuración de la pila completa en un solo servidor
- Verificación de servicios y acceso al dashboard web
- Configuración de reglas de detección

### Capítulo 6 — Incorporación de agentes

- Despliegue del agente Wazuh en Windows 10 Pro
- Despliegue del agente Wazuh en Ubuntu 24.04 LTS
- Verificación de comunicación agent → manager
- Validación de recepción de logs en el dashboard

### Capítulo 7 — Simulación de ataques y verificación

- Escaneo de puertos con **Nmap** desde Kali Linux
- Ataques de fuerza bruta con **Hydra** contra SSH y FTP
- Verificación de detección en tiempo real en el dashboard de Wazuh
- Correlación de eventos y generación de alertas
- Mapeo de hallazgos a **MITRE ATT&CK** y **PCI DSS**

---

## 🔍 Ataques simulados y detección

| Ataque | Herramienta | Objetivo | Técnica MITRE ATT&CK | Detectado |
|--------|------------|----------|----------------------|-----------|
| Escaneo de puertos | Nmap | Toda la LAN | T1046 — Network Service Discovery | ✅ Sí |
| Fuerza bruta SSH | Hydra | Ubuntu 24.04 | T1110.001 — Brute Force: Password Guessing | ✅ Sí |
| Fuerza bruta FTP | Hydra | Ubuntu 24.04 (FTP) | T1110.001 — Brute Force: Password Guessing | ✅ Sí |

---

## 🛡️ Mapeo a frameworks de cumplimiento

### MITRE ATT&CK

| Táctica | Técnica | ID | Cobertura |
|---------|---------|-----|-----------|
| Reconnaissance | Network Service Discovery | T1046 | Wazuh detecta escaneos Nmap |
| Credential Access | Brute Force: Password Guessing | T1110.001 | Alertas por múltiples intentos fallidos SSH/FTP |
| Initial Access | Valid Accounts | T1078 | Registro de accesos exitosos tras intentos fallidos |

### PCI DSS

| Requisito | Descripción | Implementación |
|-----------|-------------|----------------|
| **1.1** | Firewall instalado y mantenido | pfSense con reglas específicas por servicio |
| **2.1** | No usar valores por defecto | Cuentas por defecto deshabilitadas en todos los servicios |
| **6.1** | Identificar vulnerabilidades de seguridad | Wazuh Vulnerability Detector activo |
| **10.1** | Registro de auditoría para acceso a componentes | Logs centralizados en Wazuh de todos los endpoints |
| **10.6** | Revisión de logs de seguridad | Dashboard de Wazuh con alertas en tiempo real |

---

## 🔧 Buenas prácticas aplicadas

- **Separación de redes**: WAN aislada de LAN mediante pfSense con NAT específico
- **Mínimo privilegio**: Cada servicio corre con los permisos estrictamente necesarios
- **Deshabilitación de cuentas por defecto**: En pfSense, Wazuh y endpoints
- **Sincronización NTP**: Todos los nodos sincronizados para correlación temporal precisa de logs
- **Separación de roles**: Firewall y SIEM en máquinas independientes
- **Agentes en todos los endpoints**: Visibilidad completa de la actividad en la LAN

---

## 📂 Estructura del repositorio

```
Hospital-pfSense-Wazuh/
├── README.md                       # Este fichero
└── Hospital_pfSense_Wazuh.pdf      # Documentación completa del proyecto
```

| Archivo | Descripción |
|---------|-------------|
| `Hospital_pfSense_Wazuh.pdf` | Documento completo con las 4 fases del proyecto: configuración de pfSense, despliegue de Wazuh, incorporación de agentes y simulación de ataques. Incluye capturas reales de cada paso y explicación técnica detallada |

---

## 📄 Documentación

El fichero [`Hospital_pfSense_Wazuh.pdf`](./Hospital_pfSense_Wazuh.pdf) contiene la guía técnica completa y reproducible del laboratorio, con:

- Procedimiento paso a paso con capturas de pantalla reales
- Explicación técnica de cada decisión de diseño
- Trazabilidad completa entre la práctica real y la documentación
- Mapeo a controles MITRE ATT&CK y requisitos PCI DSS

---

## 🛠️ Stack tecnológico

| Tecnología | Versión | Uso |
|------------|---------|-----|
| [pfSense CE](https://www.pfsense.org/) | Community Edition | Firewall perimetral, DHCP, DNS, NAT |
| [Wazuh](https://wazuh.com/) | 4.14.5 | SIEM — Manager + Indexer + Dashboard |
| [VirtualBox](https://www.virtualbox.org/) | — | Hipervisor del laboratorio |
| [Kali Linux](https://www.kali.org/) | — | Máquina atacante para validación |
| [Nmap](https://nmap.org/) | — | Escaneo de puertos y descubrimiento de servicios |
| [Hydra](https://github.com/vanhauser-thc/thc-hydra) | — | Ataques de fuerza bruta SSH/FTP |
| Windows 10 Pro | — | Endpoint protegido con agente Wazuh |
| Ubuntu | 24.04 LTS | Endpoint con FTP + agente Wazuh |

---

## 🔮 Líneas futuras

- [ ] Extensión del mapeo a **HIPAA** (normativa sanitaria estadounidense)
- [ ] Extensión del mapeo a **GDPR** (protección de datos europea)
- [ ] Incorporación de más endpoints (servidores de historiales clínicos, dispositivos IoMT)
- [ ] Integración con **TheHive** para gestión de incidentes
- [ ] Reglas personalizadas de Wazuh para detección de ransomware sanitario
- [ ] Automatización de respuesta con Wazuh Active Response

---

## ⚠️ Aviso legal

> Este proyecto ha sido desarrollado **exclusivamente con fines educativos** en el marco de un trabajo académico. Toda la infraestructura es virtual y aislada. Los ataques se ejecutaron únicamente contra máquinas propias dentro del laboratorio. **No se han atacado sistemas reales ni de terceros.**

---

## 👤 Autor

**Arabot** · Carlos Andrés Aragón Nacimba

- 🔐 Estudiante de ciberseguridad | eJPT
- 🐙 GitHub: [@Caan31](https://github.com/Caan31)
- 📅 2025

---

## 📚 Referencias

- [Wazuh — Documentación oficial](https://documentation.wazuh.com/)
- [pfSense — Documentación oficial](https://docs.netgate.com/pfsense/en/latest/)
- [MITRE ATT&CK — Framework](https://attack.mitre.org/)
- [PCI DSS — Requisitos](https://www.pcisecuritystandards.org/)
- [GTFObins](https://gtfobins.github.io/)
- [Nmap — Guía oficial](https://nmap.org/book/)
- [Hydra — THC GitHub](https://github.com/vanhauser-thc/thc-hydra)

---

<div align="center">

*¿Te ha resultado útil? Dale una ⭐ al repositorio.*

</div>
