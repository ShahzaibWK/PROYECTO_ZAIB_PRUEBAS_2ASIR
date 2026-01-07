# Documentación del Proyecto WordPress: Connectix

# 1. Tema e Introducción
El sitio web ha sido desarrollado utilizando WordPress como CMS. El objetivo es ofrecer una plataforma para la venta de dispositivos de red (Routers, GPS) y servicios de logística.

* **Tema Activo:** Bizora
* **Estilo Visual:** Se ha buscado un diseño limpio y corporativo, utilizando los colores de la marca (Azules/Grises) para transmitir confianza y tecnología.
* **Adaptabilidad:** El sitio es completamente *Responsive* (adaptable a móviles y tablets).

---

## 2. Usuarios Creados
Se han configurado diferentes roles para la gestión del sitio:

| Usuario | Rol
| :--- | :--- |
| **admin** | Administrador |
| **grupo3** | Administrador |
| **editor1** | Editor |
| **editor2** | Editor |

![](../../../../imágenes/wordpress/wp_1.png)
---

## 3. Plugins Utilizados
Selección de herramientas instaladas para extender la funcionalidad base:

### Herramientas del Sistema
* **UpdraftPlus - Backup/Restore:**
    * Utilizado para realizar copias de seguridad completas (Base de datos + Archivos).
    * Clave para la migración entre entornos (Localhost XAMPP ↔ Servidor Proxmox).
* **Smush:**
    * Optimización automática de imágenes. Reduce el peso de las fotos subidas para mejorar la velocidad de carga sin perder calidad visible.
* **Weglot Translate:**
    * Implementación de sitio multilingüe. Permite la traducción automática, en este caso al Inglés.

### Funcionalidad Comercial
* **WooCommerce:**
    * El motor de la tienda online. Gestiona productos, pasarelas de pago, envíos y cuentas de clientes.

### Interacción
* **WP-Polls:**
    * Sistema de encuestas integrado. Utilizado para recoger feedback de los usuarios sobre productos o servicios (ej: "¿Qué valoras más en un router?").
* **WPForms:**
    * Generador de formularios de contacto. Implementado en la página de "Contacto" y soporte técnico.

![](../../../../imágenes/wordpress/wp_2.png)
---

## 4. Páginas (Estructura Web)
El sitio cuenta con las siguientes páginas estáticas principales:

1.  **Inicio (Home)**
2.  **Sobre Nosotros:** Información corporativa, misión , visión de Connectix y nuestro equipo.
    * **Nuestro Equipo:** Miembros del equipo.
3.  **Servicios:** Catálogo completo generado por WooCommerce.
    * WooComerce crear páginas por defecto como: carrito, finalizar compra, mi cuenta y Política de devoluciones y reembolsos.
4.  **Blog:** Sección de noticias y artículos externos.
5.  **Contacto:** Formulario de dudas, datos de contacto e ubicación.
6. **Encuesta:** Encuesta para ver opniones de clientes.

![](../../../../imágenes/wordpress/wp_3.png)
---

## 5. Entradas, Etiquetas y Categorías
Estructura del Blog para mejorar el SEO y la organización del contenido:

### Categorías
* **Logística y Seguridad:** Noticias sobre el sector transporte.
* **Optimización Logística:** Novedades sobre hardware, 5G y conectividad.

### Etiquetas (Tags)
* `#ERP`, `#GPS`, `#Hardware`, `#Ciberseguridad`, `#Ahorro de costes`.

![](../../../../imágenes/wordpress/wp_4.png)
![](../../../../imágenes/wordpress/wp_5.png)
![](../../../../imágenes/wordpress/wp_6.png)

---

## 6. Tienda Online: WooCommerce (Configuración)
Se ha realizado la instalación, configuración y comprobación completa del plugin WooCommerce con los siguientes parámetros:

### Ajustes Generales
* **Información de la tienda:** Definida ubicación física y datos de facturación de Connectix.
* **Moneda:** Configurada en Euros (€), con el separador de miles y decimales correspondiente.
![](../../../../imágenes/wordpress/wp_7.png)
![](../../../../imágenes/wordpress/wp_8.png)

### Métodos de Pago
Se han habilitado y comprobado tres pasarelas de pago offline:
1.  **Transferencia bancaria directa.**
2.  **Contra reembolso.**
3.  **Pagos por cheque.**
![](../../../../imágenes/wordpress/wp_9.png)

### Métodos de Envío
Configuración de zonas de envío con las siguientes tarifas:
* **Tarifa plana:** Coste fijo de envío estándar a 10€.
* **Envío gratuito:** Configurado automáticamente al superar un pedido mínimo de 500€ establecido.
![](../../../../imágenes/wordpress/wp_10.png)
![](../../../../imágenes/wordpress/wp_11.png)
![](../../../../imágenes/wordpress/wp_12.png)
![](../../../../imágenes/wordpress/wp_13.png)


### Impuestos
* **Configuración fiscal:** Habilitado el cálculo de impuestos automáticos.
* **Tipos:** Implementado el **IVA General del 21%** para todos los productos.
![](../../../../imágenes/wordpress/wp_14.png)

### Correos Electrónicos (Emails)
Se ha personalizado el sistema de notificaciones transaccionales:
* **Flujo comprobado:** Avisos de "Nuevo Pedido" (al administrador) y "Pedido Completado" (al cliente).
* **Identidad:** Personalización del "Remitente", inclusión del **Logo** de Connectix y ajuste del **Color base** de la plantilla para coincidir con la marca.
![](../../../../imágenes/wordpress/wp_15.png)
![](../../../../imágenes/wordpress/wp_16.png)
![](../../../../imágenes/wordpress/wp_17.png)

### Ajustes de Productos e Inventario
* **Datos físicos:** Configurados pesos y dimensiones (medidas) en los productos para cálculos logísticos.
* **Stock:** Gestión de inventario activada (control de existencias y avisos de stock bajo).
* **Social:** Activada la opción de dejar **valoraciones y estrellas** en las fichas de producto.
![alt text](../../../../imágenes/wordpress/wp_18.png)

### Catálogo de Productos
Se han creado y publicado al menos **4 productos vendibles** de prueba, incluyendo:
* Imágenes destacadas.
* Descripciones cortas y largas.
* Precios (con IVA incluido/desglosado).
* SKUs y niveles de stock.

1. **Servidor Connectix Hub - Enterprise**
![](../../../../imágenes/wordpress/wp_19.png)
![](../../../../imágenes/wordpress/wp_20.png)
![](../../../../imágenes/wordpress/wp_21.png)
![](../../../../imágenes/wordpress/wp_22.png)

2. **Servidor Connectix Hub - Enterprise**
![](../../../../imágenes/wordpress/wp_23.png)
![](../../../../imágenes/wordpress/wp_24.png)
![](../../../../imágenes/wordpress/wp_25.png)
![](../../../../imágenes/wordpress/wp_26.png)

3. **Servidor Connectix Hub - Enterprise**
![](../../../../imágenes/wordpress/wp_27.png)
Al no ser un producto físico, marcamos la opción de virtual.
![](../../../../imágenes/wordpress/wp_28.png)
![](../../../../imágenes/wordpress/wp_29.png)

4. **Servidor Connectix Hub - Enterprise**
![](../../../../imágenes/wordpress/wp_30.png)
Al no ser un producto físico, marcamos la opción de virtual.
![](../../../../imágenes/wordpress/wp_31.png)
![](../../../../imágenes/wordpress/wp_32.png)