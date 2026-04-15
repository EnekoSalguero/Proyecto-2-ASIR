# 📂 Vagrant

Esta carpeta contiene todo lo necesario para levantar y aprovisionar las máquinas virtuales del proyecto mediante **Vagrant + VirtualBox**.

---

## 📁 Contenido

```
Vagrant/
├── Vagrantfile            → Define las VMs y su configuración
├── server_ip.txt          → Generado automáticamente con la IP del borg-server
├── borg-client-ssh.cfg    → Config SSH para conectarte a borg-client desde Windows
└── ssh_keys/
    ├── apache-server.key  → Clave privada SSH del servidor Apache
    └── samba-server.key   → Clave privada SSH del servidor Samba
```

> ⚠️ La carpeta `ssh_keys/` **no está versionada en Git** por seguridad. Debes colocar las claves manualmente antes de lanzar `vagrant up`.

---

## 🖥️ Máquinas definidas

### borg-server
| Parámetro | Valor |
|---|---|
| IP estática | 192.168.1.46 |
| RAM | 2048 MB |
| CPUs | 2 |
| Disco extra | 250 GB |
| Usuario | vagrant |

### borg-client
| Parámetro | Valor |
|---|---|
| IP estática | 192.168.1.47 |
| RAM | 2048 MB |
| CPUs | 2 |
| Disco extra | 50 GB |
| Usuario | vagrant |

---

## ⚙️ Qué hace el Vagrantfile al arrancar

1. Levanta `borg-server` y le asigna la IP `192.168.1.46` con netplan
2. Levanta `borg-client` y le asigna la IP `192.168.1.47` con netplan
3. En `borg-client`:
   - Copia las claves SSH de todas las VMs a `/home/vagrant/.ssh/`
   - Instala `python3-passlib` (dependencia de Ansible)
   - Añade todas las IPs al `/etc/hosts`
   - Genera el `group_vars/all.yml` y el `inventory/hosts.ini`
4. **Lanza Ansible automáticamente** con `ansible_local` contra todas las máquinas

---

## 🚀 Comandos útiles

```bash
# Levantar toda la infraestructura
vagrant up

# Levantar solo una VM
vagrant up borg-server
vagrant up borg-client

# Conectarse por SSH
vagrant ssh borg-client
vagrant ssh borg-server

# Relanzar el aprovisionamiento de Ansible
vagrant provision borg-client

# Apagar las VMs
vagrant halt

# Destruir las VMs (¡borra todo!)
vagrant destroy -f
```

---

## 🔧 Configuración del adaptador de red

El Vagrantfile usa un adaptador puente (`public_network`). Si tu adaptador de red no se llama `Realtek PCIe GbE Family Controller`, cámbialo en las líneas:

```ruby
server.vm.network "public_network", bridge: "TU ADAPTADOR AQUÍ"
client.vm.network "public_network", bridge: "TU ADAPTADOR AQUÍ"
```

Puedes ver el nombre exacto de tu adaptador ejecutando en Windows:
```powershell
Get-NetAdapter | Select-Object Name
```

---

## ⚠️ Prerequisitos

- VirtualBox instalado
- Vagrant instalado
- Las máquinas `apache-server` (192.168.1.42) y `samba-server` (192.168.1.44) deben existir y estar encendidas antes de lanzar `vagrant up`
