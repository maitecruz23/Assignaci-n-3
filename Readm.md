# 🖥️ RDP RemoteApp + IIS Web Publishing — Windows Server 2019

> **Laboratorio de Servicios de Escritorio Remoto:** Configuración e implementación de RDP RemoteApp y Web Client en Windows Server 2019 para publicar una página web personalizada en IIS, accesible desde un cliente Windows 10 unido al dominio.


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






  <img width="2019" height="1254" alt="image" src="https://github.com/user-attachments/assets/f0f18050-7a65-45bb-a8d1-e5c612b753c1" />


  
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

- Se creó el archivo `C:\inetpub\wwwroot\index.html` con contenido HTML personalizado.
- IIS 10 sirve la página en `http://localhost` en el servidor.

### 3. Colección RemoteApp

- Se creó la colección **QuickSessionCollection** (tipo: Sesión).
- Se publicó **Internet Explorer (`iexplore`)** como programa RemoteApp apuntando a `http://localhost`.
- Recursos adicionales publicados: Calculadora, Paint, WordPad.

### 4. Solución de Certificado SSL

El servidor utiliza un **certificado SSL autofirmado**. Para permitir el acceso desde el cliente:

1. Se exportó el certificado del servidor.
2. Se instaló manualmente en el cliente en **"Entidades de certificación raíz de confianza"**.
3. Esto habilitó tanto el portal RDWeb (HTTPS) como la suscripción por Web Feed.

### 5. Resolución DNS en el Cliente

Se editó el archivo `hosts` del cliente (`C:\Windows\System32\drivers\etc\hosts`):
```
192.168.106.106    WIN-ICJ1EKEE5K0.miempresa.local
```

### 6. Acceso Método 1 — RDWeb (Web Client)

- URL: `https://192.168.106.106/RDWeb`
- Credenciales: `miempresa\usuario` + contraseña del dominio.

### 7. Acceso Método 2 — Web Feed (Panel de Control)

- Panel de Control → RemoteApp y Conexiones de Escritorio.
- URL: `https://WIN-ICJ1EKEE5K0.miempresa.local/RDWeb/Feed/webfeed.aspx`

---

## 📸 Capturas de Pantalla

### 4.1 — Portal RDWeb: Pantalla de Login






**Descripción:** Pantalla de inicio de sesión del portal RDWeb desde el cliente Windows 10. Se ingresa con formato `Dominio\nombreDeUsuario`.

---

### 4.2 — Portal RDWeb: Opciones de Seguridad

<img width="808" height="612" alt="image" src="https://github.com/user-attachments/assets/c28e672d-ceb8-46e0-9f30-01692bc40096" />

**Descripción:** Vista del login con opciones de equipo público o privado. URL visible: `https://192.168.106.106/RDWeb/Pages/es-ES/login.aspx`.

---

### 4.3 — Portal RDWeb: Aplicaciones Publicadas
<img width="824" height="596" alt="image" src="https://github.com/user-attachments/assets/bca5bc60-6872-4e56-9c8e-5914bc4e83b8" />



**Descripción:** Vista post-login mostrando las RemoteApps publicadas: Calculadora, iexplore, Paint y WordPad.

---

### 4.4 — Página IIS Consumida Remotamente
<img width="826" height="958" alt="image" src="https://github.com/user-attachments/assets/2b7ae184-cd75-47e5-a4f4-1ec8b74d6eac" />


**Descripción:** Internet Explorer abierto vía RemoteApp mostrando la página `index.html` de IIS: "Bienvenido al Portal RemoteApp".

---

### 4.5 — Archivo index.html en wwwroot
<img width="807" height="759" alt="image" src="https://github.com/user-attachments/assets/566b05e2-93f0-4199-aaf4-ea213ce3184d" />


**Descripción:** Explorador de archivos del servidor mostrando `index.html` en `C:\inetpub\wwwroot`.

---

### 4.6 — Administrador del Servidor: Roles Instalados
<img width="761" height="569" alt="image" src="https://github.com/user-attachments/assets/1cadfb9f-8fba-46d0-a4d8-9fe224dd8ba8" />


**Descripción:** Panel del Administrador del Servidor con los roles AD DS, DNS, IIS, NPAS y RDS activos.

---

### 4.7 — IIS Manager
<img width="1062" height="766" alt="image" src="https://github.com/user-attachments/assets/74a07c25-623b-4292-a24b-0d4d00c94bd7" />


**Descripción:** Consola IIS 10 con el sitio Default Web Site activo en `WIN-ICJ1EKEE5K0`.

---

### 4.8 — Colección QuickSessionCollection
<img width="849" height="533" alt="image" src="https://github.com/user-attachments/assets/4dbbf8c5-8703-44b3-94ea-5d15e5cce6be" />


**Descripción:** Colección RemoteApp configurada con FQDN `WIN-ICJ1EKEE5K0.miempresa.local`.

---

### 4.9 — Servidor y Cliente en Paralelo
<img width="836" height="516" alt="image" src="https://github.com/user-attachments/assets/39ec7abe-4440-40de-9163-e4aad5b47800" />

**Descripción:** Ambas VMs corriendo simultáneamente en Oracle VirtualBox.

---

## 🔧 Resolución de Problemas

| Problema | Causa | Solución |
|---|---|---|
| Error de certificado en RDWeb | Certificado SSL autofirmado | Instalar certificado en "Entidades de certificación raíz de confianza" del cliente |
| Web Feed no suscribe | FQDN no resuelto | Agregar entrada manual en archivo `hosts` |
| RDWeb requiere modo IE | Plugins ActiveX | Abrir en Edge modo Internet Explorer |
| Aplicación no abre | Bloqueo de archivo `.rdp` | Permitir descarga y confirmar ejecución |

---

## 👤 Autor

**Estudiante:** maitecruz23  
**Fecha:** Marzo 2026
