### Sistema de orquestación centralizada para copias de seguridad empresariales

Infraestructura diseñada para automatizar la protección de servicios críticos, combinando orquestación de sistemas, deduplicación de datos y almacenamiento distribuido para mejorar la gestión de copias de seguridad en entornos empresariales.

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

La información digital constituye uno de los recursos más importantes dentro de cualquier organización. La pérdida de datos puede provocar interrupciones del servicio, pérdida de información crítica o daños económicos.

A pesar de ello, muchas infraestructuras continúan utilizando procesos manuales o herramientas poco integradas para gestionar las copias de seguridad.

Este proyecto propone la creación de una plataforma automatizada de gestión de backups, capaz de centralizar y coordinar copias de seguridad de múltiples servidores dentro de una infraestructura simulada de empresa.

La solución implementa automatización mediante orquestación, almacenamiento replicado y deduplicación de datos, con el objetivo de acercarse al funcionamiento de sistemas utilizados en entornos reales.

---

# 📌 Objetivos del proyecto

## Objetivo principal

Diseñar e implementar un sistema automatizado de gestión de copias de seguridad que permita proteger múltiples servidores dentro de una infraestructura empresarial.

## Objetivos específicos

- Automatizar la ejecución de copias de seguridad
- Implementar un sistema eficiente de backup con deduplicación de datos
- Centralizar el almacenamiento de las copias
- Simular una infraestructura empresarial realista
- Diseñar un sistema escalable y tolerante a fallos
- Integrar almacenamiento distribuido para mejorar disponibilidad

## Arquitectura de la red

La infraestructura utilizada en el proyecto está compuesta por cinco máquinas virtuales:
||||
|-------|--------|--------|
| 🎛 ORCHESTRATOR | Servidor de automatización | Ansible, BorgBackup |
| 🗃 MYSQL | Servidor de base de datos | MySQL |
| 🌍 WEB | Servidor web | Apache |
| 📚 STORAGE1 | Nodo de almacenamiento | GlusterFS, Samba |
| 📚 STORAGE2 | Nodo de almacenamiento | GlusterFS, Samba |

---

# 🖇️ Componentes de la infraestructura

## Servidor de orquestación

El servidor ORCHESTRATOR actúa como el nodo central del sistema.

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

El servidor WEB representa una aplicación web corporativa dentro del entorno.

En este nodo se ejecuta un servidor Apache, simulando los archivos y servicios que normalmente deben ser protegidos mediante copias de seguridad.

Los directorios del servidor web forman parte del sistema automatizado de backup.

---

## Servidor de base de datos

El servidor MYSQL simula un sistema de base de datos empresarial.

Las bases de datos contienen información crítica, por lo que el sistema automatizado realiza copias periódicas para garantizar su recuperación en caso de incidente.

---

# 💾 Cluster de almacenamiento

El almacenamiento de los backups se realiza mediante un cluster basado en GlusterFS, formado por dos nodos de almacenamiento.

Este sistema permite crear un volumen replicado entre varios servidores.

Entre sus ventajas destacan:

- Replicación automática de datos
- Tolerancia frente a fallos
- Escalabilidad horizontal
- Separación entre almacenamiento y procesamiento

## Configuración del almacenamiento

Cada nodo dispone de:

- Disco dedicado: `/dev/sdb`
- Sistema de archivos: XFS

Directorio del brick:
/gluster/brick1

Volumen replicado:
samba_vol

Punto de montaje:
/mnt/gluster


---

# 🔄 Flujo de funcionamiento de los backups

El proceso de copia de seguridad sigue los siguientes pasos:

1. El servidor de orquestación inicia la tarea programada.
2. Se ejecutan tareas remotas en los servidores objetivo.
3. BorgBackup genera copias optimizadas con deduplicación.
4. Las copias se almacenan en el cluster distribuido.
5. Los repositorios quedan disponibles para su restauración.

Este proceso permite reducir el espacio utilizado y mantener diferentes versiones de los datos.

---

# 📂 Acceso al almacenamiento

El volumen creado mediante GlusterFS se comparte utilizando Samba, lo que permite que los backups puedan almacenarse y accederse desde distintos sistemas dentro de la red.

Esto facilita:

- Compatibilidad con diferentes sistemas operativos
- Centralización del almacenamiento
- Integración sencilla en redes empresariales

---

# 🛡 Diseño de alta disponibilidad

Para entornos de producción, el sistema puede ampliarse incorporando mecanismos de alta disponibilidad.

## Cluster Samba con CTDB

Mediante CTDB es posible implementar un cluster Samba activo-activo, donde ambos nodos pueden ofrecer el servicio simultáneamente.

Ventajas:

- Balanceo de carga
- Mayor disponibilidad del servicio
- Failover automático

## Dirección IP virtual

El acceso al almacenamiento puede realizarse mediante una IP virtual compartida, de modo que los clientes continúan accediendo al sistema incluso si uno de los nodos falla.

---

# 🧰 Tecnologías utilizadas

| Tecnología | Función |
|-----------|-----------|
| Ubuntu Server | Sistema operativo |
| Ansible | Automatización y orquestación |
| BorgBackup | Sistema de copias de seguridad |
| GlusterFS | Almacenamiento distribuido |
| Samba | Compartición de archivos |
| MySQL | Base de datos |
| Apache | Servidor web |
| Vagrant | Gestión de máquinas virtuales |

---

# 🖥 Entorno de despliegue

La infraestructura del proyecto se ejecuta en un entorno de laboratorio basado en máquinas virtuales gestionadas mediante Vagrant.

Esto permite reproducir fácilmente toda la arquitectura del sistema en diferentes equipos.

Entre sus ventajas se encuentran:

- Reproducibilidad del entorno
- Facilidad de despliegue
- Simulación de infraestructuras empresariales

---

# 📊 Resultados esperados

Con la implementación de este sistema se pretende obtener:

- Automatización completa de copias de seguridad
- Optimización del almacenamiento mediante deduplicación
- Infraestructura tolerante a fallos
- Plataforma escalable para múltiples servidores
- Simulación realista de un entorno empresarial

El resultado final es una plataforma centralizada de protección de datos, preparada para gestionar backups de diferentes servicios dentro de una infraestructura empresarial.

---

# 👨‍💻 Autor

Proyecto académico desarrollado por:
