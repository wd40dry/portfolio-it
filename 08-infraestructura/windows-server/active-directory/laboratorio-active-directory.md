# 🖥️ Laboratorio: Active Directory Domain Services (AD DS)
### Cómo AD DS maneja una red empresarial

**Autor:** wd40dry  
**Fecha:** Mayo 2026  
**Entorno:** VirtualBox · Windows Server 2022 · Windows 10 Pro

---

## 📌 Introducción

Este documento refleja mi primer laboratorio práctico de Active Directory Domain Services (AD DS). El objetivo fue entender cómo una empresa real gestiona su red: usuarios, equipos, permisos y políticas — todo centralizado en un servidor llamado **Domain Controller (DC)**.

Lo construí desde cero usando máquinas virtuales, sin infraestructura física.

---

## 🧠 ¿Qué es Active Directory?

Active Directory es un servicio de Microsoft que permite a las organizaciones **gestionar identidades y accesos** dentro de una red. En lugar de configurar cada computadora por separado, AD centraliza todo en un servidor.

Conceptos clave:

- **Domain Controller (DC):** El servidor que administra el dominio. Autentica usuarios y aplica políticas.
- **Dominio:** El espacio lógico donde viven todos los objetos (usuarios, equipos, grupos). En este laboratorio: `lobito.local`.
- **Forest:** El contenedor de más alto nivel. Un forest puede tener varios dominios.
- **OU (Organizational Unit):** Carpetas que organizan los objetos del dominio, igual que departamentos en una empresa.
- **GPO (Group Policy Object):** Políticas que se aplican automáticamente a usuarios o equipos.
- **DNS:** Active Directory depende completamente del DNS para funcionar. El DC actúa como servidor DNS del dominio.

---

## 🏗️ Arquitectura del laboratorio

```
┌─────────────────────────────────────┐
│           VirtualBox Host           │
│                                     │
│  ┌─────────────────┐  ┌──────────┐  │
│  │ Windows Server  │  │ Win10PRO │  │
│  │ 2022 (DC01)     │  │ (Cliente)│  │
│  │ 192.168.1.1     │  │192.168.1.10│ │
│  └────────┬────────┘  └────┬─────┘  │
│           └──── intnet ────┘        │
└─────────────────────────────────────┘
```

**Red:** Internal Network (`intnet`) — red aislada entre VMs, sin acceso a internet.

### Especificaciones

| Componente | Detalle |
|---|---|
| Hipervisor | Oracle VirtualBox |
| Servidor | Windows Server 2022 Standard Evaluation |
| Cliente | Windows 10 Pro |
| Dominio | lobito.local |
| NetBIOS | LOBITO |
| IP del DC | 192.168.1.1 |
| IP del Cliente | 192.168.1.10 |
| DNS | El propio DC (127.0.0.1 en el servidor) |

---

## 🔧 Etapa 1 — Preparación del servidor

Antes de instalar AD DS, hay que dejar el servidor listo.

### 1.1 Cambiar el nombre del servidor

El servidor debe tener un nombre descriptivo. Lo renombré a `DC01` (Domain Controller 01) desde:

`Server Manager → Local Server → Computer name → Change`

### 1.2 Configurar IP estática

Un Domain Controller **siempre debe tener IP fija**. Si cambia de IP, los clientes no pueden encontrarlo.

Configuración aplicada en el adaptador Ethernet:

```
IP Address:      192.168.1.1
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
DNS Server:      127.0.0.1  ← apunta a sí mismo
```

> **¿Por qué DNS apunta a sí mismo?**  
> Porque el DC será el servidor DNS del dominio. Todos los clientes deben consultarle a él para resolver nombres como `lobito.local`.

### 1.3 Habilitar Remote Desktop (RDP)

Permite conectarse al servidor remotamente sin necesidad de la consola de VirtualBox.

`Server Manager → Local Server → Remote Desktop → Allow remote connections`

---

## 🏛️ Etapa 2 — Instalación de AD DS y promoción a Domain Controller

### 2.1 Instalar el rol AD DS

Desde `Server Manager → Add Roles and Features`:

- Tipo: **Role-based installation**
- Servidor destino: **DC01**
- Rol seleccionado: **Active Directory Domain Services**
- Se instalan automáticamente: Group Policy Management, herramientas de administración, módulo PowerShell para AD

### 2.2 Promover el servidor a Domain Controller

Una vez instalado el rol, aparece la opción **"Promote this server to a domain controller"**.

Configuración usada:

| Parámetro | Valor |
|---|---|
| Operación | Add a new forest |
| Nombre del dominio | lobito.local |
| NetBIOS | LOBITO |
| Forest Functional Level | Windows Server 2016 |
| DNS Server | Sí (instalado automáticamente) |
| Global Catalog | Sí |
| Contraseña DSRM | (contraseña de recuperación de emergencia) |

