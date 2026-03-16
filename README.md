### Sistema de orquestación centralizada para copias de seguridad empresariales

Infraestructura diseñada para **automatizar la protección de servicios críticos**, combinando **orquestación de sistemas, deduplicación de datos y almacenamiento distribuido** para mejorar la gestión de copias de seguridad en entornos empresariales.

---

# 📑 Índice

- Introducción
- Objetivos del proyecto
- Arquitectura del sistema
- Componentes de la infraestructura
- Flujo de funcionamiento de los backups
- Cluster de almacenamiento
- Diseño de alta disponibilidad
- Tecnologías utilizadas
- Entorno de despliegue
- Resultados esperados
- Autor

---

# 🔍 Introducción

La información digital constituye uno de los recursos más importantes dentro de cualquier organización. La pérdida de datos puede provocar **interrupciones del servicio, pérdida de información crítica o daños económicos**.

A pesar de ello, muchas infraestructuras continúan utilizando procesos manuales o herramientas poco integradas para gestionar las copias de seguridad.

Este tipo de prácticas genera varios inconvenientes:

- Dependencia de procesos manuales
- Mayor probabilidad de errores humanos
- Falta de centralización en la gestión
- Uso poco eficiente del almacenamiento

Este proyecto propone la creación de una **plataforma automatizada de gestión de backups**, capaz de centralizar y coordinar copias de seguridad de múltiples servidores dentro de una infraestructura simulada de empresa.

La solución implementa **automatización mediante orquestación, almacenamiento replicado y deduplicación de datos**, con el objetivo de acercarse al funcionamiento de sistemas utilizados en entornos reales.

---

# 📌 Objetivos del proyecto

## Objetivo principal

Diseñar e implementar un **sistema automatizado de gestión de copias de seguridad** que permita proteger múltiples servidores dentro de una infraestructura empresarial.

## Objetivos específicos

- Automatizar la ejecución de copias de seguridad
- Implementar un sistema eficiente de backup con **deduplicación de datos**
- Centralizar el almacenamiento de las copias
- Simular una infraestructura empresarial realista
- Diseñar un sistema **escalable y tolerante a fallos**
- Integrar almacenamiento distribuido para mejorar disponibilidad

---

# 💻 Arquitectura del sistema

La infraestructura utilizada en el proyecto está compuesta por **cinco máquinas virtuales**, cada una con una función específica dentro del sistema.

| Máquina | Función | Servicios |
|--------|--------|--------|
| 🎛 ORCHESTRATOR | Servidor de automatización | Ansible, BorgBackup |
| 🗃 MYSQL | Servidor de base de datos | MySQL |
| 🌍 WEB | Servidor web | Apache |
| 📚 STORAGE1 | Nodo de almacenamiento | GlusterFS, Samba |
| 📚 STORAGE2 | Nodo de almacenamiento | GlusterFS, Samba |

Esta arquitectura permite separar claramente las capas de **servicios, control y almacenamiento** dentro del sistema.

---

# 🖇️ Componentes de la infraestructura

## Servidor de orquestación

El servidor **ORCHESTRATOR** actúa como el nodo central del sistema.

Desde este servidor se ejecutan las tareas automatizadas que gestionan las copias de seguridad de los diferentes servidores de la infraestructura.

Funciones principales:

- Ejecutar tareas automatizadas mediante Ansible
- Gestionar repositorios de backup con BorgBackup
- Coordinar copias de seguridad de múltiples nodos
- Aplicar deduplicación para optimizar el almacenamiento
- Enviar los backups al sistema de almacenamiento distribuido

Este servidor representa el **elemento principal del proyecto**, ya que puede desplegarse directamente en una infraestructura empresarial.

---

## Servidor web

El servidor **WEB** representa una aplicación web corporativa dentro del entorno.

En este nodo se ejecuta un servidor **Apache**, simulando los archivos y servicios que normalmente deben ser protegidos mediante copias de seguridad.

Los directorios del servidor web forman parte del sistema automatizado de backup.

---

## Servidor de base de datos

El servidor **MYSQL** simula un sistema de base de datos empresarial.

Las bases de datos contienen información crítica, por lo que el sistema automatizado realiza **copias periódicas** para garantizar su recuperación en caso de incidente.

---

# 💾 Cluster de almacenamiento

El almacenamiento de los backups se realiza mediante un **cluster basado en GlusterFS**, formado por dos nodos de almacenamiento.

Este sistema permite crear un volumen replicado entre varios servidores.

Entre sus ventajas destacan:

- Replicación automática de datos
- Tolerancia frente a fallos
- Escalabilidad horizontal
- Separación entre almacenamiento y procesamiento

## Configuración del almacenamiento

Cada nodo dispone de:

- Disco dedicado: `/dev/sdb`
- Sistema de archivos: **XFS**

Directorio del brick:
