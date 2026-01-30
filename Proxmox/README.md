# 2. Infraestructura (Servidor Proxmox)

En este proyecto, Proxmox es la plataforma de virtualización elegida para simular el entorno local de nuestros clientes. Nos permite crear y gestionar las máquinas virtuales (VMs) y contenedores (LXCs) necesarios para replicar los componentes de Connectix (servidor, ERP, WMS, GPS) en un entorno de pruebas controlado.

## ¿Qué es Proxmox VE?

Proxmox Virtual Environment (Proxmox VE) es una plataforma de virtualización de código abierto basada en Debian. Permite gestionar máquinas virtuales (KVM) y contenedores (LXC) desde una única interfaz web intuitiva.

### Características Principales

* **Virtualización con KVM y LXC:** KVM ofrece virtualización completa (VMs aisladas) mientras que LXC usa contenedores ligeros a nivel de sistema operativo.
* **Interfaz web completa:** Permite la gestión desde un navegador.
* **Gestión de almacenamiento:** Soporta ZFS, LVM, NFS, etc.
* **Snapshots y backups:** Para copias de seguridad y restauración.
* **Alta disponibilidad (HA):**
* **Clústeres y migración en vivo:**
* **Integración con Ceph:** Para almacenamiento distribuido definido por software.

### Casos de Uso

Proxmox se utiliza comúnmente en:
* Laboratorios educativos.
* Entornos de desarrollo y pruebas.
* Servidores de producción.
* Formación en ciberseguridad y administración de sistemas.

---

## Guías de Documentación de Proxmox

Aquí se encuentran las guías detalladas para este proyecto:

* ➡️ **[Instalación del servidor](./Instalación/README.md)**
    * *Guía paso a paso para instalar Proxmox VE desde cero.*
* ➡️ **[Configuración](./Configuración/README.md)**
    * *Configuración de interfaces de red, Bridges (vmbr0, redwp), NAT y persistencia de iptables.*

### SERVIDORES

*  ➡️ **[DNS-DHCP](./Servidores/DNS-DHCP/README.md)**
    * *Configuración de Bind9, ISC-DHCP-Server y zonas DNS directas e inversas.*

* ➡️ **[WEB](./Servidores/WEB/README.md)**
    * *Instalación de XAMPP (modo texto), despliegue de WordPress y configuración de Base de Datos vía túnel SSH.*