> **¿Qué es el DSRM?**  
> Directory Services Restore Mode. Es una contraseña especial para recuperar AD en caso de falla grave. Se usa arrancando el DC en modo de recuperación.

Tras la instalación el servidor se reinicia y queda como **Domain Controller de lobito.local**.

---

## 👥 Etapa 3 — Estructura del dominio: OUs, usuarios y grupos

Esta etapa simula cómo una empresa organiza su gente en AD.

### 3.1 Crear una Organizational Unit (OU)

Una OU es como un departamento dentro del dominio. Creé la OU `Empleados` para contener los usuarios del laboratorio.

`Active Directory Users and Computers → clic derecho en lobito.local → New → Organizational Unit`

### 3.2 Crear un usuario de dominio

Dentro de la OU `Empleados` creé el usuario:

| Campo | Valor |
|---|---|
| Nombre | Juan Perez |
| User logon name | jperez |
| Contraseña | Admin123! |
| Password never expires | Sí (para laboratorio) |

### 3.3 Crear un grupo de seguridad

Los grupos permiten asignar permisos a muchos usuarios a la vez.

| Campo | Valor |
|---|---|
| Nombre | GrupoEmpleados |
| Scope | Global |
| Type | Security |

Luego agregué `jperez` como miembro del grupo.

> **¿Por qué usar grupos?**  
> En una empresa real, no se asignan permisos a usuarios individuales sino a grupos. Si un empleado cambia de rol, simplemente se mueve de grupo.

---

## 💻 Etapa 4 — Agregar un cliente al dominio

### 4.1 Configurar red del cliente

La VM Win10PRO necesita comunicarse con el DC01. Configuré su IP estática:

```
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
DNS Server:      192.168.1.1  ← apunta al DC
```

> **¿Por qué el DNS del cliente apunta al DC?**  
> Para que pueda resolver `lobito.local`. Sin esto, el cliente no puede encontrar el dominio y no puede unirse.

### 4.2 Verificar conectividad

Desde el cliente, ping al DC:

```cmd
ping 192.168.1.1
```

Resultado: 4 paquetes enviados, 0 perdidos ✅

### 4.3 Unir el cliente al dominio

`sysdm.cpl → Change → Domain → lobito.local`

Se solicitaron credenciales del administrador del dominio (`Administrator`). Tras reiniciar, el equipo quedó unido a `lobito.local`.

### 4.4 Iniciar sesión con usuario del dominio

En la pantalla de login del Win10PRO, usando **Other user**:

```
Usuario:    LOBITO\jperez
Contraseña: Admin123!
```

✅ Juan Perez inició sesión correctamente en un equipo del dominio.

---

## 💡 Conclusiones: Cómo AD DS maneja una red empresarial

Después de construir este laboratorio entendí que Active Directory no es solo un "listado de usuarios". Es la **columna vertebral de la identidad en una red Windows**.

Cuando un usuario inicia sesión en cualquier equipo del dominio:

1. El equipo consulta al **DNS** para encontrar el Domain Controller
2. El DC **autentica** al usuario con su contraseña (protocolo Kerberos)
3. El DC verifica a qué **grupos** pertenece el usuario
4. Se aplican las **GPOs** correspondientes a su OU
5. El usuario accede solo a los recursos que tiene **permiso**

Todo esto ocurre en segundos, de forma transparente para el usuario.

En una empresa con 500 empleados, AD permite que:
- Un nuevo empleado tenga acceso a todo lo necesario el primer día
- Un empleado desvinculado pierda acceso instantáneamente deshabilitando su cuenta
- Las políticas de seguridad se apliquen automáticamente en todos los equipos

---

## 🔭 Próximos pasos

- [ ] Crear y aplicar GPOs defensivas (bloqueo de USB, restricción de software, políticas de contraseñas)
- [ ] Configurar Event Viewer para monitorear inicios de sesión y cambios en AD
- [ ] Instalar y configurar Sysmon para logging avanzado de eventos en Windows
- [ ] Centralizar logs con un SIEM básico (Wazuh o Splunk free tier)
- [ ] Simular incidentes y practicar detección y respuesta (IR)

---

## 📚 Recursos recomendados

- [Microsoft Learn — Active Directory](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [TryHackMe — Active Directory Basics](https://tryhackme.com/room/activedirectorybasics)
- [TryHackMe — SOC Level 1](https://tryhackme.com/path/outline/soclevel1)
- [Wazuh — SIEM open source](https://wazuh.com)

---

*Documento escrito como parte de mi camino autodidacta hacia Cloud Security Engineering y Blue Team.*  
*— wd40dry | Uruguay 🇺🇾*
