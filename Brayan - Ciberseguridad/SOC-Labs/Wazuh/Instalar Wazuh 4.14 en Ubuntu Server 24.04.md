---
tags:
  - wazuh
  - siem
  - soc
  - ubuntu
  - virtualbox
  - laboratorio
herramienta: Wazuh 4.14
sistema_operativo: Ubuntu Server 24.04.5 LTS
plataforma_virtualizacion: VirtualBox
fecha_laboratorio: 2026-11-03
estado: Completado ✅
---

# Instalar Wazuh 4.14 en Ubuntu Server 24.04

Laboratorio de despliegue de **Wazuh** (SIEM/XDR open source: indexer + manager + dashboard, instalación "todo en uno") sobre una VM de Ubuntu Server 24.04 en VirtualBox.

## 📋 Resumen

| Campo | Detalle |
|---|---|
| Herramienta | Wazuh 4.14.9 |
| SO de la VM | Ubuntu Server 24.04.5 LTS |
| Hipervisor | Oracle VirtualBox |
| RAM asignada | 8192 MB |
| CPUs asignadas | 4 |
| Disco | ~104.5 GB (VDI, LVM) |
| Red | Adaptador puente |
| IP del servidor | `192.168.1.10` /24 (DHCP) — en el despliegue final: `192.168.100.87` |
| Usuario del sistema | `sistemas` |
| Contraseña | `Sistemas123` |
| Acceso al dashboard | `https://<IP-DEL-SERVIDOR>` — Usuario `admin` |

---

## 🖥️ 1. Creación de la máquina virtual (VirtualBox)

- **Nombre de la VM:** `Wazuh`
- **Imagen ISO:** `ubuntu-24.04.5-live-server-amd64.iso`
- **Sistema:** Linux → Ubuntu (64-bit)

![Nombre de VM e ISO](attachments/01-vm-nombre-iso.png)

**Hardware asignado:**
- RAM: **8192 MB**
- CPUs: **4**

![RAM y CPU](attachments/02-vm-ram-cpu.png)

**Disco virtual:** ~104.55 GB, formato VDI.

![Configuración de disco](attachments/03-vm-disco.png)

**Red:** se cambió el adaptador a **"Adaptador puente"**, para que la VM obtenga IP directamente de la red física (necesario para acceder al dashboard desde el navegador del host).

![Red en modo puente](attachments/04-red-adaptador-puente.png)

---

## 💿 2. Instalación de Ubuntu Server 24.04

**Configuración de red** durante el instalador — IP asignada por DHCP: `192.168.1.10/24`.

![Configuración de red del instalador](attachments/05-netplan-ip-servidor.png)

**Particionado de disco** — configuración guiada con **LVM**:

![Storage guiado con LVM](attachments/06-storage-guiado-lvm.png)

Resumen de particiones generadas (`/` sobre LVM con 51.26 GB, `/boot` con 2 GB):

![Resumen de particiones](attachments/07-storage-resumen-particiones.png)

**Perfil de usuario:**
- Nombre del servidor: `wazuh`
- Usuario: `sistemas`
- Contraseña: `Sistemas123`

![Configuración de usuario](attachments/08-profile-config-usuario.png)

**SSH:** se habilitó la instalación del servidor OpenSSH con autenticación por contraseña, para poder administrar la VM remotamente desde el inicio.

![Configuración SSH](attachments/09-ssh-config.png)

Snaps destacados: no se seleccionó ninguno (no son necesarios para este laboratorio).

![Snaps disponibles](attachments/10-featured-snaps.png)

Instalación finalizada:

![Instalación completa](attachments/11-instalacion-completa.png)

---

## 🔌 3. Conexión remota (PuTTY)

Se conectó por SSH a `192.168.1.10` desde PuTTY, aceptando la huella del host (primera conexión):

![Alerta de seguridad PuTTY](attachments/12-putty-security-alert.png)

Login exitoso como `sistemas`, confirmando versión del sistema (Ubuntu 24.04.5 LTS, kernel 6.8.0-147-generic):

![Login SSH exitoso](attachments/13-login-ssh-ok.png)

