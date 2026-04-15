# 📂 Ansible

Esta carpeta contiene todos los roles, playbooks, inventario y variables que **Ansible** usa para configurar automáticamente toda la infraestructura de backups.

---

## 📁 Contenido

```
ansible/
├── site.yml                          → Playbook principal (punto de entrada)
├── inventory/
│   └── hosts.ini                     → Inventario de máquinas
├── group_vars/
│   └── all.yml                       → Variables globales del proyecto
├── host_vars/
│   └── apache-server.yml             → Variables específicas del Apache
└── roles/
    ├── borg_server/
    │   └── tasks/
    │       └── main.yml              → Configura el servidor Borg
    ├── borg_client/
    │   └── tasks/
    │       └── main.yml              → Configura los clientes Borg
    ├── apache_server/
    │   └── tasks/
    │       └── main.yml              → Instala y configura Apache
    └── samba_server/
        ├── tasks/
        │   └── main.yml              → Instala y configura Samba
        └── handlers/
            └── main.yml              → Handler: reinicia smbd
```

---

## 🔄 Orden de ejecución (`site.yml`)

Ansible ejecuta los roles en este orden concreto para respetar las dependencias:

```
1. samba_server   → Debe estar listo antes de que borg-server intente montarlo
2. borg_server    → Monta el share Samba y queda listo para recibir clientes
3. borg_client    → Se conecta al borg-server y registra sus claves
4. apache_server  → Instala Apache y lo registra también como cliente Borg
   + borg_client
```

---

## 📋 Roles

### 🟦 borg_server
Configura la máquina que centraliza todos los backups:
- Instala `borgbackup` y `cifs-utils`
- Crea el usuario `borg` con su directorio de repositorios
- Prepara el `authorized_keys` vacío para los clientes
- Monta el share Samba en `/mnt/samba_backup` de forma persistente (fstab)
- Guarda las credenciales Samba en `/etc/samba-borg.creds` (permisos 0600)
- Despliega el script de sincronización y programa el cron a las 03:00

### 🟦 borg_client
Configura cualquier máquina que deba hacer backup:
- Instala `borgbackup`
- Genera un par de claves SSH dedicadas (`id_borg` / `id_borg.pub`)
- Añade la clave pública al `authorized_keys` del servidor **con restricciones** (`command=borg serve`)
- Crea el subdirectorio del repositorio en el servidor
- Inicializa el repositorio Borg cifrado (`--encryption=repokey`)
- Despliega el script de backup y programa el cron a las 02:00

### 🟦 apache_server
- Instala Apache2
- Despliega una página de ejemplo en `/var/www/html/index.html`
- Asegura que el servicio arranca y está habilitado

### 🟦 samba_server
- Instala Samba
- Crea el usuario del sistema y le asigna contraseña Samba
- Crea el directorio del share en `/srv/samba/borg_backups`
- Configura el bloque `[borg_backups]` en `smb.conf`
- Asegura que smbd arranca y está habilitado

---

## 🔧 Variables principales (`group_vars/all.yml`)

| Variable | Descripción | Valor por defecto |
|---|---|---|
| `borg_server_ip` | IP del servidor Borg | `192.168.1.46` |
| `borg_repo_path` | Ruta de repositorios en el servidor | `/home/borg/backups` |
| `borg_backup_user` | Usuario Borg en el servidor | `borg` |
| `borg_passphrase` | Passphrase de cifrado | ⚠️ **Cámbiala** |
| `borg_client_user` | Usuario que ejecuta los backups | `vagrant` |
| `borg_backup_paths` | Rutas a respaldar | `/home`, `/etc`, `/var/log` |
| `samba_server_ip` | IP del servidor Samba | `192.168.1.44` |
| `samba_share_name` | Nombre del share | `borg_backups` |
| `samba_user` | Usuario Samba | `borguser` |
| `samba_password` | Contraseña Samba | ⚠️ **Cámbiala** |
| `samba_mount_point` | Punto de montaje en borg-server | `/mnt/samba_backup` |

> ⚠️ **Antes de desplegar**, edita `group_vars/all.yml` y cambia `borg_passphrase` y `samba_password` por valores seguros reales.

---

## 🚀 Ejecutar Ansible manualmente

Si quieres relanzar Ansible sin recrear las VMs, conéctate al `borg-client` y ejecuta:

```bash
vagrant ssh borg-client

# Desde dentro de la VM:
cd /vagrant
ansible-playbook -i inventory/hosts.ini site.yml

# Solo un rol o host concreto:
ansible-playbook -i inventory/hosts.ini site.yml --limit apache_server
ansible-playbook -i inventory/hosts.ini site.yml --tags borg_client
```

---

## 🔒 Seguridad SSH de Borg

La clave pública de cada cliente se añade al `authorized_keys` del servidor con una restricción `command=`:

```
command="borg serve --restrict-to-path /home/borg/backups/HOSTNAME",
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty
ssh-rsa AAAA...
```

Esto significa que aunque alguien consiga la clave privada, **solo puede ejecutar `borg serve`** en el servidor — no tiene acceso a una shell normal.
