# Lab 01 — Protocolos Básicos de Red

## Objetivo

Comprender el funcionamiento de los protocolos básicos de la pila TCP/IP, identificar sus puertos, capas y roles dentro de la comunicación en red, y ejecutar comandos de diagnóstico que evidencien su operación en un sistema real.

---

## Protocolos estudiados

| Protocolo | Puerto(s) | Transporte | Capa OSI | Función |
|-----------|-----------|------------|----------|---------|
| HTTP | 80 | TCP | Aplicación (7) | Navegación web |
| HTTPS | 443 | TCP | Aplicación (7) | Web cifrada (TLS) |
| DNS | 53 | UDP/TCP | Aplicación (7) | Resolución de nombres |
| DHCP | 67/68 | UDP | Aplicación (7) | Configuración automática IP |
| FTP | 20/21 | TCP | Aplicación (7) | Transferencia de archivos |
| SSH | 22 | TCP | Aplicación (7) | Acceso remoto seguro |
| SMTP | 25 | TCP | Aplicación (7) | Envío de correo electrónico |
| ICMP | — | — | Red (3) | Diagnóstico y control |
| TCP | — | — | Transporte (4) | Transporte confiable |
| UDP | — | — | Transporte (4) | Transporte rápido sin estado |

---

## Comandos utilizados

### PING — diagnóstico ICMP

```powershell
# Windows / PowerShell
ping -n 5 8.8.8.8

# Linux / macOS
ping -c 5 8.8.8.8
ping -c 4 -s 1000 8.8.8.8
```

**Campos a analizar:**
- `bytes` — tamaño del paquete ICMP
- `TTL` — Time To Live, permite calcular saltos: TTL_inicial - TTL_recibido = saltos
- `tiempo/time` — latencia Round Trip Time (RTT) en milisegundos
- `perdidos/loss` — porcentaje de paquetes perdidos

---

### TRACEROUTE — ruta al destino

```powershell
# Windows
tracert 8.8.8.8
tracert google.com

# Linux / macOS
traceroute -n google.com
traceroute -m 15 8.8.8.8
```

**Campos a analizar:**
- Cada línea numerada = un router (salto) en la ruta
- `* * *` = router con ICMP bloqueado por firewall (normal en ISPs)
- El último salto = destino alcanzado

---

### DNS — resolución de nombres

```powershell
# Windows
nslookup google.com
nslookup -type=MX google.com

# PowerShell
Resolve-DnsName google.com
Resolve-DnsName google.com -Type MX
Resolve-DnsName google.com -Server 8.8.8.8

# Linux / macOS / Windows con BIND instalado
dig google.com
dig google.com MX
dig -x 8.8.8.8
dig +short google.com
dig +trace google.com
```

**Tipos de registros DNS:**
| Tipo | Descripción |
|------|-------------|
| A | IPv4 del dominio |
| AAAA | IPv6 del dominio |
| MX | Servidor de correo |
| CNAME | Alias de dominio |
| PTR | DNS inverso (IP → nombre) |

---

### Puertos y conexiones activas

```powershell
# Ver todas las conexiones activas
netstat -an

# Ver puertos en escucha con PID
netstat -ano | findstr LISTENING

# Verificar conectividad TCP a un puerto (PowerShell)
Test-NetConnection google.com -Port 443
Test-NetConnection google.com -Port 80

# Linux
ss -tlnp
nc -zv google.com 443
```

---

## Equivalencias Linux / Windows / PowerShell

| Función | Linux/macOS | Windows CMD | PowerShell |
|---------|-------------|-------------|------------|
| Ping | `ping -c 5` | `ping -n 5` | `ping -n 5` |
| Traceroute | `traceroute` | `tracert` | `tracert` |
| DNS lookup | `dig` | `nslookup` | `Resolve-DnsName` |
| Puertos activos | `ss -tlnp` | `netstat -ano` | `netstat -ano` |
| Test de puerto | `nc -zv host port` | `telnet host port` | `Test-NetConnection host -Port N` |

---

## Notas adicionales

- `dig` no viene instalado en Windows por defecto. Se instala con `winget install ISC.BIND`
- `telnet` en Windows requiere habilitarlo: `Enable-WindowsOptionalFeature -Online -FeatureName TelnetClient`
- `ss` es exclusivo de Linux, el equivalente Windows es `netstat`
- `Test-NetConnection` es más completo que `telnet` en PowerShell: muestra latencia y resultado explícito

---

## Archivos

- [`informe.md`](./informe.md) — resultados y análisis del laboratorio
- [`capturas/`](./capturas/) — screenshots de cada comando ejecutado