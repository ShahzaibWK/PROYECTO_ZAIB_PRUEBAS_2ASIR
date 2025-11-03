# Guía: Instalación del Servidor Proxmox VE

Esta guía detalla el proceso de instalación de Proxmox VE, basándose en el Módulo 2 del curso de virtualización.

## 1. Requisitos Previos

Antes de instalar, asegúrate de cumplir con los requisitos del sistema y de la BIOS.

### Requisitos del Sistema
* **CPU:** Compatible con virtualización (Intel VT-x o AMD-V).
* **RAM:** Mínimo 2 GB (Recomendado 8 GB o más).
* **Disco:** Al menos 32 GB.
* **Red:** Conectividad de red.

### Configuración de la BIOS
Es fundamental verificar que el servidor físico tenga soporte para virtualización.
1.  Entra en la BIOS de la máquina.
2.  Busca la opción "Virtualization Technology" (normalmente en "Advanced BIOS Features").
3.  Asegúrate de que esté **[Enabled]** (Habilitada).

## 2. Preparación de la Instalación

### Paso 2.1: Descarga de la ISO Oficial
Visita el sitio oficial de Proxmox para descargar la última ISO del "Proxmox VE ISO Installer".
* **URL:** `https://www.proxmox.com/en/downloads`

### Paso 2.2: Creación del USB Booteable
Utiliza una herramienta para crear un USB de arranque (booteable).
* Herramientas recomendadas: **Rufus**, **BalenaEtcher** o **Ventoy**.

## 3. Instalación Paso a Paso

Arranca el servidor desde el USB booteable para comenzar el instalador.

### Paso 3.1: Inicio
En el menú de bienvenida, selecciona la opción recomendada:
* **`Install Proxmox VE (Graphical)`**: Esta opción lanza el instalador gráfico que guía paso a paso.

### Paso 3.2: EULA
Lee y acepta el acuerdo de licencia (EULA). Deberás hacer clic en **`I agree`**.

### Paso 3.3: Selección de Disco
El instalador mostrará el disco duro de destino (`Target Harddisk`).
* **Advertencia:** Todos los datos existentes en el disco se perderán.
* Pulsa **`Options`** (Opciones) para configurar el disco.

### Paso 3.4: Opciones de Disco
Aquí puedes definir los parámetros del disco:
* **`Filesystem`**: Tipo de sistema de archivos (ej. `ext4`, `xfs` o `zfs`).
* **`hdsize`**: Tamaño total del disco que usará Proxmox.
* **`swapsize`**: Tamaño de la partición de intercambio (swap).
* **`maxroot`**: Tamaño máximo para la partición raíz (`/`).
* **`minfree`**: Espacio mínimo libre que se reservará.
* **`maxvz`**: Tamaño máximo para el almacenamiento de contenedores y VMs.

**Valores Recomendados (para un laboratorio con 60GB):**
* **hdsize:** 60 GB
* **swapsize:** 4 GB
* **maxroot:** 8 GB
* **minfree:** 1 GB
* **maxvz:** Resto (~47 GB)

Pulsa `OK` y luego `Next`.

### Paso 3.5: Configuración Regional
Configura tu ubicación y zona horaria.
* **Country (País):** `Spain`. Se usa para encontrar mirrors de descarga cercanos.
* **Time zone (Zona horaria):** `Europe/Madrid`.
* **Keyboard Layout (Teclado):** `Spanish`.

Pulsa `Next`.

### Paso 3.6: Contraseña de Administrador
Establece la contraseña para el usuario `root`.
* **Password (Contraseña):** Debe ser una contraseña fuerte (mínimo 8 caracteres, con letras, números y símbolos).
* **Email:** Introduce un email válido para recibir notificaciones de alerta del servidor.

Pulsa `Next`.

### Paso 3.7: Configuración de Red
Configura la red de gestión.
* **Management Interface (Interfaz):** La tarjeta de red que usará Proxmox (ej. `ens33`).
* **Hostname (FQDN):** Nombre completo del host ( `pc3.fpmislata.fp`).
* **IP Address (CIDR):** La IP estática para acceder a la interfaz web (`172.16.204.233/24`).
* **Gateway (Puerta de enlace):** La IP de tu router (ej. `192.168.204.0/24`).
* **DNS Server:** Servidor DNS (`8.8.8.8` o tu DNS interno).

Pulsa `Next`.

### Paso 3.8: Resumen e Instalación
Confirma que toda la información del resumen (`Summary`) es correcta.
* Pulsa **`Install`**.
* El sistema se instalará y se reiniciará automáticamente (si la casilla está marcada).

## 4. Primer Acceso
Una vez reiniciado, la consola del servidor te mostrará la URL de acceso.

* Desde otro equipo en la misma red, abre un navegador web.
* Accede a la dirección: **`https://[IP-DEL-SERVIDOR]:8006`**.
* Inicia sesión con el usuario **`root`** y la contraseña que configuraste.