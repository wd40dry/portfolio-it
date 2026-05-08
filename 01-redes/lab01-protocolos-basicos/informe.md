# Informe — Lab 01: Protocolos Básicos de Red

**Alumno:** Damian Trinidad  
**Fecha:** 07/05/2026  
**Entorno:** Windows 11 — PowerShell — Red Wi-Fi (ISP: Antel, Uruguay)

---

## Entregable 1 — PING con análisis de TTL y latencia

**Comando ejecutado:**
```powershell
ping -n 5 8.8.8.8
```

**Resultados:**
- Paquetes enviados: 5 | Recibidos: 5 | Perdidos: 0 (0%)
- Latencia mínima: 15ms | Máxima: 18ms | Media: 16ms
- TTL recibido: 117

**Análisis:**

Se realizó ping con 5 paquetes a 8.8.8.8 obteniendo 0% de pérdida y una latencia promedio de 16ms. El TTL recibido fue 117, lo que indica que el paquete partió con TTL=128 (valor inicial de Windows en destino) y atravesó 11 routers antes de llegar (128 - 117 = 11 saltos). La conectividad es óptima.

---

## Entregable 2 — TRACEROUTE con identificación de saltos

**Comando ejecutado:**
```powershell
tracert 8.8.8.8
```

**Resultados:**

| Salto | IP / Hostname | Latencia | Observación |
|-------|--------------|----------|-------------|
| 1–6 | * * * | — | ICMP bloqueado por firewall del ISP |
| 7 | crl1i1.eze-ae1101.antel.net.uy | 14ms | Red Antel — salida por Ezeiza, Argentina |
| 8 | 74.125.50.205 | 20ms | Infraestructura Google |
| 9 | 74.125.50.x | 13ms | Infraestructura Google |
| 10 | 108.170.255.31 | 13ms | Infraestructura Google |
| 11 | 142.251.77.171 | 14ms | Infraestructura Google |
| 12 | dns.google [8.8.8.8] | 16ms | Destino alcanzado |

**Análisis:**

El traceroute hacia 8.8.8.8 mostró 12 saltos en total. Los primeros 6 no respondieron por tener ICMP bloqueado, comportamiento normal en redes de ISP. El salto 7 identificó la red de Antel con salida por Ezeiza, Argentina. Los saltos 8 al 11 corresponden a infraestructura interna de Google, alcanzando el destino dns.google en el salto 12 con 16ms de latencia.

---

## Entregable 3 — Consulta DNS con dig

**Comando ejecutado:**
```powershell
dig google.com MX
```

**Resultados:**
- Status: NOERROR
- Servidor consultado: 8.8.8.8 (Google DNS) vía UDP
- Registro MX: `google.com. 26 IN MX 10 smtp.google.com.`
- Query time: 22ms

**Análisis de campos:**

| Campo | Valor | Significado |
|-------|-------|-------------|
| status | NOERROR | Consulta exitosa |
| SERVER | 8.8.8.8 | Servidor DNS consultado |
| Transporte | UDP | DNS usa UDP por defecto (puerto 53) |
| TTL | 26 | Segundos que la respuesta es válida en caché |
| MX | 10 smtp.google.com | Prioridad 10, servidor que recibe correos @google.com |
| Query time | 22ms | Tiempo de respuesta del servidor DNS |

**Análisis:**

Se consultó el registro MX de google.com usando dig contra el servidor DNS 8.8.8.8 mediante UDP. La respuesta fue NOERROR con 1 registro: smtp.google.com con prioridad 10 y TTL de 26 segundos. El registro MX indica qué servidor recibe los correos destinados a ese dominio, usando el protocolo SMTP (capa 7).

---

## Entregable 4 — Tabla de protocolos

| Protocolo | Puerto(s) | Transporte | Capa OSI | Función |
|-----------|-----------|------------|----------|---------|
| HTTP | 80 | TCP | Aplicación (7) | Navegación web |
| HTTPS | 443 | TCP | Aplicación (7) | Web cifrada (TLS) |
| DNS | 53 | UDP/TCP | Aplicación (7) | Resolución de nombres |
| DHCP | 67/68 | UDP | Aplicación (7) | Configuración automática IP |
| FTP | 20/21 | TCP | Aplicación (7) | Transferencia de archivos |
| SSH | 22 | TCP | Aplicación (7) | Acceso remoto seguro |
| SMTP | 25 | TCP | Aplicación (7) | Envío de correo electrónico |

---

## Entregable 5 — TCP vs UDP

La diferencia entre TCP y UDP es que TCP establece una conexión previa mediante el **3-way handshake** (SYN → SYN-ACK → ACK) antes de transmitir datos, garantizando que todos los paquetes lleguen en orden y sin pérdida. Si un paquete se pierde, TCP lo retransmite automáticamente.

UDP en cambio no establece conexión ni confirma la entrega — simplemente envía los datos y no verifica si llegaron.

UDP conviene cuando la consulta se resuelve con un simple request/response (como DNS), o cuando la velocidad es más importante que la precisión, como en streaming en vivo, videollamadas o juegos online — donde es preferible perder un frame que pausar para retransmitir. TCP conviene cuando la integridad de los datos es crítica, como en transferencia de archivos, navegación web o correo electrónico.

---

## Entregable 6 — Puertos activos con netstat

**Comandos ejecutados:**
```powershell
Test-NetConnection google.com -Port 443
netstat -ano | findstr LISTENING
```

**Test-NetConnection:**
- RemoteAddress: 172.217.28.14
- RemotePort: 443
- InterfaceAlias: Wi-Fi
- SourceAddress: 192.168.1.9
- TcpTestSucceeded: **True** ✅

**Puertos en escucha identificados:**

| Puerto | Protocolo | PID | Servicio probable |
|--------|-----------|-----|-------------------|
| 135 | TCP | 72 | RPC — Remote Procedure Call (Windows) |
| 445 | TCP | 4 | SMB — compartición de archivos Windows |
| 902/912 | TCP | 4464 | VMware — virtualización |
| 5040 | TCP | 4332 | Windows Search / CDPSvc |
| 5357 | TCP | 4 | WSD — descubrimiento de dispositivos en red |
| 139 | TCP | 4 | NetBIOS — red local Windows |

**Análisis:**

Se ejecutó Test-NetConnection confirmando conectividad TCP exitosa al puerto HTTPS (443) desde la IP local 192.168.1.9 vía Wi-Fi. Con netstat se identificaron los puertos activos del sistema: 135 (RPC), 445 (SMB), 902/912 (VMware), 139 (NetBIOS) y 5357 (descubrimiento de dispositivos). Todos operan sobre TCP en estado LISTENING, esperando conexiones entrantes.

---

## Conclusiones

- Se verificó conectividad completa hacia Internet con 0% de pérdida de paquetes
- Se identificó la ruta de red pasando por infraestructura de Antel con salida por Argentina
- Se comprendió el funcionamiento de DNS consultando registros A y MX
- Se diferenciaron los protocolos TCP y UDP en base a sus características de confiabilidad y velocidad
- Se identificaron los servicios activos en el sistema local mediante netstat