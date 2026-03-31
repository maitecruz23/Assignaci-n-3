# 🖥️ RDP RemoteApp + IIS Web Publishing — Windows Server 2019

> **Laboratorio de Servicios de Escritorio Remoto:** Configuración e implementación de RDP RemoteApp y Web Client en Windows Server 2019 para publicar una página web personalizada en IIS, accesible desde un cliente Windows 10 unido al dominio.

---

## 📋 Tabla de Contenidos

- [Objetivo](#-objetivo)
- [Topología de Red](#-topología-de-red)
- [Tecnologías y Roles Utilizados](#-tecnologías-y-roles-utilizados)
- [Configuraciones Implementadas](#-configuraciones-implementadas)
- [Capturas de Pantalla](#-capturas-de-pantalla)
- [Resolución de Problemas](#-resolución-de-problemas)
- [Estructura del Repositorio](#-estructura-del-repositorio)

---

## 🎯 Objetivo

Configurar e implementar los **Servicios de Escritorio Remoto (RDS)** en Windows Server 2019 para:

1. Instalar y configurar el rol de **RDP RemoteApp** (basado en sesión).
2. Habilitar el **Web Client de RDP RemoteApp** (portal RDWeb).
3. Crear y alojar una **página web personalizada en IIS**.
4. **Publicar la página IIS** como programa RemoteApp accesible remotamente.
5. Consumir la página desde un cliente Windows 10 mediante **dos métodos**: Web Client (RDWeb) y Web Feed (integración en Panel de Control).

---

## 🌐 Topología de Red

```
┌─────────────────────────────────────────────────────────────────┐
│                      Red NAT / Host-Only                        │
│                    Subred: 192.168.106.0/24                     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────┐
          │                                 │
┌─────────▼──────────┐           ┌──────────▼──────────┐
│   WINDOWS SERVER   │           │    WINDOWS 10        │
│   2019 Standard    │           │    (Cliente VM)      │
├────────────────────┤           ├─────────────────────┤
│ IP: 192.168.106.106│◄──RDP────►│ IP: 192.168.106.107 │
│ Dominio: miempresa │           │ Unida al dominio     │
│ .local             │           │ miempresa.local      │
│ Host: WIN-ICJ1E... │           │                      │
├────────────────────┤           ├─────────────────────┤
│ Roles instalados:  │           │ Acceso via:          │
│ • AD DS            │           │ • RDWeb (HTTPS)      │
│ • DNS              │           │ • Web Feed (Panel    │
│ • IIS              │           │   de Control)        │
│ • RDS (Session)    │           │                      │
│ • NPAS             │           │                      │
└────────────────────┘           └─────────────────────┘
```

### Direccionamiento IP

| Dispositivo | Rol | Dirección IP | Hostname | Dominio |
|---|---|---|---|---|
| Windows Server 2019 | Servidor RDS / AD DS / IIS | `192.168.106.106` | `WIN-ICJ1EKEE5K0` | `miempresa.local` |
| Windows 10 (VM Cliente) | Cliente de dominio | `192.168.106.107` | — | `miempresa.local` |

---

## 🛠️ Tecnologías y Roles Utilizados

| Componente | Descripción |
|---|---|
| **Windows Server 2019 Standard** | Sistema operativo del servidor |
| **Active Directory Domain Services (AD DS)** | Directorio y autenticación de dominio |
| **DNS Server** | Resolución de nombres internos |
| **Internet Information Services (IIS) 10** | Servidor web para alojar `index.html` |
| **Remote Desktop Services (RDS)** | Implementación de escritorio basado en sesión |
| **RD Web Access** | Portal web para acceder a RemoteApps |
| **RD Session Host** | Host donde se ejecutan las aplicaciones remotas |
| **RD Connection Broker** | Gestión de conexiones y colecciones |
| **Oracle VirtualBox** | Plataforma de virtualización |

---

## ⚙️ Configuraciones Implementadas

### 1. Instalación de Roles en el Servidor

Se instalaron los siguientes roles desde el **Administrador del Servidor**:

- Active Directory Domain Services (AD DS)
- DNS Server
- Internet Information Services (IIS)
- Network Policy and Access Services (NPAS)
- **Remote Desktop Services** → Implementación basada en sesión:
  - RD Web Access
  - RD Session Host
  - RD Connection Broker

### 2. Configuración de IIS y Página Personalizada

- Se creó el archivo `C:\inetpub\wwwroot\index.html` con contenido HTML personalizado ("Bienvenido al Portal RemoteApp").
- IIS 10 sirve la página en `http://localhost` en el servidor.

### 3. Colección RemoteApp

- Se creó la colección **QuickSessionCollection** (tipo: Sesión).
- Se publicó **Internet Explorer (`iexplore`)** como programa RemoteApp, apuntando a `http://localhost` para mostrar la página de IIS.
- Recursos adicionales publicados: Calculadora, Paint, WordPad.

### 4. Solución de Certificado SSL

El servidor utiliza un **certificado SSL autofirmado**. Para permitir el acceso desde el cliente:

1. Se exportó el certificado del servidor.
2. Se instaló manualmente en el cliente en **"Entidades de certificación raíz de confianza"**.
3. Esto habilitó tanto el portal RDWeb (HTTPS) como la suscripción por Web Feed.

### 5. Resolución DNS en el Cliente

Se editó el archivo `hosts` del cliente (`C:\Windows\System32\drivers\etc\hosts`) para resolver el FQDN del servidor:

```
192.168.106.106    WIN-ICJ1EKEE5K0.miempresa.local
```

### 6. Acceso Método 1 — RDWeb (Web Client)

- URL de acceso: `https://192.168.106.106/RDWeb`
- Credenciales: `miempresa\<usuario>` + contraseña del dominio.
- El cliente accede al portal, hace clic en la aplicación publicada y se abre remotamente en su equipo.

### 7. Acceso Método 2 — Web Feed (Panel de Control)

- Panel de Control → **RemoteApp y Conexiones de Escritorio**.
- URL del feed: `https://WIN-ICJ1EKEE5K0.miempresa.local/RDWeb/Feed/webfeed.aspx`
- Las aplicaciones se integran en el menú de inicio del cliente.

---

## 📸 Capturas de Pantalla

### 4.1 — Portal RDWeb: Pantalla de Login (IE Mode)

![Login RDWeb](screenshots/01_rdweb_login.png)

**Descripción:** Pantalla de inicio de sesión del portal **Acceso web de RD** (`RDWeb`) accedida desde el cliente Windows 10. Se accede mediante el navegador con la URL `https://192.168.106.106/RDWeb`. El cliente debe ingresar con el formato `Dominio\nombreDeUsuario` y la contraseña del dominio. Se observa la advertencia de seguridad sobre el cumplimiento de la directiva organizacional.

---

### 4.2 — Portal RDWeb: Login con Opciones de Equipo

![Login con opciones](screenshots/02_rdweb_login_opciones.png)

**Descripción:** Vista del portal de login con las opciones de seguridad visibles: **"Éste es un equipo público o compartido"** / **"Éste es un equipo privado"**. La URL en la barra muestra el acceso mediante Edge en modo Internet Explorer (`https://192.168.106.106/RDWeb/Pages/es-ES/login.aspx`).

---

### 4.3 — Portal RDWeb: Aplicaciones Publicadas

![Apps publicadas](screenshots/03_rdweb_apps.png)

**Descripción:** Vista post-login del portal RDWeb mostrando las **aplicaciones RemoteApp publicadas**: Calculadora, iexplore, Paint y WordPad. El menú superior ofrece las opciones "RemoteApp y escritorios" y "Conectarse a un equipo remoto". Las aplicaciones se ejecutan de forma remota en el servidor al hacer clic en ellas.

---

### 4.4 — Página IIS Consumida Remotamente (Web Feed)

![Página IIS remota](screenshots/04_iis_pagina_remota.png)

**Descripción:** Captura tomada desde el cliente Windows 10 mostrando el navegador **Internet Explorer abierto de forma remota vía RemoteApp**, visualizando la página `index.html` alojada en IIS del servidor: "**Bienvenido al Portal RemoteApp**". En segundo plano se ve el portal RDWeb. Esto demuestra la publicación exitosa de la página IIS a través de RemoteApp.

---

### 4.5 — Servidor Windows Server: Explorador con wwwroot

![wwwroot IIS](screenshots/05_servidor_wwwroot.png)

**Descripción:** Vista del **Explorador de archivos en el servidor Windows Server 2019** mostrando el archivo `index.html` en `C:\inetpub\wwwroot`, confirmando la creación del archivo HTML personalizado que es servido por IIS.

---

### 4.6 — Administrador del Servidor: Roles Instalados

![Server Manager](screenshots/06_server_manager.png)

**Descripción:** Panel del **Administrador del Servidor** en Windows Server 2019 mostrando los roles instalados: AD DS, DNS, IIS, NPAS y Servicios de Escritorio Remoto. Se visualizan los paneles de estado de AD DS y DNS en verde (operativos).

---

### 4.7 — IIS Manager: Sitio Web Predeterminado

![IIS Manager](screenshots/07_iis_manager.png)

**Descripción:** Consola **IIS 10 (Internet Information Services Manager)** mostrando el servidor `WIN-ICJ1EKEE5K0` con el sitio **Default Web Site** activo. Confirma que IIS está instalado y configurado correctamente para servir la página `index.html`.

---

### 4.8 — Colección RemoteApp: QuickSessionCollection

![Colección RDS](screenshots/08_coleccion_rds.png)

**Descripción:** Vista de **Servicios de Escritorio Remoto → Colecciones** en el Administrador del Servidor. Se muestra la colección **QuickSessionCollection** de tipo Sesión con el recurso "Programas RemoteApp". En la sección de Conexiones se confirma el FQDN del servidor: `WIN-ICJ1EKEE5K0.miempresa.local`.

---

### 4.9 — Vista Completa: Servidor y Cliente en Paralelo

![Servidor y Cliente](screenshots/09_server_cliente_paralelo.png)

**Descripción:** Vista completa del entorno de virtualización con **ambas máquinas corriendo en paralelo** en Oracle VirtualBox: a la izquierda Windows Server 2019 y a la derecha el cliente Windows 10. Muestra la topología real del laboratorio con ambas VMs operativas simultáneamente.

---

## 🔧 Resolución de Problemas

| Problema | Causa | Solución Aplicada |
|---|---|---|
| Portal RDWeb muestra error de certificado | Certificado SSL autofirmado no confiable | Exportar e instalar el certificado en "Entidades de certificación raíz de confianza" del cliente |
| Web Feed no se puede suscribir | FQDN no resuelto en el cliente | Agregar entrada manual en `hosts`: `192.168.106.106 WIN-ICJ1EKEE5K0.miempresa.local` |
| RDWeb requiere modo IE | Funciones ActiveX/plugins de RDWeb | Abrir en Edge en modo Internet Explorer o usar IE directamente |
| Aplicación no abre desde RDWeb | Bloqueo de descarga de archivo `.rdp` | Permitir descarga en el navegador y confirmar la ejecución |

---

## 📁 Estructura del Repositorio

```
rdp-remoteapp-iis-lab/
│
├── README.md                          # Documentación principal (este archivo)
│
├── screenshots/                       # Capturas de pantalla del laboratorio
│   ├── 01_rdweb_login.png
│   ├── 02_rdweb_login_opciones.png
│   ├── 03_rdweb_apps.png
│   ├── 04_iis_pagina_remota.png
│   ├── 05_servidor_wwwroot.png
│   ├── 06_server_manager.png
│   ├── 07_iis_manager.png
│   ├── 08_coleccion_rds.png
│   └── 09_server_cliente_paralelo.png
│
└── config/
    └── index.html                     # Página personalizada publicada en IIS
```

---

## 👤 Autor

**Estudiante:** [Tu Nombre]  
**Curso:** Administración de Sistemas / Redes  
**Fecha:** Marzo 2026  

---

> **Nota:** Este laboratorio fue realizado con fines educativos usando máquinas virtuales en Oracle VirtualBox. Las IPs y certificados son de entorno local.
