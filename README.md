# Proyecto-AutomatizaciónBackupInfraestructura
<div align="center">

<img src="assets/logo_backup.png" width="420">

# 💾 Sistema Automatizado de Copias de Seguridad

### Infraestructura automatizada para gestión y orquestación de backups empresariales

Infraestructura diseñada para **automatizar completamente el proceso de copias de seguridad en un entorno empresarial**, reduciendo la intervención humana, optimizando el almacenamiento mediante **deduplicación** y garantizando **alta disponibilidad de los datos**.

</div>

---

# ⚙️ Stack tecnológico del proyecto

<table>
<tr>
<td align="center" width="160">

**Virtualización**

📦  
Vagrant

</td>

<td align="center" width="160">

**Automatización**

⚙️  
Ansible

</td>

<td align="center" width="160">

**Backups**

💾  
BorgBackup

</td>

<td align="center" width="160">

**Almacenamiento distribuido**

🔗  
GlusterFS

</td>

<td align="center" width="160">

**Compartición de archivos**

📂  
Samba

</td>

<td align="center" width="160">

**Base de datos**

🗄  
MySQL

</td>

</tr>
</table>

---

# 📖 Descripción del proyecto

En cualquier empresa, la **protección de la información es crítica**. La pérdida de datos puede provocar interrupciones del servicio, pérdidas económicas o incluso problemas legales.

Sin embargo, muchas organizaciones todavía dependen de **copias de seguridad manuales**, lo que genera varios problemas:

* ⏳ Dependencia de intervención humana  
* ⚠️ Riesgo de errores en el proceso de backup  
* 📂 Duplicación innecesaria de archivos  
* 🔧 Infraestructuras poco automatizadas  

El proyecto **Automatización y Orquestación de Copias de Seguridad** propone una solución basada en **infraestructura virtualizada, automatización y almacenamiento distribuido**, permitiendo crear un sistema de backups **robusto, escalable y eficiente**.

La infraestructura permite:

* Automatizar completamente la creación de backups
* Reducir el consumo de almacenamiento mediante **deduplicación**
* Centralizar la gestión de copias de seguridad
* Garantizar disponibilidad de datos mediante **replicación**

---

# 🎯 Objetivos

## Objetivo principal

Diseñar e implementar una **infraestructura automatizada de copias de seguridad** capaz de gestionar y almacenar datos de múltiples servidores sin necesidad de intervención manual.

## Objetivos específicos

* Automatizar la infraestructura utilizando **Ansible**
* Crear entornos de prueba mediante **Vagrant**
* Implementar un sistema de backups eficiente con **BorgBackup**
* Reducir almacenamiento mediante **deduplicación**
* Crear un sistema de almacenamiento distribuido con **GlusterFS**
* Permitir acceso compartido mediante **Samba**
* Diseñar una arquitectura preparada para entornos empresariales

---

# 🏗 Arquitectura del sistema

La infraestructura está compuesta por **cinco máquinas virtuales**, cada una con un rol específico dentro del sistema.

| Máquina | Función | Servicios |
|-------|--------|--------|
| 🗄 MYSQL | Servidor de base de datos | MySQL |
| 🌐 WEB | Servidor web | Apache |
| 📂 STORAGE1 | Nodo de almacenamiento | GlusterFS + Samba |
| 📂 STORAGE2 | Nodo de almacenamiento | GlusterFS + Samba |
| 💾 BACKUP | Orquestación y backups | BorgBackup + Ansible |

Esta arquitectura permite construir un entorno **modular y escalable**, donde cada servicio se encuentra separado y puede evolucionar de forma independiente.

---

# 🖥 Infraestructura

## 💾 Servidor BACKUP

El servidor **BACKUP** es el encargado de gestionar las copias de seguridad y automatizar la infraestructura.

### Servicios instalados

**BorgBackup**

Herramienta avanzada de copias de seguridad que permite:

* Realizar backups incrementales
* Aplicar **deduplicación de datos**
* Reducir el uso de almacenamiento
* Restaurar archivos de forma rápida

**Ansible**

Sistema de automatización utilizado para:

* Configurar los servidores automáticamente
* Gestionar la infraestructura
* Mantener consistencia entre máquinas
* Automatizar tareas de despliegue y mantenimiento

---

## 🗄 Servidor MYSQL

El servidor **MYSQL** aloja la base de datos utilizada por los servicios de la infraestructura.

Este servidor representa un caso real de **servicio crítico**, cuyos datos deben ser protegidos mediante el sistema de copias de seguridad.

### Servicio instalado

**MySQL**

Funciones principales:

* Almacenamiento estructurado de datos
* Soporte para aplicaciones web
* Integración futura con el servidor Apache

---

## 🌐 Servidor WEB

El servidor **WEB** ejecuta un servidor Apache que servirá como plataforma para una futura aplicación web conectada a la base de datos MySQL.

### Servicio instalado

**Apache**

Funciones:

* Servidor web para aplicaciones internas
* Punto de acceso a servicios web
* Integración con la base de datos del servidor MYSQL

---

## 📂 Cluster de almacenamiento

El sistema de almacenamiento está formado por **dos servidores conectados mediante GlusterFS**, creando un volumen replicado.

### Características del almacenamiento

* Replicación entre nodos
* Alta disponibilidad
* Escalabilidad horizontal
* Protección frente a fallos de nodo

### Configuración actual

El cluster utiliza:

* 2 servidores Ubuntu  
* Disco dedicado `/dev/sdb`  
* Sistema de archivos **XFS**  
* Brick de almacenamiento en `/gluster/brick1`

El volumen replicado se llama:
