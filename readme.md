# 🛡️ TFG — Infraestructura de Backups Automatizada con BorgBackup

> Trabajo de Fin de Grado — Sistema de copias de seguridad cifradas, automatizadas y redundantes mediante **Vagrant**, **Ansible** y **BorgBackup**.

---

## 📋 Descripción

Este proyecto despliega de forma completamente automatizada una infraestructura de backups que incluye:

- **Cifrado** de los repositorios con BorgBackup (repokey)
- **Compresión** LZ4 para minimizar el espacio usado
- **Redundancia** mediante sincronización a un servidor Samba
- **Automatización** total con Ansible y cronjobs
- **Política de retención** configurable (diaria, semanal, mensual)

---

## 🖥️ Arquitectura

```
┌─────────────────┐        SSH + Borg        ┌──────────────────────┐
│  borg-client    │ ──────────────────────►  │   borg-server        │
│  192.168.1.47   │                           │   192.168.1.46       │
└─────────────────┘                           └──────────┬───────────┘
                                                         │ rsync 03:00
┌─────────────────┐        SSH + Borg                    ▼
│  apache-server  │ ──────────────────────►  ┌──────────────────────┐
│  192.168.1.42   │                           │   samba-server       │
└─────────────────┘                           │   192.168.1.44       │
                                              └──────────────────────┘
```

| Máquina | IP | Rol |
|---|---|---|
| borg-server | 192.168.1.46 | Servidor central de backups + cliente Samba |
| borg-client | 192.168.1.47 | Cliente Borg (VM Vagrant) |
| apache-server | 192.168.1.42 | Servidor web Apache + cliente Borg |
| samba-server | 192.168.1.44 | Almacenamiento redundante Samba |

---

## 📁 Estructura del repositorio

```
📦 TFG-Backups/
├── 📂 Vagrant/          → Definición y aprovisionamiento de las VMs
├── 📂 ansible/          → Roles, playbooks e inventario de Ansible
└── 📂 scripts/          → Scripts de backup y sincronización (.j2)
```

Consulta el README de cada carpeta para más detalles.

---

## ⚙️ Requisitos previos

- [VirtualBox](https://www.virtualbox.org/) instalado
- [Vagrant](https://www.vagrantup.com/) instalado
- Máquinas `apache-server` y `samba-server` ya creadas y accesibles por SSH
- Claves SSH de apache y samba en `Vagrant/ssh_keys/`

---

## 🚀 Despliegue rápido

```bash
# 1. Clona el repositorio
git clone https://github.com/tu-usuario/TFG-Backups.git
cd TFG-Backups

# 2. Coloca las claves SSH en la carpeta correcta
cp apache-server.key Vagrant/ssh_keys/
cp samba-server.key  Vagrant/ssh_keys/

# 3. Edita las contraseñas en ansible/group_vars/all.yml
#    Cambia: borg_passphrase y samba_password

# 4. Levanta la infraestructura
cd Vagrant
vagrant up
```

Vagrant levantará ambas VMs y lanzará Ansible automáticamente desde `borg-client`.

---

## ⏰ Programación de tareas

| Hora | Tarea | Ejecutado por |
|---|---|---|
| 02:00 | Backup de borg-client | cron en borg-client |
| 02:00 | Backup de apache-server | cron en apache-server |
| 03:00 | Sincronización a Samba | cron en borg-server |

---

## 🔒 Seguridad

- Las claves SSH de Borg usan restricciones `command=` en `authorized_keys` — solo permiten ejecutar `borg serve`, nada más.
- El repositorio Borg está cifrado con `--encryption=repokey`.
- Las credenciales Samba se guardan en `/etc/samba-borg.creds` con permisos `0600`.
- Los logs se guardan en `/var/log/borg-backup.log` y `/var/log/borg-sync-samba.log`.

---

## 📄 Licencia

Proyecto académico — TFG. Todos los derechos reservados.
