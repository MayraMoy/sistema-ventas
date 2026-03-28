# **Arquitectura de APIs (REVISAR)**

## Versión v1 – MVP (Definición contractual de APIs)

## **1\. Objetivo del documento**

Definir de forma clara y ordenada **la arquitectura de las APIs del sistema**, estableciendo:

* Endpoints disponibles  
* Rol autorizado para cada acción  
* Propósito funcional de cada API

Este documento actúa como **contrato técnico** entre backend, frontend y testing.

## **2\. Roles del sistema**

| Rol | Descripción |
| ----- | ----- |
| **ADMIN\_SAAS** | Administradores del sistema (desarrolladores / dueños del SaaS) |
| **CLIENTE** | Propietario del kiosco o comercio que contrata el sistema |
| **EMPLEADO** | Personal del comercio que registra ventas |

## **3\. Convenciones generales de la API**

* Base URL: /api  
* Autenticación: JWT  
* Todas las rutas (excepto login y planes públicos) requieren token  
* El aislamiento de datos se realiza por negocioId desde el backend  
* El cliente nunca envía ni modifica el negocioId

## **4\. APIs de Autenticación**

| Método | Endpoint | Rol | Descripción |
| ----- | ----- | ----- | ----- |
| **POST** | /auth/login | Todos | Inicio de sesión y generación de JWT |
| **GET** | /auth/me | Todos | Obtiene datos del usuario autenticado |
| **POST** | /auth/logout | Todos | Cierre de sesión |

## **5\. APIs del Administrador del SaaS (ADMIN\_SAAS)**

### **5.1 Gestión de Clientes**

| Método | Endpoint | Descripción |
| ----- | ----- | ----- |
| **GET** | /admin/clientes | Listado de clientes del sistema |
| **GET** | /admin/clientes/:id | Detalle de cliente |
| **PUT** | /admin/clientes/:id/estado | Activar / suspender cliente |
| **PUT** | /admin/clientes/:id/plan | Asignar o cambiar plan |

### **5.2 Métricas y control del SaaS**

| Método | Endpoint | Descripción |
| ----- | ----- | ----- |
| **GET** | /admin/metricas | Métricas globales (clientes, ingresos, estados) |

## **6\. APIs de Planes y Suscripciones**

### **6.1 Planes (públicos)**

| Método | Endpoint | Rol | Descripción |
| ----- | ----- | ----- | ----- |
| **GET** | /planes | Público | Listado de planes disponibles |

### **6.2 Suscripciones**

| Método | Endpoint | Rol | Descripción |
| ----- | ----- | ----- | ----- |
| **POST** | /suscripciones | CLIENTE | Contratar plan |
| **GET** | /suscripciones | CLIENTE | Historial de suscripciones |

## **7\. APIs del Cliente (Gestión del comercio)**

### **7.1 Productos**

| Método | Endpoint | Rol | Descripción |
| ----- | ----- | ----- | ----- |
| **POST** | /productos | CLIENTE | Crear producto |
| **GET** | /productos | CLIENTE | Listar productos |
| **GET** | /productos/:id | CLIENTE | Detalle de producto |
| **PUT** | /productos/:id | CLIENTE | Editar producto |
| **DELETE** | /productos/:id | CLIENTE | Eliminar producto |

### **7.2 Stock**

| Método | Endpoint | Rol | Descripción |
| ----- | ----- | ----- | ----- |
| **GET** | /stock | CLIENTE | Consulta de stock |
| **PUT** | /stock/:productoId | CLIENTE | Ajuste manual de stock |

### **7.3 Reportes**

| Método | Endpoint | Rol | Descripción |
| ----- | ----- | ----- | ----- |
| **GET** | /reportes/ventas | CLIENTE | Reporte de ventas |
| **GET** | /reportes/stock | CLIENTE | Reporte de stock |

**7.4 Gestión de Caja**

| Método | Endpoint | Rol | Descripción |
| :---- | :---- | :---- | :---- |
| **POST** | /caja/abrir | CLIENTE, EMPLEADO | Apertura de caja (inicio de turno) con monto inicial |
| **POST** | /caja/cerrar | CLIENTE, EMPLEADO | Cierre de caja y registro de arqueo (monto real vs sistema) |
| **GET** | /caja/estado | CLIENTE, EMPLEADO | Consulta de estado (Abierta/Cerrada) y totales parciales |

## 

## **7.5 Dashboard y Métricas**

| Método | Endpoint | Rol | Descripción |
| :---- | :---- | :---- | :---- |
| **GET** | /dashboard/resumen | CLIENTE | Resumen general del negocio (Ventas, Ticket Promedio, Top Productos) |

## **8\. APIs del Empleado**

### **8.1 Ventas**

| Método | Endpoint | Rol | Descripción |
| ----- | ----- | ----- | ----- |
| **POST** | /ventas | EMPLEADO | Registrar venta **(Requiere caja ABIERTA)** |
| **GET** | /ventas | EMPLEADO | Listar ventas propias |
| **GET** | /ventas/:id | EMPLEADO | Detalle de venta |

### **8.2 Comprobantes**

| Método | Endpoint | Rol | Descripción |
| ----- | ----- | ----- | ----- |
| **GET** | /comprobantes/:ventaId | EMPLEADO | Obtener comprobante |

## **9\. Reglas transversales (middlewares)**

| Middleware | Función |
| ----- | ----- |
| **authJWT** | Validación de token |
| **validarRol** | Control de permisos por rol |
| **validarClienteActivo** | Bloqueo por pago vencido |
| **validarPlan** | Límite según plan |
| **aislamientoNegocio** | Aislamiento de datos |

## **10\. Mensajes de error**

| Nombre del error | ¿Cuándo ocurren? | ¿Qué debe pasar? | Respuesta esperada |
| :---- | :---- | :---- | :---- |
| Errores de autenticación | No se envía JWT JWT inválido JWT expirado | La request se corta **antes de llegar al controller** Nunca ejecuta lógica de negocio | 401 Unauthorized  |
| Errores de autorización (roles) | Rol incorrecto para la acción Usuario intenta hacer algo que no le corresponde |  | 403 Forbidden  |
| Errores de estado del cliente (regla SaaS) | Cliente vencido Cliente suspendido  |  | 403 Forbidden  |
| Errores de aislamiento multi-cliente (CRÍTICO) | Usuario intenta acceder a datos de otro negocio ID válido pero de otro cliente  |  | 403 Forbidden  |
| Errores de validación de datos | Campos faltantes Tipos incorrectos Valores fuera de rango  |  | 422 Unprocessable Entity  |
| Errores de reglas de negocio | Venta sin stock Supera límite del plan Producto inactivo |  | 409 Conflict  |
| Errores de recursos inexistentes | ID no existe Recurso eliminado  |  | 404 Not Found  |
| Errores de concurrencia (menos obvio) | Dos ventas al mismo tiempo Stock insuficiente por race condition  |  | 409 Conflict  |
| Errores internos (los únicos 500\) | Falla inesperada Bug real Error de infraestructura  |  | 500 Internal Server Error  |
| Errores de flujo operativo | Intento de venta con caja cerrada | Bloquear operación hasta que se abra caja | 409 Conflict |

## **11 Consideraciones finales**

* Este documento define exclusivamente el **MVP**  
* No incluye integraciones externas ni pagos automáticos  
* Es la base directa para tests de integración y OpenAPI

**Fin del documento**
