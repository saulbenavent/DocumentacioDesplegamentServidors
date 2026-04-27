# 1. Manual de Infraestructura de IT

Bienvenido a la documentación técnica oficial de la infraestructura de red de nuestra organización. Este portal ha sido diseñado utilizando la metodología **SCRUM** para garantizar una implementación robusta, escalable y segura de todos los servicios críticos de la empresa.

---

## 1.1. Resumen del Proyecto
El objetivo principal de este proyecto es la simulación y despliegue de un entorno empresarial completo, integrando servicios de red, almacenamiento centralizado y protocolos de seguridad avanzada bajo un entorno de red interna controlada.

### 1.1.1. Servicios Core implementados:
* **Servidor Web:** Hosting de aplicaciones corporativas y portal de información.
* **Servidor DNS:** Resolución de nombres de dominio internos (`empresa.local`).
* **Servidor DHCP:** Gestión automatizada de direccionamiento IP.
* **Servidor NAS:** Almacenamiento centralizado y gestión de recursos compartidos.
* **Seguridad Centralizada:** Protección perimetral, control de accesos y endurecimiento de sistemas (Hardening).

---

## 1.2. Equipo de Trabajo y Roles
Para la ejecución de este proyecto, hemos dividido las responsabilidades siguiendo roles técnicos reales en el sector IT:

| Rol | Responsable | Usuario de Sistema | Especialidad |
| :--- | :--- | :--- | :--- |
| **Scrum Master** | Saül Benavent | `admin-scrum` | Gestión de Trello, Sprints y QA |
| **Admin. Web** | Samuel Puma | `admin-web` | Apache/Nginx, PHP y Despliegue |
| **Admin. de Red (DHCP + DNS)** | Alberto Belda | `admin-red` | Protocolos DHCP y DNS |
| **Admin. Almacenamiento (NAS)** | Jacob Vila | `admin-nas` | Samba, Cuotas y Permisos |
| **Admin. de Seguridad** | Blai Camús | `admin-sec` | Firewall, Fail2Ban y SSH Hardening |

---

## 1.3 Metodología de Trabajo (SCRUM)
Hemos organizado el desarrollo en tres fases críticas para asegurar la entrega de un producto mínimo viable de alta calidad:

1.  **Sprint 1 (Cimientos):** Virtualización, diseño de arquitectura de red y despliegue de la estructura MkDocs.
2.  **Sprint 2 (Desarrollo):** Configuración técnica individual de cada servicio y despliegue de funcionalidades.
3.  **Sprint 3 (Validación y Documentación):** Pruebas de interconectividad, capturas de comandos y redacción técnica final.

> 💡 **Nota para el Administrador:** Todas las tareas, progresos y cuellos de botella se encuentran documentados históricamente en nuestro tablero de **Trello**.

---

## 1.4 Contenido de la Documentación
Navega a través del menú lateral para consultar los detalles específicos de cada módulo:

* **[Servidores](Red_Alberto.md):** Guía de instalación y configuración de Web, DHCP y DNS.
* **[Almacenamiento (NAS)](Nas_Jacob.md):** Estructura de carpetas compartidas y políticas de acceso.
* **[Seguridad y Hardening](Seguridad_Blai.md):** Auditoría de puertos, reglas de firewall y protección de fuerza bruta.
* **[Gestión de Usuarios](Web_Samu.md):** Matriz de permisos y perfiles administrativos.

---

*Última actualización: Abril 2026 - Generado por el Equipo de Infraestructura.*