> **Nota:** desde ahí en adelante, el documento original indica que se usó **Putty** por comodidad para pegar comandos largos (en vez de la consola de VirtualBox).

---

## ⚙️ 4. Comandos usados

Ver la IP del servidor:
```bash
ip a
```

Comprobar salida a internet:
```bash
ping www.google.com
```

Cambiar a usuario root *(opcional, usado en el laboratorio)*:
```bash
sudo su
```

![Ping y apt update colgado](attachments/14-ping-apt-update-lento.png)

Actualizar el sistema:
```bash
sudo apt update
sudo apt upgrade -y
```

![apt upgrade sin cambios pendientes](attachments/15-apt-upgrade.png)

> **Incidencia encontrada:** el mirror colombiano (`co.archive.ubuntu.com`) no respondía (timeout de conexión al puerto 80), lo que dejaba `apt update` colgado o con errores de conexión.

**Solución — cambiar el mirror al repositorio global de Ubuntu:**
```bash
sudo sed -i 's/co.archive.ubuntu.com/archive.ubuntu.com/g' /etc/apt/sources.list.d/ubuntu.sources
sudo apt update
```

![apt update tras cambiar el mirror](attachments/16-apt-update-cambio-mirror.png)

> **Lección:** cuando `apt update`/`upgrade` se cuelga o da timeout contra un mirror regional, reemplazarlo por el mirror global (`archive.ubuntu.com`) resuelve el problema sin tener que tocar nada más de la configuración de red.

---

## 🚀 5. Instalación de Wazuh 4.14 (todo en uno)

Instalación del indexer + manager + dashboard con el script oficial:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

El instalador reporta la versión (**Wazuh 4.14.9**), genera certificados, instala y arranca el indexer, y luego el manager:

![Log de instalación de Wazuh](attachments/17-wazuh-install-script-log.png)

---

## 🌐 6. Acceso al dashboard

Entrar al dashboard desde el navegador:
```
https://IP-DE-TU-SERVIDOR
```
- Usuario: `admin`
- Contraseña: la que entrega el instalador al finalizar (**guardarla de inmediato**, solo se muestra una vez).

El navegador advierte que el certificado es autofirmado — es normal en una instalación local: clic en **Avanzado → Continuar**.

![Certificado autofirmado](attachments/18-certificado-autofirmado.png)

Carga del dashboard:

![Wazuh cargando](attachments/19-wazuh-cargando.png)

Dashboard funcionando — vista general sin agentes registrados todavía:

![Dashboard Wazuh - Overview](attachments/20-wazuh-dashboard-overview.png)

**Credenciales registradas en este despliegue:**
```
Usuario: admin
Password: n61lYr3E+iZCBKmXakemtP6u+czZuSG7
```

---

## 🛠️ Comandos útiles (post-instalación)

Si se olvida la contraseña del dashboard — está dentro del archivo que genera el instalador:
```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

Desactivar el repositorio de Wazuh para evitar actualizaciones accidentales:
```bash
sudo sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/wazuh.list
sudo apt update
```

Revisar si el sistema necesita reiniciarse:
```bash
ls /var/run/reboot-required
```

Ver el estado de los servicios de Wazuh:
```bash
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard
```

---

## 🧠 Lecciones aprendidas

- La instalación "todo en uno" (`-a`) de Wazuh levanta indexer + manager + dashboard en un solo comando, ideal para laboratorio o pruebas pequeñas.
- Un mirror regional de `apt` caído no significa que el sistema no tenga internet — revisar con `ping` primero y, si el mirror falla, cambiar a `archive.ubuntu.com`.
- La contraseña del dashboard solo se muestra una vez al final de la instalación; si se pierde, sigue estando disponible en `wazuh-install-files.tar`.
- El certificado HTTPS autofirmado es esperado en un despliegue on-premise sin CA propia — no indica una falla de la instalación.

## 📚 Referencias

- Documentación oficial: `https://documentation.wazuh.com/`
- Script de instalación "todo en uno": `https://packages.wazuh.com/4.14/wazuh-install.sh`
