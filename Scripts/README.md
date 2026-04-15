# 📂 Scripts

Esta carpeta contiene las plantillas **Jinja2** (`.j2`) que Ansible transforma en scripts de shell y despliega automáticamente en cada máquina. Son el corazón operativo del sistema de backups.

---

## 📁 Contenido

```
scripts/
├── backup.sh.j2           → Script de backup diario con BorgBackup
└── sync_to_samba.sh.j2    → Script de sincronización al servidor Samba
```

---

## 📄 backup.sh.j2

**Desplegado en:** todos los clientes Borg (`borg-client` y `apache-server`)
**Ruta final en la máquina:** `/usr/local/bin/borg-backup.sh`
**Ejecutado por:** cron a las **02:00** diariamente
**Log:** `/var/log/borg-backup.log`

### ¿Qué hace?

1. **Exporta las variables de entorno** necesarias para Borg:
   - `BORG_PASSPHRASE` — passphrase de descifrado del repositorio
   - `BORG_RSH` — comando SSH con la clave privada `id_borg`

2. **Crea un nuevo snapshot** con `borg create`:
   - Compresión **LZ4** (rápida y eficiente)
   - Nombre del snapshot: `backup-YYYY-MM-DDTHH:MM:SS`
   - Muestra estadísticas al finalizar (`--stats`)

3. **Aplica la política de retención** con `borg prune`:

| Tipo | Backups que conserva |
|---|---|
| Diarios | 7 (última semana) |
| Semanales | 4 (último mes) |
| Mensuales | 6 (últimos 6 meses) |

4. Sale con **código de error** si `borg create` falla, para que el cron lo registre.

### Rutas que respalda

- `borg-client`: `/home`, `/etc`, `/var/log`
- `apache-server`: `/var/www/html`

---

## 📄 sync_to_samba.sh.j2

**Desplegado en:** `borg-server`
**Ruta final en la máquina:** `/usr/local/bin/borg-sync-samba.sh`
**Ejecutado por:** cron a las **03:00** diariamente (una hora después de los backups)
**Log:** `/var/log/borg-sync-samba.log`

### ¿Qué hace?

1. **Verifica que el share Samba está montado** en `/mnt/samba_backup`:
   - Si no está montado, intenta montarlo
   - Si no puede montarlo, sale con error

2. **Sincroniza incrementalmente** con `rsync`:
   - Solo transfiere los ficheros que han cambiado
   - El flag `--delete` elimina del destino lo que ya no existe en origen
   - Origen: `/home/borg/backups/`
   - Destino: `/mnt/samba_backup/` (share Samba)

3. **Registra el resultado** con timestamps de inicio y fin.

---

## 🔧 ¿Qué es un archivo `.j2`?

Las plantillas Jinja2 son ficheros de texto con **variables** entre dobles llaves:

```bash
export BORG_PASSPHRASE="{{ borg_passphrase }}"
```

Cuando Ansible ejecuta la tarea `template`, sustituye esas variables por los valores reales definidos en `group_vars/all.yml` y genera el script `.sh` definitivo en la máquina destino. **Nunca debes editar los `.sh` directamente** — edita la plantilla `.j2` y relanza Ansible.

---

## 🕐 Cronograma completo de ejecución

```
02:00 ──► borg-backup.sh en borg-client      → Backup de /home, /etc, /var/log
02:00 ──► borg-backup.sh en apache-server    → Backup de /var/www/html
          │
          │  (espera 1 hora para que terminen los backups)
          │
03:00 ──► borg-sync-samba.sh en borg-server  → rsync de todos los repos a Samba
```

---

## 🔍 Comprobar los logs

```bash
# Ver el log del último backup (en borg-client o apache-server)
tail -f /var/log/borg-backup.log

# Ver el log de la sincronización (en borg-server)
tail -f /var/log/borg-sync-samba.log

# Listar todos los snapshots de un repositorio
BORG_PASSPHRASE="tu_passphrase" \
borg list ssh://borg@192.168.1.46/home/borg/backups/borg-client

# Restaurar un snapshot concreto
BORG_PASSPHRASE="tu_passphrase" \
borg extract ssh://borg@192.168.1.46/home/borg/backups/borg-client::backup-2025-01-15T02:00:00
```
