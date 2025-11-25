# 📘 Administración de Sistemas y Servicios en Red

Este documento consolida la teoría y práctica sobre la integración de sistemas operativos, servicios de directorio y administración remota, cubriendo **Samba (Workstation y AD), NFS, VNC, RDP y OpenSSH**.

## 📑 Índice Principal

1. [UD1: Integración de Sistemas Operativos en Red (Samba y NFS)](#unidad-1-integración-de-sistemas-operativos-en-red-samba-y-nfs)
2. [UD2: Administración de Servicio de Directorio (Samba AD DC)](#unidad-2-administración-de-servicio-de-directorio-samba-ad-dc)
3. [UD3: Administración y Acceso Remoto (SSH, VNC, RDP)](#unidad-3-administración-y-acceso-remoto-ssh-vnc-rdp)

---

-----

# Unidad 1: Integración de Sistemas Operativos en Red (Samba y NFS)

Este documento sirve como guía completa de estudio y referencia técnica para la **Unidad Didáctica 1**. Se centra en la interoperabilidad entre sistemas heterogéneos (Windows y Linux) utilizando el protocolo SMB/CIFS (implementado por Samba) y la compartición de archivos en entornos UNIX mediante NFS.

---

## Índice de Contenidos

1. [Introducción a la Interoperabilidad](#1-introducción-a-la-interoperabilidad)
2. [SAMBA (Modo Workstation)](#2-samba-modo-workstation)
    - [2.1. Teoría Avanzada y Arquitectura](#21-teoría-avanzada-y-arquitectura)
    - [2.2. Instalación y Demonios](#22-instalación-y-demonios)
    - [2.3. Gestión de Identidades y Usuarios](#23-gestión-de-identidades-y-usuarios)
    - [2.4. Configuración Profunda: smb.conf](#24-configuración-profunda-smbconf)
    - [2.5. Recursos Compartidos y Control de Acceso](#25-recursos-compartidos-y-control-de-acceso)
    - [2.6. Seguridad y SELinux](#26-seguridad-y-selinux)
    - [2.7. Clientes Samba](#27-clientes-samba)
3. [NFS (Network File System)](#3-nfs-network-file-system)
    - [3.1. Teoría del Protocolo y Versiones](#31-teoría-del-protocolo-y-versiones)
    - [3.2. Configuración del Servidor (Exports)](#32-configuración-del-servidor-exports)
    - [3.3. Mapeo de Usuarios y Seguridad (Squashing)](#33-mapeo-de-usuarios-y-seguridad-squashing)
    - [3.4. Gestión del Servicio](#34-gestión-del-servicio)
    - [3.5. Montaje y Persistencia en Cliente](#35-montaje-y-persistencia-en-cliente)

---

#  1. Introducción a la Interoperabilidad

En la administración de sistemas, rara vez nos encontramos con entornos homogéneos. La necesidad de compartir recursos (archivos, impresoras) entre sistemas operativos con diferentes arquitecturas de archivos y modelos de permisos (Windows vs. UNIX) se resuelve mediante protocolos de red estandarizados.

* **SMB (Server Message Block):** Protocolo nativo de Windows para compartir archivos e impresoras. CIFS (Common Internet File System) fue una versión pública de SMB.
* **Samba:** Suite de software libre que implementa el protocolo SMB/CIFS en sistemas UNIX/Linux, permitiendo que estos actúen como clientes o servidores en redes Windows sin necesidad de instalar software adicional en los clientes Windows.
* **NFS (Network File System):** Protocolo estándar en el mundo UNIX/Linux que permite montar sistemas de archivos remotos como si fueran locales, optimizado para el rendimiento en redes locales.

---

## 2. SAMBA (Modo Workstation)

### 2.1. Teoría Avanzada y Arquitectura
Samba no es un solo programa, sino una suite que implementa una docena de servicios y protocolos, incluyendo NetBIOS sobre TCP/IP (NetBT), RPC (Remote Procedure Calls), y la gestión de autenticación.

En el modo **Workstation (Standalone Server)**, el servidor Samba no forma parte de un dominio de Active Directory. Actúa de forma independiente, gestionando su propia base de datos de usuarios y contraseñas localmente. Es ideal para compartir archivos en redes pequeñas o grupos de trabajo (Workgroups).

### 2.2. Instalación y Demonios
El servicio se basa principalmente en dos demonios que deben estar en ejecución:

1.  **`smbd` (Server Message Block Daemon):** Es el "corazón" de Samba. Gestiona la transferencia de archivos, la autenticación de usuarios y el bloqueo de archivos (file locking) para evitar corrupción de datos cuando varios usuarios acceden al mismo fichero. Escucha en los puertos **TCP 139 y 445**.
2.  **`nmbd` (NetBIOS Message Block Daemon):** Se encarga de la resolución de nombres NetBIOS (convertir nombres de máquinas como `SERVIDOR-01` a direcciones IP) y del "browsing" (la lista de equipos que ves en "Red" en Windows). Escucha en los puertos **UDP 137 y 138**.

**Instalación y Configuración de Red:**
```bash
# Instalación del paquete en Fedora/RHEL
sudo dnf -y install samba

# Configuración del Firewall (Imprescindible para permitir el tráfico entrante)
# Abre los puertos 137/138 (UDP) y 139/445 (TCP)
sudo firewall-cmd --permanent --add-service=samba
sudo firewall-cmd --reload

# Habilitar el arranque automático e iniciar los servicios
sudo systemctl enable --now smb nmb
````

### 2.3. Gestión de Identidades y Usuarios

Samba **no** utiliza directamente el archivo `/etc/passwd` o `/etc/shadow` de Linux para la autenticación debido a que los hashes de contraseñas de Windows (NTLM) y Linux (SHA-512/yescrypt) son incompatibles.

Samba mantiene su propia base de datos de contraseñas (usualmente `passdb.tdb` en `/var/lib/samba/private/`). Por tanto, el proceso de creación de un usuario es de dos pasos:

1.  **Identidad del Sistema:** El usuario debe existir en Linux para tener un UID (User ID) y permisos en el sistema de archivos.
2.  **Identidad Samba:** El usuario debe ser dado de alta en Samba para generar el hash compatible con Windows.

**Comandos de Gestión:**

```bash
# 1. Crear usuario de sistema (sin login shell para seguridad)
# Esto evita que el usuario pueda conectarse por SSH, solo por Samba.
sudo useradd -s /sbin/nologin marta

# 2. Añadir el usuario a la base de datos Samba
sudo smbpasswd -a marta

# Otros comandos administrativos:
sudo pdbedit -L           # Listar usuarios registrados en Samba
sudo smbpasswd -x usuario # Eliminar usuario de la base de datos Samba
sudo smbpasswd -d usuario # Deshabilitar usuario (temporalmente)
sudo smbpasswd -e usuario # Habilitar usuario
```

### 2.4. Configuración Profunda: `smb.conf`

El archivo `/etc/samba/smb.conf` controla todo el comportamiento. Se estructura en formato INI.

**Sección [global]:** Define parámetros que afectan a todo el servidor.

  * `workgroup`: Debe coincidir con el grupo de trabajo de los clientes Windows (por defecto WORKGROUP).
  * `server string`: Descripción del servidor que ven los clientes. Se pueden usar variables (`%L` nombre del servidor, `%v` versión).
  * `security = user`: Modelo de seguridad donde cada cliente debe autenticarse con usuario/contraseña.
  * `passdb backend = tdbsam`: Define el motor de base de datos para usuarios.
  * `map to guest = bad user`: Fundamental para compartir carpetas públicas. Si un usuario intenta entrar y falla la autenticación (o es desconocido), Samba lo trata como "invitado" en lugar de rechazar la conexión inmediatamente.

**Ejemplo de Configuración Global:**

```ini
[global]
    workgroup = ASIR2
    server string = Servidor Samba %L
    security = user
    passdb backend = tdbsam
    
    # Configuración WINS (Resolución de nombres antigua)
    wins support = yes
    dns proxy = no
    
    # Optimización: Deshabilitar impresión si es solo servidor de archivos
    load printers = no
    printing = bsd
    printcap name = /dev/null
    disable spoolss = yes
```

**Verificación:** Siempre usa `testparm` tras editar el fichero para detectar errores de sintaxis antes de reiniciar el servicio.

### 2.5. Recursos Compartidos y Control de Acceso

Cada recurso (carpeta compartida) tiene su propia sección. Los permisos finales efectivos son la **intersección (el más restrictivo)** entre los permisos del sistema de archivos Linux (`chmod/chown`) y los permisos definidos en `smb.conf`.

**Directivas de Recurso:**

  * `path`: Ruta absoluta en el servidor.
  * `writable = yes` o `read only = no`: Permite escritura.
  * `valid users`: Lista de control de acceso.
      * `usuario`: Solo ese usuario.
      * `@grupo`: Todos los miembros del grupo Linux `grupo`.
  * `browseable`: Si es `yes`, aparece en la lista de red. Si es `no`, está oculto (tipo `C$` en Windows) pero accesible si conoces la ruta.
  * `create mask` y `directory mask`: Definen los permisos (en octal) que tendrán los nuevos archivos creados a través de la red. Es una máscara positiva (se aplica con AND).

**Ejemplo 1: Recurso Privado para un Grupo**

```ini
[informatica]
    path = /srv/samba/informatica
    comment = Documentación del Dept. Informática
    valid users = @informaticos    # Solo miembros del grupo 'informaticos'
    read only = no                 # Permite escribir
    create mask = 0740             # Archivos: rwx para dueño, r para grupo
    directory mask = 0750          # Directorios: acceso completo dueño y grupo
```

**Ejemplo 2: Recurso Público (Invitados)**
Para que esto funcione, debe estar configurado `map to guest` en `[global]`.

```ini
[publico]
    path = /srv/samba/anonimo
    guest ok = yes                 # No pide contraseña
    read only = yes                # Solo lectura para invitados
```

**Otras directivas avanzadas:**

  * `hide files = /patron/`: Oculta archivos (atributo hidden) pero permite acceso si se conoce el nombre.
  * `veto files = /patron/`: Prohíbe totalmente el acceso y la visibilidad a archivos que cumplan el patrón (ej. `/*.exe/`).
  * `hosts allow = 192.168.1.`: Seguridad a nivel de red, solo permite IPs de esa subred.

### 2.6. Seguridad y SELinux

En sistemas como Fedora, RHEL o CentOS, **SELinux (Security-Enhanced Linux)** está activo por defecto y bloquea el acceso de Samba a carpetas del sistema a menos que se etiqueten correctamente.

Si creas una carpeta fuera de las rutas estándar, Samba no podrá leerla aunque los permisos `chmod` sean 777.

**Gestión de Contextos SELinux:**

```bash
# Asignar el contexto 'samba_share_t' de forma persistente a una ruta
sudo semanage fcontext -a -t samba_share_t "/srv/samba(/.*)?"

# Aplicar el cambio de contexto al disco
sudo restorecon -Rv /srv/samba/
```

**Booleanos de SELinux:**
Son interruptores para permitir funcionalidades específicas.

```bash
# Permitir compartir los directorios HOME (/home/usuario)
sudo setsebool -P samba_enable_home_dirs 1
```

### 2.7. Clientes Samba

**Desde Windows:**

  * Uso del explorador de archivos: Escribir `\\IP_SERVIDOR` o `\\NOMBRE_NETBIOS` en la barra de direcciones.
  * Mapear unidad de red: Asignar una letra (ej. `Z:`) a un recurso compartido para que aparezca como disco local.
  * Comando `net use`: `net use Z: \\servidor\recurso /user:usuario`.

**Desde Linux:**

  * **Interactivo (`smbclient`):** Funciona similar a un cliente FTP. Útil para pruebas rápidas o scripts.
    ```bash
    smbclient //192.168.1.150/recurso -U usuario
    ```
  * **Montaje en Sistema de Archivos (`cifs-utils`):** Permite integrar la carpeta remota en el árbol de directorios local.
    ```bash
    # Montaje manual
    sudo mount -t cifs //192.168.1.150/recurso /mnt/punto_montaje -o user=usuario,pass=clave
    ```
  * **Navegadores Gráficos:** En Dolphin (KDE) o Nautilus (GNOME), usar la barra de dirección con el protocolo `smb://`: `smb://192.168.1.150`.

-----

## 3\. NFS (Network File System)

### 3.1. Teoría del Protocolo y Versiones

NFS es el estándar *de facto* para compartir archivos en entornos UNIX/Linux. A diferencia de Samba, que "emula" Windows, NFS es nativo. Se basa en el mecanismo RPC (Remote Procedure Call).

  * **NFSv3:** Versión clásica. Utiliza el puerto 2049 pero depende del **Portmapper** (puerto 111) para asignar puertos dinámicos aleatorios a servicios auxiliares como `mountd` o `nlockmgr`. Esto lo hace difícil de gestionar a través de firewalls. No tiene autenticación fuerte (confía en la IP del cliente y el UID del usuario).
  * **NFSv4:** Versión moderna. Funciona únicamente sobre TCP y utiliza un solo puerto (**2049**), eliminando la necesidad de portmapper para la comunicación de datos (aunque rpcbind sigue siendo necesario para el descubrimiento inicial). Soporta características avanzadas como ACLs y seguridad Kerberos.

### 3.2. Configuración del Servidor (Exports)

La configuración no se hace en un archivo `.conf` complejo, sino en `/etc/exports`, que es una lista de control de acceso.

**Sintaxis de `/etc/exports`:**
`<directorio_local>  <cliente1>(opciones)  <cliente2>(opciones)`

  * **Cliente:** Puede ser una IP (`192.168.1.10`), un nombre DNS (`pc1.dominio`), una red completa (`192.168.1.0/24`) o un comodín `*` (todos).

**Ejemplos Prácticos:**

```bash
# Compartir /srv/nfs/datos con una máquina específica, permitiendo lectura/escritura
/srv/nfs/datos   192.168.1.10(rw,sync,no_root_squash)

# Compartir /srv/nfs/publico con toda la red en modo solo lectura
/srv/nfs/publico 192.168.1.0/24(ro,sync)
```

### 3.3. Mapeo de Usuarios y Seguridad (Squashing)

NFS confía plenamente en el cliente. Si en el cliente soy el usuario con UID 1000, el servidor NFS me dejará acceder a los archivos del UID 1000 del servidor. Esto es un riesgo si los UIDs no están sincronizados entre máquinas.

El mayor riesgo es el usuario **root**. Si un administrador local (root) en el cliente accede al servidor NFS, podría modificar cualquier archivo del servidor. Para evitar esto, existe el **Squashing**:

  * `root_squash` (Por defecto): Si el cliente conecta como `root` (UID 0), el servidor lo "aplasta" (mapea) automáticamente al usuario anónimo `nfsnobody` (o `nobody`, UID 65534), que tiene privilegios mínimos.
  * `no_root_squash`: Permite que el `root` del cliente actúe como `root` real en el servidor. Es necesario para clientes sin disco (diskless clients) que arrancan por red, pero es **muy inseguro** en otros contextos.
  * `all_squash`: Convierte a *todos* los usuarios (no solo a root) en el usuario anónimo. Útil para carpetas públicas tipo FTP.

### 3.4. Gestión del Servicio

**Firewall:**
NFS requiere abrir varios servicios debido a su arquitectura RPC.

```bash
# Abrir nfs, mountd y rpc-bind
sudo firewall-cmd --permanent --add-service={nfs,mountd,rpc-bind}
sudo firewall-cmd --reload
```

**Comando `exportfs`:**
Permite gestionar la lista de exportaciones sin reiniciar el servicio, manteniendo activas las conexiones existentes.

  * `sudo exportfs -r`: Relee `/etc/exports` y aplica cambios (Reload).
  * `sudo exportfs -v`: Muestra qué se está compartiendo y con qué opciones (Verbose).
  * `sudo exportfs -u <dir>`: Deja de compartir un directorio (Unexport).
  * `sudo exportfs -a`: Exporta o desexporta todo según el archivo de configuración.

### 3.5. Montaje y Persistencia en Cliente

**Descubrimiento:**
El cliente puede preguntar al servidor qué carpetas tiene disponibles.

```bash
showmount -e IP_DEL_SERVIDOR
```

**Montaje Manual:**

```bash
# Sintaxis: mount -t nfs Servidor:RutaRemota PuntoMontajeLocal
sudo mount -t nfs 192.168.1.150:/srv/nfs/datos /mnt/datos
```

**Montaje Persistente (`/etc/fstab`):**
Para que la carpeta se monte sola al arrancar el equipo. Es crucial usar la opción `_netdev`, que indica al sistema operativo que debe esperar a que la red esté levantada antes de intentar montar este recurso, evitando bloqueos en el arranque.

```fstab
192.168.1.150:/srv/nfs/datos   /mnt/datos   nfs   rw,sync,_netdev   0  0
```
---




















# Unidad 2: Administración de Servicio de Directorio (Samba AD DC)


---

## 📑 Índice de Contenidos

1. [Fundamentos Teóricos de Active Directory con Samba](#1-fundamentos-teóricos-de-active-directory-con-samba)
    - [1.1. Arquitectura y Protocolos](#11-arquitectura-y-protocolos)
    - [1.2. Roles FSMO y Catálogo Global](#12-roles-fsmo-y-catálogo-global)
2. [Preparación del Entorno](#2-preparación-del-entorno)
    - [2.1. Requisitos de Red y Sistema](#21-requisitos-de-red-y-sistema)
    - [2.2. Sistema de Archivos (ACLs y XATTR)](#22-sistema-de-archivos-acls-y-xattr)
3. [Despliegue del Dominio (Provisioning)](#3-despliegue-del-dominio-provisioning)
    - [3.1. El proceso de Provisión](#31-el-proceso-de-provisión)
    - [3.2. Integración UNIX (RFC2307)](#32-integración-unix-rfc2307)
4. [Configuración de Servicios Críticos](#4-configuración-de-servicios-críticos)
    - [4.1. Kerberos (Autenticación)](#41-kerberos-autenticación)
    - [4.2. DNS (Resolución de Servicios)](#42-dns-resolución-de-servicios)
    - [4.3. NTP (Sincronización de Tiempo)](#43-ntp-sincronización-de-tiempo)
5. [Gestión de Identidades](#5-gestión-de-identidades)
    - [5.1. Objetos: Usuarios y Grupos](#51-objetos-usuarios-y-grupos)
    - [5.2. Políticas de Contraseñas (PSO)](#52-políticas-de-contraseñas-pso)
6. [Administración de Políticas y Estructura](#6-administración-de-políticas-y-estructura)
    - [6.1. Unidades Organizativas (OU)](#61-unidades-organizativas-ou)
    - [6.2. Directivas de Grupo (GPO) y Sysvol](#62-directivas-de-grupo-gpo-y-sysvol)
7. [Integración de Clientes Linux (Winbind)](#7-integración-de-clientes-linux-winbind)
    - [7.1. NSS y PAM](#71-nss-y-pam)
8. [Auditoría y Monitorización](#8-auditoría-y-monitorización)

---

## 1. Fundamentos Teóricos de Active Directory con Samba

### 1.1. Arquitectura y Protocolos
Un Controlador de Dominio Samba 4 no es una simple base de datos de usuarios; es una orquestación compleja de varios servicios estándar que trabajan al unísono:

* **LDAP (Lightweight Directory Access Protocol):** Es el "cerebro". Actúa como base de datos jerárquica donde se almacenan todos los objetos del dominio (usuarios, computadoras, impresoras, políticas). Samba utiliza una base de datos interna llamada **LDB** (una versión de TDB con semántica LDAP) para almacenar esta información.
* **Kerberos (KDC):** Es el "portero". Protocolo de autenticación de red que utiliza criptografía de clave simétrica. Evita que las contraseñas viajen por la red. Emite "tickets" (TGT y TGS) que permiten a los usuarios acceder a servicios una vez autenticados.
* **DNS (Domain Name System):** Es el "mapa". En AD, el DNS es dinámico. Los controladores de dominio registran registros **SRV** especiales (como `_ldap._tcp` o `_kerberos._udp`) para que los clientes sepan a qué IP dirigirse para iniciar sesión. Sin un DNS correcto, AD no funciona.
* **SMB/CIFS:** Es el "transporte". Se utiliza para compartir archivos, pero en el contexto de un DC, es vital para compartir la carpeta **SYSVOL**, que contiene las Políticas de Grupo (GPO) y scripts de inicio de sesión que los clientes descargan al arrancar.

### 1.2. Roles FSMO y Catálogo Global
Al igual que en Windows Server, Samba soporta los 5 roles FSMO (Flexible Single Master Operation) necesarios para evitar conflictos en la replicación:
1.  **Schema Master:** Controla las definiciones de objetos (ej. qué atributos tiene un usuario).
2.  **Domain Naming Master:** Controla la creación/borrado de dominios en el bosque.
3.  **PDC Emulator:** Crucial para compatibilidad con clientes antiguos, cambios de contraseña y sincronización de hora.
4.  **RID Master:** Asigna bloques de identificadores (RIDs) a otros DCs para crear objetos (SIDs).
5.  **Infrastructure Master:** Mantiene referencias a objetos de otros dominios.

Además, Samba actúa como **Catálogo Global (GC)**, escuchando en los puertos 3268/3269 para responder consultas sobre objetos de todo el bosque.

---

## 2. Preparación del Entorno

### 2.1. Requisitos de Red y Sistema
Un Controlador de Dominio es una pieza de infraestructura crítica. No debe depender de otros para su configuración de red básica.

* **Dirección IP Estática:** El DC debe tener una IP fija. Si la IP cambia, los registros DNS apuntarán a un lugar incorrecto y los clientes no podrán iniciar sesión.
* **Nombre de Host (FQDN):** El nombre del equipo debe estar bien definido. El FQDN (*Fully Qualified Domain Name*) combina el nombre de la máquina y el dominio (ej. `atenea.fpmislata.fp`).

**Configuración `/etc/hosts`:**
El archivo hosts debe resolver el FQDN a la IP de la interfaz LAN (no a 127.0.0.1).
```bash
127.0.0.1   localhost
172.18.0.1  atenea.fpmislata.fp  atenea
````

### 2.2. Sistema de Archivos (ACLs y XATTR)

Samba requiere un sistema de archivos que soporte **Atributos Extendidos (xattr)** y **Listas de Control de Acceso (ACLs)** para mapear los permisos de Windows (NT ACLs) al sistema de archivos de Linux.

  * Sistemas modernos como **XFS** (por defecto en Rocky/Fedora) o **EXT4** ya incluyen soporte nativo.
  * Verificar soporte: `cat /boot/config-$(uname -r) | grep ACL`.

-----

## 3\. Despliegue del Dominio (Provisioning)

### 3.1. El proceso de Provisión

El comando `provision` inicializa la base de datos, crea los certificados de seguridad, configura la zona DNS inicial y establece la cuenta de Administrador. Es un proceso destructivo si ya existe una configuración previa.

**Comando de Provisión:**

```bash
sudo samba-tool domain provision --use-rfc2307 --interactive
```

  * **Realm:** El dominio DNS en mayúsculas (ej. `FPMISLATA.FP`). Es el reino de Kerberos.
  * **Domain:** El nombre NetBIOS para compatibilidad antigua (ej. `FPMISLATA`).
  * **DNS Backend:**
      * `SAMBA_INTERNAL`: Recomendado. Samba gestiona el DNS directamente. Sencillo y eficaz.
      * `BIND9_DLZ`: Samba actualiza zonas en un servidor BIND externo. Más complejo, usado en infraestructuras grandes.

### 3.2. Integración UNIX (RFC2307)

La opción `--use-rfc2307` carga extensiones en el esquema LDAP de Active Directory.

  * **¿Por qué es importante?** Windows identifica usuarios por **SID** (Security Identifier, cadenas largas numéricas), mientras que Linux usa **UID/GID** (números enteros simples).
  * RFC2307 añade atributos a los objetos de usuario en AD (`uidNumber`, `gidNumber`) para que Samba pueda asignar un ID de Linux consistente a un usuario de Windows en todas las máquinas del dominio.

-----

## 4\. Configuración de Servicios Críticos

### 4.1. Kerberos (Autenticación)

Samba genera un archivo `krb5.conf` optimizado para el nuevo dominio. El sistema operativo Linux (las librerías del sistema) necesita este archivo para saber dónde está el KDC (Key Distribution Center) predeterminado.

**Configuración:**
Sobrescribimos el archivo del sistema con el generado por Samba:

```bash
sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

*Esto permite que herramientas como `kinit` o el login del sistema encuentren el dominio.*

### 4.2. DNS (Resolución de Servicios)

El servidor debe usar su propio servicio DNS para resolver las consultas del dominio. Si usa el DNS de Google o del ISP, no podrá encontrar los registros SRV de AD.

**Configuración `/etc/resolv.conf`:**

```bash
nameserver 127.0.0.1
domain fpmislata.fp
```

**Gestión de Zonas:**
Samba permite gestionar el DNS sin editar archivos de texto, usando RPC. Es vital crear la **zona inversa** para que las herramientas de diagnóstico (`nslookup`, logs) resuelvan IPs a nombres.

```bash
# Crear zona inversa para la red 172.18.x.x
samba-tool dns zonecreate 172.18.0.1 18.172.in-addr.arpa

# Crear un registro A manualmente
samba-tool dns add 172.18.0.1 fpmislata.fp PC01 A 172.18.0.50
```

### 4.3. NTP (Sincronización de Tiempo)

Kerberos es extremadamente sensible al tiempo. Si el reloj de un cliente difiere del servidor en más de **5 minutos**, el token de seguridad se considera inválido para prevenir ataques de repetición (Replay Attacks).

  * Samba no es un servidor de hora completo, pero ofrece un socket para firmar paquetes NTP.
  * Se debe configurar `chronyd` o `ntpd` para que sirva la hora a la red y permita a Samba firmar las respuestas.

-----

## 5\. Gestión de Identidades

La gestión se puede realizar gráficamente desde Windows (RSAT) o mediante la potente herramienta de línea de comandos `samba-tool`.

### 5.1. Objetos: Usuarios y Grupos

Los usuarios en AD son objetos globales. Al crearlos, se les asigna un SID único.

**Comandos de gestión:**

```bash
# Crear un usuario básico
samba-tool user create emilia contraseña123

# Crear usuario con atributos específicos (Nombre real, descripción)
samba-tool user create benito contraseña123 --given-name="Benito" --surname="Pérez"

# Listar y Borrar
samba-tool user list
samba-tool user delete emilia

# Gestión de Grupos
samba-tool group add Comercial
samba-tool group addmembers Comercial emilia
```

**Caducidad de Cuentas:**
Por seguridad, las cuentas pueden tener fecha de expiración.

```bash
# Establecer caducidad en 30 días
samba-tool user setexpiry emilia --days 30

# Hacer que la cuenta nunca caduque (ej. cuentas de servicio)
samba-tool user setexpiry emilia --noexpiry
```

### 5.2. Políticas de Contraseñas (PSO)

Active Directory aplica políticas de contraseña por defecto (complejidad, longitud, historial). Samba permite modificar esto a nivel global o mediante objetos de configuración de contraseña (FGPP - Fine Grained Password Policies).

```bash
# Ver política actual
samba-tool domain passwordsettings show

# Desactivar complejidad (útil en laboratorios, NO en producción)
samba-tool domain passwordsettings set --complexity=off

# Cambiar longitud mínima
samba-tool domain passwordsettings set --min-pwd-length=8
```

-----

## 6\. Administración de Políticas y Estructura

### 6.1. Unidades Organizativas (OU)

Las OUs son contenedores dentro del dominio que permiten:

1.  Organizar objetos jerárquicamente (por departamentos, ubicaciones).
2.  Delegar administración (dar permisos a un usuario solo sobre esa OU).
3.  Aplicar Directivas de Grupo (GPO) específicas.

<!-- end list -->

```bash
# Crear una OU
samba-tool ou create "OU=Ventas"

# Mover un usuario existente a una OU
samba-tool user move emilia "OU=Ventas"
```

### 6.2. Directivas de Grupo (GPO) y Sysvol

Las GPO son conjuntos de reglas (configuración de escritorio, seguridad, instalación de software) que el DC envía a los clientes Windows.

  * Se almacenan físicamente en la carpeta compartida **SYSVOL** (`/var/lib/samba/sysvol`).
  * Samba replica esta carpeta entre controladores de dominio (usando Rsync o GlusterFS, ya que la replicación DFS-R nativa aún es experimental en Samba).
  * **Gestión:** Se recomienda usar **RSAT (Group Policy Management)** desde Windows. `samba-tool` permite verlas o forzar su replicación, pero no editarlas fácilmente.

<!-- end list -->

```bash
# Listar GPOs en el dominio
samba-tool gpo listall

# Ver qué GPOs se aplican a un usuario específico (simulación)
samba-tool gpo list user emilia
```

-----

## 7\. Integración de Clientes Linux (Winbind)

Para que un cliente Linux (o el propio servidor Samba) pueda "ver" y utilizar los usuarios del dominio, se necesita un nexo entre el sistema AD y el sistema Linux local.

### 7.1. NSS y PAM

  * **Winbind:** Es el demonio de Samba que habla con el DC, solicita información de usuarios y grupos, y gestiona la autenticación.
  * **NSS (Name Service Switch):** Configurado en `/etc/nsswitch.conf`, le dice a Linux: "Si buscas un usuario y no está en `/etc/passwd`, pregúntale a Winbind".
  * **PAM (Pluggable Authentication Modules):** Permite que el login de Linux (`/bin/login`, SSH, GDM) use Winbind para verificar la contraseña contra AD.

**Comando `authselect` (Rocky/RHEL/Fedora):**
Automatiza la edición de archivos de configuración PAM y NSS.

```bash
# Habilitar perfil winbind y forzar la escritura de archivos
sudo authselect select winbind --force
```

**Verificación:**
Si la integración funciona, el comando `id` debe devolver datos de un usuario del dominio que NO existe en local.

```bash
id "FPMISLATA\administrator"
# Salida esperada: uid=3000000(FPMISLATA\administrator) gid=...
```

-----

## 8\. Auditoría y Monitorización

Samba permite niveles granulares de registro (logging) para auditoría de seguridad y depuración.

**Parámetros en `smb.conf`:**

```ini
[global]
    # Nivel de log global (0-10). 1 es normal, 3 para depuración, 10 es excesivo.
    # 'auth_audit:3' habilita logs específicos de auditoría de autenticación.
    log level = 1 auth_audit:3
    
    # Archivo específico para logs
    log file = /var/log/samba/audit.log
    
    # Rotación de logs al llegar a 1MB
    max log size = 1024
```

**Interpretación:** Esto permite detectar intentos de fuerza bruta, accesos desde IPs no autorizadas o errores de Kerberos (`NT_STATUS_WRONG_PASSWORD`, `NT_STATUS_LOGON_FAILURE`).





























# Unidad 3: Administración y Acceso Remoto (SSH, VNC, RDP)

Este documento sirve como manual técnico y guía de estudio para la administración remota de sistemas. Abarca desde la línea de comandos segura con **OpenSSH**, hasta la gestión de escritorios gráficos multiplataforma con **VNC** y la implementación empresarial de **RDP** en entornos Active Directory.

---

## 📑 Índice de Contenidos

1. [Protocolo SSH (Secure Shell)](#1-protocolo-ssh-secure-shell)
    - [1.1. Fundamentos Teóricos y Criptografía](#11-fundamentos-teóricos-y-criptografía)
    - [1.2. Gestión de Claves (PKI) y `known_hosts`](#12-gestión-de-claves-pki-y-known_hosts)
    - [1.3. Configuración del Cliente (`config`) y Agente](#13-configuración-del-cliente-config-y-agente)
    - [1.4. Bastionado del Servidor](#14-bastionado-del-servidor)
    - [1.5. Herramientas Adicionales: SCP, SFTP y SSHFS](#15-herramientas-adicionales-scp-sftp-y-sshfs)
2. [Protocolo VNC (Virtual Network Computing)](#2-protocolo-vnc-virtual-network-computing)
    - [2.1. Teoría del Protocolo RFB](#21-teoría-del-protocolo-rfb)
    - [2.2. Implementación en Linux (TigerVNC)](#22-implementación-en-linux-tigervnc)
    - [2.3. Implementación en Windows](#23-implementación-en-windows)
3. [Protocolo RDP (Remote Desktop Protocol)](#3-protocolo-rdp-remote-desktop-protocol)
    - [3.1. Teoría y Características](#31-teoría-y-características)
    - [3.2. Habilitación Básica](#32-habilitación-básica)
    - [3.3. Despliegue Empresarial con GPO](#33-despliegue-empresarial-con-gpo)
    - [3.4. Seguridad Avanzada con Certificados (PKI)](#34-seguridad-avanzada-con-certificados-pki)
    - [3.5. Firma de Archivos .rdp](#35-firma-de-archivos-rdp)

---

## 1. Protocolo SSH (Secure Shell)

### 1.1. Fundamentos Teóricos y Criptografía
SSH es el estándar *de facto* para la administración remota de sistemas UNIX/Linux, reemplazando a protocolos inseguros como Telnet o Rlogin. Funciona sobre el puerto **22 TCP**.

* **Arquitectura:** Cliente-Servidor. El demonio `sshd` escucha en el servidor y el cliente `ssh` inicia la conexión.
* **Cifrado:** Todo el tráfico está cifrado (simétrico para la sesión, asimétrico para el intercambio de claves).
* **Integridad:** Garantiza que los datos no han sido alterados en tránsito.

### 1.2. Gestión de Claves (PKI) y `known_hosts`
SSH utiliza criptografía de clave pública (asimétrica) para dos propósitos fundamentales:

1.  **Identidad del Servidor (Host Key):** La primera vez que conectas, el servidor envía su clave pública. El cliente calcula la "huella digital" (fingerprint) y pide confirmación. Si se acepta, se guarda en `~/.ssh/known_hosts`.
    * *Protección MITM:* Si la clave del servidor cambia (por reinstalación o ataque Man-in-the-Middle), el cliente bloqueará la conexión con una alerta de seguridad severa.
    * *Comandos:*
        ```bash
        # Borrar una entrada obsoleta de un servidor
        ssh-keygen -R 192.168.1.150
        
        # Ver la huella digital de un servidor remoto
        ssh-keyscan 192.168.1.150
        ```

2.  **Identidad del Usuario (User Key):** Permite autenticación sin contraseña.
    * **Tipos de Clave:**
        * **RSA:** Estándar antiguo. Requiere 2048 o 4096 bits para ser segura.
        * **Ed25519:** Estándar moderno (Curva Elíptica). Más segura y rápida con claves más pequeñas. Recomendada.
        * **ECDSA:** Otra variante de curva elíptica.

**Generación y Copia de Claves:**
```bash
# Generar par de claves (Ed25519 recomendada)
ssh-keygen -t ed25519 -C "comentario_identificativo"

# Copiar la clave PÚBLICA al servidor (autoriza el acceso)
ssh-copy-id -i ~/.ssh/id_ed25519.pub usuario@servidor
````

### 1.3. Configuración del Cliente (`config`) y Agente

Para evitar memorizar IPs, usuarios y rutas de claves, se usa el archivo `~/.ssh/config`.

**Ejemplo de `~/.ssh/config`:**

```ssh
Host miServidor
    Hostname 192.168.1.150
    User alumno
    IdentityFile ~/.ssh/clave_especial
    Port 2222
```

*Ahora basta con escribir `ssh miServidor`.*

**Agente SSH (`ssh-agent`):**
Si protegemos nuestra clave privada con una contraseña (passphrase) —práctica recomendada—, tendríamos que escribirla en cada conexión. El agente guarda la clave descifrada en memoria durante la sesión.

```bash
# Iniciar agente (si no está activo)
eval $(ssh-agent -s)

# Añadir clave al agente
ssh-add ~/.ssh/id_ed25519
```

### 1.4. Bastionado del Servidor

El archivo `/etc/ssh/sshd_config` controla la seguridad del servicio. Prácticas esenciales de hardening:

| Directiva | Configuración Recomendada | Explicación |
| :--- | :--- | :--- |
| `PermitRootLogin` | `no` o `prohibit-password` | Evita ataques de fuerza bruta directos a root. |
| `PasswordAuthentication` | `no` | Obliga a usar claves PKI, eliminando el riesgo de contraseñas débiles. |
| `AllowUsers` | `usuario1 usuario2` | Lista blanca: solo estos usuarios pueden conectar. |
| `Port` | `2222` (Ejemplo) | Seguridad por oscuridad (reduce el ruido de bots, pero no es una medida fuerte real). |

### 1.5. Herramientas Adicionales: SCP, SFTP y SSHFS

SSH no es solo una terminal; es un túnel seguro para transporte de datos.

  * **SCP/SFTP:** Transferencia de archivos. `scp archivo usuario@destino:/ruta`.
  * **X11 Forwarding:** Permite ejecutar aplicaciones gráficas en el servidor y verlas en el cliente. Requiere `X11Forwarding yes` en el servidor y conectar con `ssh -X`.
  * **SSHFS (Sistema de Archivos):** Monta un directorio remoto vía SSH usando FUSE. No requiere configuración especial en el servidor (solo acceso SFTP).
    ```bash
    sshfs usuario@servidor:/ruta/remota /punto/de/montaje/local
    ```

-----

## 2\. Protocolo VNC (Virtual Network Computing)

### 2.1. Teoría del Protocolo RFB

VNC utiliza el protocolo **RFB (Remote Frame Buffer)**. Funciona a nivel de píxel: el servidor envía rectángulos de la pantalla (bitmaps) al cliente, y el cliente envía eventos de teclado y ratón.

  * **Características:** Es multiplataforma y "tonto" (no entiende de ventanas o gráficos, solo píxeles).
  * **Seguridad:** El protocolo RFB original **no cifra** el tráfico. Las contraseñas y la pantalla viajan en texto claro. Se recomienda encarecidamente usarlo a través de un túnel SSH o VPN.
  * **Puertos:** 5900 + N (donde N es el número de display). Display :0 usa 5900, Display :1 usa 5901.

### 2.2. Implementación en Linux (TigerVNC)

En Linux existen dos modos de funcionamiento principales:

1.  **Clonación de Sesión (`x0vncserver`):**
    Comparte la pantalla física que el usuario está viendo en el monitor. Similar a TeamViewer.

    ```bash
    # Crear contraseña VNC
    vncpasswd
    # Iniciar servidor para la sesión actual
    x0vncserver -SecurityTypes=VncAuth -PasswordFile ~/.vnc/passwd
    ```

2.  **Sesión Virtual Independiente (`vncserver`):**
    Crea un escritorio nuevo y separado en memoria, invisible en el monitor físico. Útil para múltiples usuarios concurrentes. Se gestiona como un servicio de systemd (`vncserver@:1.service`).

### 2.3. Implementación en Windows

En Windows, VNC actúa capturando el escritorio del usuario. Implementaciones como **TigerVNC Server** o **RealVNC** se instalan como servicio.

  * Al conectar, si nadie ha iniciado sesión, VNC muestra la pantalla de login de Windows.
  * Es necesario abrir explícitamente el puerto **5900** en el Firewall de Windows.

-----

## 3\. Protocolo RDP (Remote Desktop Protocol)

### 3.1. Teoría y Características

RDP es un protocolo propietario de Microsoft, mucho más sofisticado que VNC. No envía solo píxeles, sino instrucciones de dibujo (primitivas gráficas), lo que lo hace más eficiente en ancho de banda.

  * **Redirección de Dispositivos:** Permite usar impresoras, portapapeles, audio y unidades de disco locales dentro de la sesión remota.
  * **Puerto:** 3389 (TCP y UDP).
  * **Seguridad (NLA):** *Network Level Authentication*. Autentica al usuario antes de crear la sesión gráfica completa, mitigando ataques de DoS y ahorrando recursos.

### 3.2. Habilitación Básica

Por defecto, RDP está desactivado en Windows.

  * **GUI:** Configuración \> Sistema \> Escritorio Remoto \> Activar.
  * **PowerShell:**
    ```powershell
    Set-ItemProperty -Path 'HKLM\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
    Enable-NetFirewallRule -DisplayGroup "Escritorio Remoto"
    ```

### 3.3. Despliegue Empresarial con GPO

En un dominio Active Directory, no se activa RDP máquina por máquina. Se usan **Directivas de Grupo (GPO)** vinculadas a Unidades Organizativas (OU).

**Rutas de GPO Clave:**

1.  *Configuración del equipo \> Plantillas administrativas \> Componentes de Windows \> Servicios de escritorio remoto \> Host de sesión... \> Conexiones:*
      * **Permitir que los usuarios se conecten de forma remota.**
2.  *... \> Seguridad:*
      * **Requerir autenticación de usuario mediante NLA.**
3.  *Configuración del equipo \> Configuración de Windows \> Configuración de seguridad \> Firewall de Windows:*
      * Crear regla de entrada predefinida para **Escritorio Remoto**.

### 3.4. Seguridad Avanzada con Certificados (PKI)

Por defecto, RDP usa un certificado autofirmado, lo que provoca que el cliente muestre una advertencia de "Identidad no verificada". Para eliminar esto y asegurar la identidad del servidor:

1.  **Infraestructura:** Se requiere una Autoridad de Certificación (CA) empresarial en el dominio.
2.  **Plantilla de Certificado:** Se duplica la plantilla "Equipo" o "Servidor Web" para crear una específica para "Autenticación de Escritorio Remoto".
3.  **GPO de Auto-enrollment:** Se configura una GPO para que los servidores RDP soliciten automáticamente este certificado a la CA.
4.  **GPO de Configuración RDP:**
      * Ruta: *Configuración del equipo \> ... \> Host de sesión \> Seguridad \> Plantilla de certificado de autenticación de servidor*.
      * Se especifica el **nombre exacto** de la plantilla creada.

### 3.5. Firma de Archivos .rdp

Para evitar que los usuarios modifiquen los archivos de conexión (`.rdp`) o para asegurar que provienen de una fuente confiable (evitando la alerta de "Editor desconocido"):

1.  Se obtiene el hash del certificado de confianza (Thumbprint).
2.  Se firma el archivo `.rdp` usando la herramienta `rdpsign`:
    ```powershell
    rdpsign.exe /sha256 <HASH_DEL_CERTIFICADO> archivo.rdp
    ```
3.  En el cliente, mediante GPO, se deben especificar las huellas digitales de los "publicadores .rdp de confianza" para que acepten estos archivos silenciosamente (SSO o confianza explícita).

