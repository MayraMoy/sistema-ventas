# Sistema de Gestión de Ventas – SaaS para Comercios

## Información general

**Proyecto:** Sistema de Gestión de Ventas para Kioscos y Pequeños Comercios  
**Tipo:** Producto SaaS (Software as a Service)  
[![Estado](https://img.shields.io/badge/Estado-En%20Desarrollo-yellow.svg)]()
[![Versión](https://img.shields.io/badge/Versión-1.0-green.svg)]()

Este repositorio **no contiene código fuente**.  
Su objetivo es **documentar, presentar y explicar el proyecto**, su alcance funcional y su arquitectura a alto nivel.

---

## Descripción del producto

El Sistema de Gestión de Ventas es una plataforma orientada a **pequeños comercios** (kioscos, almacenes, panaderías) que permite:

- Registrar ventas de forma simple y rápida  
- Gestionar productos y stock  
- Controlar el estado del negocio en tiempo real  
- Generar comprobantes de venta  
- Operar bajo un modelo de suscripción (SaaS)

El sistema está diseñado para ser:
- Fácil de usar
- Rápido en la operación diaria
- Escalable a múltiples comercios
- Seguro en el manejo de datos

---

## Público objetivo

- Kioscos  
- Almacenes  
- Panaderías  
- Pequeños comercios minoristas  

Usuarios no técnicos que necesitan **una herramienta práctica**, no un sistema contable complejo.

---

## Roles del sistema

El sistema contempla **tres roles claramente diferenciados**:

| Rol | Descripción |
|---|---|
| **Administrador SaaS** | Rol interno. Gestiona clientes, planes y estados del sistema |
| **Cliente (Comercio)** | Dueño del negocio que contrata el servicio |
| **Empleado** | Usuario operativo encargado de registrar ventas |

Cada rol tiene permisos y vistas específicas.

---

## Funcionalidades principales

### Cliente (Comercio)

- Gestión de inventario (alta, edición y consulta de productos)
- Control de stock y alertas de stock crítico
- Registro de ventas en tiempo real
- Generación de comprobantes
- Configuración de datos del comercio
- Historial de ventas y reportes básicos

### Administrador SaaS

- Gestión de clientes
- Control de planes y suscripciones
- Activación y desactivación de cuentas
- Supervisión general del sistema

---

## Diseño de interfaz

El proyecto cuenta con prototipos funcionales de interfaz que incluyen:

- Home del comercio
- Login / Registro
- Panel del administrador SaaS

Las capturas y diseños se encuentran en la carpeta `/ui`.

> Las imágenes representan el diseño conceptual y pueden ajustarse durante el desarrollo.

---

## Arquitectura (alto nivel)

El sistema está construido bajo una arquitectura **cliente-servidor**, con una clara separación de responsabilidades.

- Frontend: aplicación web
- Backend: API REST
- Base de datos: relacional
- Autenticación basada en tokens
- Aislamiento de datos por comercio (multi-tenant)

No se exponen detalles internos ni código en este repositorio.

---

## Tecnologías utilizadas (resumen)

- Frontend: React  
- Backend: Node.js + Express  
- Base de datos: MySQL  
- ORM: Prisma  
- Autenticación: JWT  

> La implementación técnica completa se mantiene en repositorios privados.

---

## Estado del proyecto

- Arquitectura definida
- Requisitos funcionales documentados
- Diseño de interfaz en validación
- Desarrollo del MVP en curso

⚠️ El sistema **no se encuentra en producción**.

---

## Roadmap (resumido)

- [x] Definición de roles y alcance  
- [x] Diseño inicial de interfaz  
- [ ] Autenticación y autorización  
- [ ] Gestión de productos y stock  
- [ ] Registro de ventas  
- [ ] Panel administrativo SaaS  
- [ ] Pruebas de integración  
- [ ] Despliegue inicial  

---

## Sobre este repositorio

Este repositorio existe con fines de:

- Documentación del proyecto  
- Presentación conceptual  
- Portfolio profesional  
- Validación funcional  

El código fuente se mantiene **privado**.

---

## Contacto

Para consultas, feedback o interés en el proyecto:

📩 *(mayrayazminmoyano@gmail.com)*
📩 *(alejandroloyola0803@gmail.com)*

---

## Nota final

Este proyecto está siendo desarrollado con un enfoque **profesional**, priorizando:

- Claridad funcional  
- Escalabilidad  
- Seguridad  
- Experiencia de usuario real  
