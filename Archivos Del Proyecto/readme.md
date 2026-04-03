# 🗄️ BorgBackup automatizado con Vagrant + Ansible

Infraestructura como código para desplegar y configurar automáticamente un sistema de copias de seguridad cifradas usando **BorgBackup**, con dos máquinas virtuales aprovisionadas mediante **Vagrant** y configuradas mediante **Ansible**.

---

## 📐 Arquitectura

```
┌─────────────────────┐        SSH / Borg         ┌─────────────────────┐
│    borg-client      │ ──────────────────────►   │    borg-server      │
│  192.168.1.214      │                           │  192.168.1.213      │
│  Ubuntu 22.04       │                           │  Ubuntu 22.04       │
│  RAM: 2 GB          │                           │  RAM: 2 GB          │
│  Disco: 50 GB       │                           │  Disco extra: 250 GB│
└─────────────────────┘                           └─────────────────────┘
```

El cliente lanza las copias de seguridad hacia el servidor a través de SSH, usando un repositorio Borg cifrado con `repokey`.

---

## 📁 Estructura del proyecto

```
.
├── Vagrantfile                          # Define y lanza las dos VMs
├── site.yml                             # Playbook principal de Ansible
├── inventory/
│   └── hosts.ini                        # Inventario con IPs y claves SSH
├── group_vars/
│   └── all.yml                          # Variables globales compartidas
└── roles/
    ├── borg_server/
    │   └── tasks/main.yml               # Configura el servidor (usuario, repo, SSH)
    └── borg_client/
        ├── tasks/main.yml               # Configura el cliente y el script de backup
        └── templates/backup.sh.j2       # Script Bash de backup con prune automático
```

---

## 🛠️ Tecnologías utilizadas

| Herramienta | Versión / Notas |
|---|---|
| **Vagrant** | Aprovisionamiento de VMs con VirtualBox |
| **Ubuntu 22.04 LTS** | Sistema operativo base (`ubuntu/jammy64`) |
| **Ansible** | Configuración automatizada (modo `ansible_local`) |
| **BorgBackup** | Copias de seguridad deduplicadas y cifradas |
| **SSH** | Transporte seguro con restricción de comandos vía `authorized_keys` |

---

## ⚙️ Qué hace cada componente

**`Vagrantfile`** — Levanta dos VMs en red pública (bridge), asigna IPs estáticas, discos adicionales y lanza el playbook de Ansible automáticamente desde el cliente.

**`site.yml`** — Playbook de Ansible que aplica los roles `borg_server` y `borg_client` a sus respectivos grupos de hosts.

**`hosts.ini`** — Inventario que define las conexiones SSH a cada máquina, con `StrictHostKeyChecking` deshabilitado para entornos de laboratorio.

**`all.yml`** — Variables compartidas: ruta del repositorio, usuario Borg, passphrase de cifrado e IP del servidor.

**`roles/borg_server/tasks/main.yml`** — Instala BorgBackup, crea el usuario `borg`, prepara el directorio del repositorio y su estructura `.ssh`.

**`roles/borg_client/tasks/main.yml`** — Instala BorgBackup, genera un par de claves SSH dedicadas, añade la clave pública al servidor con restricción de comando `borg serve`, agrega el servidor a `known_hosts`, inicializa el repositorio cifrado y despliega el script de backup.

**`backup.sh.j2`** — Plantilla Jinja2 que genera un script Bash para crear snapshots diarios (`/home`, `/etc`, `/var/log`) y aplicar retención de 7 días con `borg prune`.

---

> **Requisitos previos:** VirtualBox y Vagrant instalados en el host. Compatible con Windows gracias al uso de `ansible_local`.

---

## 🔒 Seguridad

- El repositorio Borg usa cifrado `repokey` con passphrase.
- El acceso SSH del cliente al servidor está restringido mediante `command=` en `authorized_keys`, impidiendo cualquier acción que no sea `borg serve`.
- Las claves SSH de Vagrant se gestionan con permisos `600` y propietario `vagrant`.

> ⚠️ En un entorno de producción, se recomienda almacenar la passphrase en un gestor de secretos (Ansible Vault, HashiCorp Vault, etc.) en lugar de en texto plano.
