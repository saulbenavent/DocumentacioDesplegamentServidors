# Servidor NAS (Jacob)

## Información general

| Campo | Valor |
|---|---|
| Sistema operativo | Ubuntu Server 22.04 LTS |
| IP fija | 192.168.1.50 |
| Servicio principal | Samba (SMB/CIFS) |
| Carpeta compartida | `/srv/nas` |
| Nombre del recurso | `nas` |
| Usuario de acceso | `nasuser` |
| Grupo de acceso | `nasusers` |

---

## 1. Instalación del sistema operativo

Se ha utilizado **Ubuntu Server 22.04 LTS** como sistema operativo para el servidor NAS por su estabilidad, soporte a largo plazo (hasta 2027) y compatibilidad con Samba.

La máquina virtual se configuró con los siguientes parámetros en VirtualBox:

- RAM: 2048 MB
- Disco: 20 GB (VDI, asignación dinámica)
- Red: Adaptador en red interna
- Durante la instalación se activó el servidor OpenSSH

> **Captura requerida:** Pantalla final de instalación de Ubuntu Server mostrando "Installation complete".

---

## 2. Configuración de red (IP fija)

Se asignó una IP estática al servidor editando el archivo de configuración de Netplan:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Contenido del archivo:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1.50/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [192.168.1.10]
```

Aplicar la configuración:

```bash
sudo netplan apply
```

Verificar la IP asignada:

```bash
ip a
```

> **Captura requerida:** Salida del comando `ip a` mostrando la IP `192.168.1.50` en la interfaz `enp0s3`.

---

## 3. Instalación de Samba

Actualización del sistema e instalación del servicio Samba:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install samba samba-common-bin -y
```

Verificar que el servicio está activo:

```bash
sudo systemctl status smbd
```

Habilitar Samba en el arranque del sistema:

```bash
sudo systemctl enable smbd nmbd
```

Abrir los puertos necesarios en el firewall:

```bash
sudo ufw allow samba
```

> **Captura requerida:** Salida de `sudo systemctl status smbd` mostrando el estado `active (running)`.

---

## 4. Creación de la carpeta compartida

Se creó la carpeta que actuará como almacenamiento compartido del NAS:

```bash
sudo mkdir -p /srv/nas
```

Se creó un grupo y un usuario dedicados al acceso NAS:

```bash
sudo groupadd nasusers
sudo useradd -m nasuser -s /bin/bash
sudo usermod -aG nasusers nasuser
```

Se asignaron los permisos correctos a la carpeta:

```bash
sudo chown root:nasusers /srv/nas
sudo chmod 2770 /srv/nas
```

> El bit `setgid (2)` garantiza que los archivos creados dentro hereden el grupo `nasusers`.

Se creó la contraseña Samba para el usuario:

```bash
sudo smbpasswd -a nasuser
```

> **Captura requerida:** Salida de `ls -la /srv/` mostrando los permisos `drwxrws---` y el grupo `nasusers` en la carpeta `nas`.

---

## 5. Configuración de Samba (`smb.conf`)

Se realizó una copia de seguridad del archivo de configuración original:

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

Se editó el archivo de configuración:

```bash
sudo nano /etc/samba/smb.conf
```

Se añadió el siguiente bloque al final del archivo:

```ini
[nas]
   comment = Carpeta compartida NAS
   path = /srv/nas
   valid users = @nasusers
   read only = no
   browseable = yes
   create mask = 0660
   directory mask = 0770
   force group = nasusers
```

Se verificó la configuración con la herramienta `testparm`:

```bash
testparm
```

Se reinició el servicio para aplicar los cambios:

```bash
sudo systemctl restart smbd nmbd
```

> **Captura requerida:** Salida de `testparm` sin errores, mostrando el recurso `[nas]` en la lista de shares.

---

## 6. Prueba de acceso desde otro equipo

### Desde Linux

Listar los recursos compartidos del servidor:

```bash
smbclient -L //192.168.1.50 -U nasuser
```

Conectarse a la carpeta compartida:

```bash
smbclient //192.168.1.50/nas -U nasuser
```

Montar la unidad de forma permanente:

```bash
sudo mount -t cifs //192.168.1.50/nas /mnt/nas -o user=nasuser
```

### Desde Windows

Acceder desde el Explorador de archivos escribiendo en la barra de direcciones:

```
\\192.168.1.50\nas
```

> **Captura requerida:** Acceso exitoso al recurso compartido `nas` desde otro equipo de la red (puede ser desde la máquina del servidor web o desde Windows), mostrando la carpeta accesible y que es posible crear/leer archivos.

---

## 7. Resumen de comandos utilizados

| Comando | Descripción |
|---|---|
| `sudo apt install samba samba-common-bin -y` | Instala Samba |
| `sudo mkdir -p /srv/nas` | Crea la carpeta compartida |
| `sudo groupadd nasusers` | Crea el grupo de acceso |
| `sudo useradd -m nasuser` | Crea el usuario NAS |
| `sudo chmod 2770 /srv/nas` | Asigna permisos con setgid |
| `sudo smbpasswd -a nasuser` | Crea contraseña Samba |
| `testparm` | Verifica la configuración |
| `sudo systemctl restart smbd nmbd` | Reinicia Samba |
| `smbclient //192.168.1.50/nas -U nasuser` | Prueba el acceso desde Linux |

---

## 8. Capturas necesarias (resumen)

- [ ] Instalación de Ubuntu Server completada
- [ ] Salida de `ip a` con IP estática `192.168.1.50`
- [ ] `systemctl status smbd` en estado `active (running)`
- [ ] `ls -la /srv/` mostrando permisos de la carpeta `nas`
- [ ] Salida de `testparm` sin errores
- [ ] Acceso exitoso desde otro equipo de la red