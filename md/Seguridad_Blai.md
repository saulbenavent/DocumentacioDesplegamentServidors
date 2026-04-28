# 🔐 Seguridad del Sistema

> **Responsable:** Blai  
> **Rol:** Administrador de Seguridad  
> **Usuario del sistema:** `admin-sec`  
> **Proyecto:** SRE-Docs

---

## 📋 Índice

1. [Introducción](#introducción)
2. [Usuario admin-sec](#usuario-admin-sec)
3. [Tarea 1 — Cambio de puerto SSH](#tarea-1--cambio-de-puerto-ssh)
4. [Tarea 2 — Deshabilitar acceso root](#tarea-2--deshabilitar-acceso-root)
5. [Tarea 3 — Firewall con UFW](#tarea-3--firewall-con-ufw)
6. [Tarea 4 — Fail2Ban](#tarea-4--fail2ban)
7. [Resumen de tareas](#resumen-de-tareas)

---

## Introducción

Mi parte del proyecto consiste en asegurar el sistema frente a accesos no autorizados y ataques externos. Para ello he aplicado varias medidas de **hardening** (endurecimiento) sobre el servidor:

- Modificar la configuración de SSH para que sea más difícil de atacar
- Impedir que el usuario root pueda conectarse directamente
- Configurar un firewall que filtre el tráfico no deseado
- Instalar Fail2Ban para bloquear automáticamente IPs que intenten hacer fuerza bruta

Todo esto está documentado en este archivo y forma parte de la sección de seguridad del proyecto SRE-Docs.

---

## Usuario admin-sec

Para gestionar la seguridad del sistema se ha creado un usuario dedicado llamado `admin-sec`. Este usuario tiene permisos de `sudo` únicamente para las tareas de seguridad, siguiendo el principio de **mínimo privilegio**.

### Creación del usuario

```bash
# Crear el usuario
sudo adduser admin-sec

# Añadirlo al grupo sudo
sudo usermod -aG sudo admin-sec

# Verificar que se ha creado correctamente
id admin-sec
```

### Permisos del usuario

| Acción | Permiso |
|--------|---------|
| Editar `/etc/ssh/sshd_config` | ✅ Sí |
| Gestionar reglas UFW | ✅ Sí |
| Configurar Fail2Ban | ✅ Sí |
| Login directo como root | ❌ No |
| Acceder a carpetas NAS | ❌ No |
| Modificar el servidor web | ❌ No |

> El usuario `admin-sec` solo puede actuar sobre su área de responsabilidad. El resto de servidores están fuera de su alcance.

---

## Tarea 1 — Cambio de puerto SSH

### ¿Qué es SSH?

SSH (Secure Shell) es el protocolo que permite conectarse de forma remota a un servidor mediante la terminal. Por defecto usa el **puerto 22**, que es el primero que escanean los bots automáticos en internet.

### ¿Qué he hecho?

He cambiado el puerto SSH del 22 al **2222** para reducir el ruido de ataques automatizados. Esto no elimina el riesgo, pero filtra una gran cantidad de intentos de intrusión que se dirigen al puerto estándar.

### Pasos realizados

**1. Abrir el fichero de configuración de SSH:**

```bash
sudo nano /etc/ssh/sshd_config
```

**2. Buscar la línea del puerto y cambiarla:**

```bash
# Antes (línea comentada por defecto):
#Port 22

# Después (activada con el nuevo puerto):
Port 2222
```

**3. Guardar y reiniciar el servicio SSH:**

```bash
sudo systemctl restart ssh
```

**4. Verificar que el servicio escucha en el nuevo puerto:**

```bash
sudo ss -tlnp | grep 2222
```

**5. Actualizar el firewall para permitir el nuevo puerto (ver Tarea 3):**

```bash
sudo ufw allow 2222
```

> ⚠️ **Importante:** Antes de cerrar la sesión actual, hay que abrir otra sesión con el nuevo puerto para asegurarse de que funciona correctamente y no quedarse sin acceso.

---

## Tarea 2 — Deshabilitar acceso root

### ¿Por qué deshabilitar root?

El usuario `root` tiene acceso total al sistema. Si un atacante consigue acceder como root puede hacer cualquier cosa: borrar archivos, instalar malware, modificar configuraciones críticas, etc. Por eso es una buena práctica impedir que root pueda conectarse directamente por SSH.

Al deshabilitarlo, forzamos a que cualquier persona que quiera administrar el sistema tenga que:
1. Conectarse con un usuario normal (`admin-sec`)
2. Usar `sudo` para ejecutar comandos privilegiados

Esto añade una capa extra de seguridad y deja trazabilidad de quién hizo qué.

### Pasos realizados

**1. Abrir el fichero de configuración de SSH:**

```bash
sudo nano /etc/ssh/sshd_config
```

**2. Buscar la línea `PermitRootLogin` y deshabilitarla:**

```bash
# Antes:
#PermitRootLogin yes

# Después:
PermitRootLogin no
```

**3. Reiniciar el servicio SSH:**

```bash
sudo systemctl restart ssh
```

**4. Comprobar que root no puede conectarse:**

```bash
# Intentar conectar como root debe dar error:
ssh root@192.168.1.13 -p 2222
# Permission denied, please try again.
```

---

## Tarea 3 — Firewall con UFW

### ¿Qué es UFW?

UFW (Uncomplicated Firewall) es una herramienta para gestionar el firewall de Linux de forma sencilla. Permite definir qué puertos y conexiones están permitidos y cuáles se bloquean.

### ¿Qué he configurado?

He aplicado una política de **denegar todo por defecto** y solo he permitido explícitamente los puertos necesarios para el funcionamiento del servidor.

### Reglas configuradas

| Puerto / Servicio | Acción | Descripción |
|-------------------|--------|-------------|
| 2222 (SSH) | ✅ ALLOW | Acceso remoto al servidor |
| 80 (HTTP) | ✅ ALLOW | Tráfico web del servidor Apache |
| 443 (HTTPS) | ✅ ALLOW | Tráfico web cifrado (TLS) |
| Todo lo demás | ❌ DENY | Bloqueado por defecto |

### Pasos realizados

**1. Instalar UFW:**

```bash
sudo apt install ufw -y
```

**2. Establecer política por defecto (denegar todo):**

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

**3. Permitir los puertos necesarios:**

```bash
# Permitir SSH en el nuevo puerto
sudo ufw allow 2222

# Permitir tráfico web
sudo ufw allow 80
sudo ufw allow 443
```

**4. Activar el firewall:**

```bash
sudo ufw enable
```

**5. Verificar el estado y las reglas activas:**

```bash
sudo ufw status verbose
```

**Salida esperada:**

```
Status: active

To                         Action      From
--                         ------      ----
2222                       ALLOW IN    Anywhere
80                         ALLOW IN    Anywhere
443                        ALLOW IN    Anywhere
```

---

## Tarea 4 — Fail2Ban

### ¿Qué es Fail2Ban?

Fail2Ban es un servicio que **monitoriza los logs del sistema** en tiempo real. Cuando detecta que una misma IP ha fallado demasiadas veces al intentar autenticarse, la banea automáticamente usando el firewall, impidiéndole seguir intentándolo.

Es especialmente útil contra ataques de **fuerza bruta**, donde un atacante prueba miles de contraseñas automáticamente.

### ¿Cómo funciona?

```
IP externa intenta conectar por SSH
        │
        ▼
¿Ha fallado más de 5 veces en 10 minutos?
        │
   Sí ──┤── Fail2Ban banea la IP con UFW
        │
   No ──┘── Se permite continuar intentando
```

### Parámetros configurados

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| `maxretry` | 5 | Intentos fallidos antes del baneo |
| `findtime` | 600 | Ventana de tiempo en segundos (10 min) |
| `bantime` | 600 | Duración del baneo en segundos (10 min) |
| Servicio protegido | `sshd` | Monitoriza los intentos SSH |

### Pasos realizados

**1. Instalar Fail2Ban:**

```bash
sudo apt install fail2ban -y
```

**2. Copiar el fichero de configuración para no modificar el original:**

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

**3. Editar los parámetros:**

```bash
sudo nano /etc/fail2ban/jail.local
```

Dentro del fichero, en la sección `[DEFAULT]`:

```ini
bantime  = 600
findtime = 600
maxretry = 5
```

Y en la sección `[sshd]` (activar el jail de SSH):

```ini
[sshd]
enabled = true
port    = 2222
```

> Es importante indicar el puerto 2222 porque hemos cambiado el puerto por defecto.

**4. Activar y arrancar el servicio:**

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

**5. Comprobar que está funcionando:**

```bash
sudo systemctl status fail2ban
```

**6. Ver las IPs baneadas en tiempo real:**

```bash
sudo fail2ban-client status sshd
```

**Salida esperada:**

```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     3
|  `- File list:        /var/log/auth.log
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   192.168.1.99
```

---

## Resumen de tareas

A continuación se muestra un resumen de todas las tareas de seguridad realizadas y su estado:

| # | Tarea | Comando clave | Estado |
|---|-------|---------------|--------|
| 1 | Cambiar puerto SSH 22 → 2222 | `Port 2222` en `sshd_config` | ✅ Completado |
| 2 | Deshabilitar login root | `PermitRootLogin no` en `sshd_config` | ✅ Completado |
| 3 | Configurar firewall UFW | `ufw allow 2222`, `ufw enable` | ✅ Completado |
| 4 | Instalar y configurar Fail2Ban | `apt install fail2ban` + `jail.local` | ✅ Completado |
| 5 | Crear usuario `admin-sec` | `adduser admin-sec` | ✅ Completado |

---

## Referencias

- Documentación oficial de UFW: https://help.ubuntu.com/community/UFW
- Documentación oficial de Fail2Ban: https://www.fail2ban.org/wiki/index.php/Main_Page
- Manual de OpenSSH: https://www.openssh.com/manual.html
- Guía de hardening en Ubuntu: https://ubuntu.com/security/certifications/docs/usg/cis
