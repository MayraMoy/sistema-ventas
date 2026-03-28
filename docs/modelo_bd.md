# ***Documento de Diseño de Base de Datos***

## **Proyecto: Sistema de Gestión Comercial SaaS (Kioscos)**

* ### Versión: 2.0

* ### Fecha: 1 de Febrero de 2026

* ### Estado: Especificación Técnica Final

### 

### **1\. Introducción**

### El presente documento detalla la arquitectura de datos para un sistema de gestión comercial (SaaS) diseñado para kioscos y negocios minoristas. El sistema abarca la administración de usuarios, gestión de suscripciones, control de inventario escalable, puntos de venta (POS) y auditoría de caja.

### El diseño sigue un enfoque relacional normalizado, priorizando la integridad de los datos (para evitar errores financieros) y la escalabilidad (para soportar el crecimiento del negocio).

### 

### **2\. Stack Tecnológico y Justificación**

### Para la persistencia de datos se han seleccionado las siguientes tecnologías:

#### **2.1 Base de Datos: Relacional (MySQL 8.0+)**

### Se descartan bases de datos NoSQL por la naturaleza transaccional del sistema.

* ### **Integridad ACID:** Operaciones como "Vender" requieren atomicidad estricta (restar stock, sumar dinero y generar comprobante en un solo paso).

* ### **Estructura Definida:** Los comprobantes fiscales y los inventarios requieren tipos de datos estrictos (Decimales, Fechas) que SQL garantiza nativamente.


#### **2.2 Stack Seleccionado:**

* **Motor: MySQL (Versión 8.0+):** Seleccionado por su ubicuidad, soporte de JSON nativo (para configuraciones flexibles) y bajo costo en infraestructura cloud.  
* **ORM: Prisma:** Utilizado para la capa de acceso a datos. Provee seguridad de tipos (Type Safety) con TypeScript y facilita la gestión de migraciones de esquema.

#### **2.3 Capa de Acceso: Prisma ORM**

* ### **Type Safety:** Al trabajar con TypeScript, Prisma previene errores de tipos antes de compilar.

* ### **Migraciones:** Facilita el versionado de la base de datos a lo largo del ciclo de vida del software.

### 

### **3\. Implementación de Modelos**

#### **3.1 Modelo Conceptual**

El modelo conceptual representa de forma abstracta las entidades del dominio, enfocándose en "qué" se almacena y no "cómo". Se identifican los módulos principales:

* **Gestión de Acceso:** Usuarios, Roles y Permisos.  
* **SaaS:** Planes y Suscripciones.  
* **Negocio:** Configuración y Sucursales.  
* **Inventario:** Productos, Categorías y Stock Físico.  
* **Operativa:** Cajas, Sesiones (Turnos) y Ventas.

### **3.2 Diagrama Entidad-Relación (DER)**

![][image1]

#### 

#### **3.2 Modelo Lógico**

El modelo lógico define las reglas de negocio estructurales. Las decisiones de arquitectura clave incluyen:

* **Escalabilidad de Stock:** La relación **Productos a Stock** es de uno a muchos (1:N). Esto permite que un mismo producto exista en múltiples ubicaciones (Depósito, Sucursal 1\) simultáneamente.  
* **Auditoría:** Se introduce la entidad **MovimientosStock** para trazar cada cambio de inventario, y el **DetalleVenta** guarda una "foto" (snapshot) del precio y nombre del producto al momento de la venta.  
* **Control de Caja:** Se reemplaza el concepto de "Cierre diario" por **Sesiones de Caja**, vinculando cada venta a un turno específico mediante SesionId.

#### **3.3 Modelo Lógico (Resumen)**

* ### ***Usuarios y Roles:*** Estructura jerárquica donde un Usuario tiene un Rol, y el Rol define sus Permisos.

* ### **Multi-Tenancy:** La entidad Negocios actúa como separador lógico de datos.

* ### **Inventario:** Separación de Productos (Catálogo) y Stock (Existencia física), permitiendo múltiples depósitos.

* ### **Caja Dinámica:** Implementación de SesionesCaja para manejar turnos rotativos, reemplazando el cierre diario estático.

#### **3.2 Normalización**

### El esquema cumple con la 3ra Forma Normal (3NF), eliminando redundancias, salvo en las tablas de DetalleVenta y Comprobantes, donde se aplica desnormalización controlada (Snapshotting) para preservar el historial de precios y datos fiscales ante cambios futuros.

* **Excepción Estratégica:** En DetalleVenta se duplica el PrecioUnitario y Nombre. Aunque dependen del Producto, se guardan en la venta para mantener la integridad histórica si el catálogo cambia de precio en el futuro.

### **4\. Relaciones entre Entidades** 

**Módulo de Accesos (IAM)**

* **Usuarios → Roles:** (N:1) Muchos usuarios pueden tener el mismo rol (ej: 3 cajeros).  
* **Roles → Permisos:** (1:N) Un Rol posee muchos Permisos asignados directamente. 

**Módulo SaaS**

* **Plan → Suscripcion:** (1:N) Un plan tiene muchas suscripciones.  
* **Usuarios ↔ Suscripcion:** (N:M) Gestionado por UsuarioSuscripcion.

**Módulo de Negocio**

* **Usuarios → Negocios:** (1:N) Un dueño, muchos negocios.  
* **Negocios → ConfigNegocio:** (1:1) Relación estricta.  
* **Negocios → Empleados:** (1:N).

**Módulo de Inventario**

* **Negocios → Categorias:** (1:N).  
* **Categorias → Productos:** (1:N).  
* **Productos → Stock:** (1:N).  
* **Stock → MovimientosStock:** (1:N).

**Módulo de Caja y Ventas**

* **Negocios → Caja:** (1:N).  
* **Caja → SesionesCaja:** (1:N).  
* **SesionesCaja → Ventas:** (1:N).  
* **SesionesCaja → DetallesCierreCaja:** (1:N).  
* **Empleados → Ventas:** (1:N).  
* **Ventas → DetalleVenta:** (1:N).  
* **Ventas → Comprobantes:** (1:1).  
* **Usuarios (Clientes) → Ventas:** (1:N, Opcional).


### 5\. Modelo Físico (Diccionario de Datos)

### 

# **5.1 Modelo Físico** 

### **Módulo A:** Gestión de Accesos

### **Tabla:** Usuarios

### ***Personas con acceso al sistema.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador único.** |
| **RolId** | **INT (FK)** | **Relación (N:1). Define el perfil (ej: Admin, Cajero).** |
| **Email** | **VARCHAR** | **(Unique) Credencial de acceso.** |
| **PasswordHash** | **VARCHAR** | **Seguridad (Bcrypt/Argon2).** |
| **Nombre** | **VARCHAR** | **Nombre de pila.** |
| **TelefonoContacto** | **VARCHAR** | **Para soporte urgente o 2FA.** |
| **EstadoUsuario** | **ENUM** | **'Activo', 'Suspendido', 'Baja'.** |
| **FechaCreacion** | **DATETIME** | **Auditoría de antigüedad del usuario.** |

### 

### **Tabla:** Roles

### ***Perfiles de usuario.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **Nombre** | **VARCHAR** | **Etiqueta del rol.** |
| **Descripcion** | **VARCHAR** | **Detalle funcional del alcance.** |

### 

### **Tabla:** Permisos

### ***Capacidades del sistema.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **RolId** | **INT (FK)** | **Relación directa 1:N.** |
| **Nombre** | **VARCHAR** | **Slug técnico (Stock.Editar).** |

### 

### **Módulo** B: SaaS y Suscripciones

### **Tabla:** Plan

### ***Oferta comercial.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **Nombre** | **VARCHAR** | **Nombre comercial ("Pro Mensual").** |
| **Descripcion** | **TEXT** | **Lista de beneficios para mostrar en el frontend.** |
| **Precio** | **DECIMAL** | **Costo del plan.** |
| **DuracionDias** | **INT** | **Vigencia (30, 365 días).** |

### 

### **Tabla:** Suscripcion

### ***Contrato activo.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **PlanId** | **INT (FK)** | **Qué se contrató.** |
| **FechaInicio** | **DATETIME** | **Alta del servicio.** |
| **FechaFin** | **DATETIME** | **Vencimiento / Corte de servicio.** |
| **Estado** | **ENUM** | **'Activa', 'Pendiente', 'Cancelada'.** |

### 

### **Tabla:** UsuarioSuscripcion

### ***Historial y vinculación.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **UsuarioId** | **INT (FK)** | **Cliente.** |
| **SuscripcionId** | **INT (FK)** | **Contrato asociado.** |

### 

### **Módulo C:** Estructura del Negocio

### **Tabla:** Negocios

### ***El comercio (Tenant).***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador maestro.** |
| **UsuarioId** | **INT (FK)** | **Dueño.** |
| **ConfigNegocioId** | **INT (FK)** | **Relación 1:1.** |
| **Nombre** | **VARCHAR** | **Nombre Fantasía.** |
| **Direccion** | **VARCHAR** | **Domicilio comercial (para tickets).** |
| **FechaAlta** | **DATETIME** | **Antigüedad del cliente.** |

### 

### **Tabla:** ConfigNegocio

### ***Reglas operativas.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **NegocioId** | **INT (FK)** | **Relación inversa 1:1.** |
| **ConfigCaja** | **JSON** | **Guarda monto mínimo, cierre ciego, etc.** |
| **ConfigComprobante** | **JSON** | **Guarda logo, pie de página, tipo de letra.** |

### 

### **Tabla:** Empleados

### ***Personal.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **NegocioId** | **INT (FK)** | **Lugar de trabajo.** |
| **Nombre** | **VARCHAR** | **Nombre real.** |
| **Apellido** | **VARCHAR** | **Apellido real.** |
| **Estado** | **ENUM** | **'Activo', 'Baja'.** |
| **FechaAlta** | **DATETIME** | **Fecha de ingreso laboral.** |

### 

### **Módulo D:** Inventario y Logística

### **Tabla:** Categorias

### ***Agrupación.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **NegocioId** | **INT (FK)** | **Tenant.** |
| **Nombre** | **VARCHAR** | **Etiqueta ("Golosinas").** |
| **Descripcion** | **VARCHAR** | **Detalle interno.** |

### 

### **Tabla:** Productos

### ***Catálogo.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **CategoriaId** | **INT (FK)** | **Rubro.** |
| **Nombre** | **VARCHAR** | **Descripción corta.** |
| **Descripcion** | **TEXT** | **Descripción detallada (ingredientes, etc).** |
| **Precio** | **DECIMAL** | **Precio lista.** |
| **Estado** | **ENUM** | **'Activo', 'Discontinuado'.** |

### 

### **Tabla:** Stock

### ***Existencia física (1:N).***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **ProductoId** | **INT (FK)** | **Referencia al catálogo.** |
| **CantidadDisponible** | **DECIMAL** | **Stock actual real.** |
| **PuntoReposicion** | **DECIMAL** | **lerta de stock mínimo.** |
| **Ubicacion** | **VARCHAR** | **Necesario para el 1:N (Sucursal/Depósito).** |

### 

### **Tabla:** MovimientosStock

### ***Auditoría.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **StockId** | **INT (FK)** | **Qué lote se modificó.** |
| **Cantidad** | **DECIMAL** | **Cuánto entró o salió.** |
| **Motivo** | **VARCHAR** | **Venta, Compra, Ajuste.** |
| **Fecha** | **DATETIME** | **Timestamp.** |

### 

### **Módulo E:** Operativa de Caja y Ventas

### **Tabla:** Caja

### ***Terminal.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **NegocioId** | **INT (FK)** | **Propiedad.** |
| **Nombre** | **VARCHAR** | **Identificación ("Caja 1").** |
| **Estado** | **ENUM** | **'Abierta', 'Cerrada'.** |

### 

### **Tabla:** SesionesCaja

### ***Turnos (Reemplaza CierreCaja).***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador (Equivale a CierreCajaId).** |
| **CajaId** | **INT (FK)** | **Terminal.** |
| **EmpleadoId** | **INT (FK)** | **Responsable.** |
| **FechaInicio** | **DATETIME** | **Apertura.** |
| **FechaFin** | **DATETIME** | **Cierre.** |
| **TotalFinal** | **DECIMAL** | **Recaudación total calculada.** |
| **Descripcion** | **VARCHAR** | **Observaciones del cierre.** |

### 

### **Tabla:** DetallesCierreCaja

### ***Arqueo (Dinero contado).***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **SesionId** | **INT (FK)** | **Link al turno cerrado.** |
| **TipoPago** | **ENUM** | **Efectivo, Tarjeta (Separa los montos).** |
| **MontoDeclarado** | **DECIMAL** | **Lo que contó el cajero.** |
| **Diferencia** | **DECIMAL** | **Diferencia vs Sistema.** |

### 

### **Tabla:** Ventas

### ***Cabecera Ticket.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **SesionId** | **INT (FK)** | **Vincula al turno (Reemplaza la lógica vieja).** |
| **NegocioId** | **INT (FK)** | **Tenant.** |
| **EmpleadoId** | **INT (FK)** | **Vendedor.** |
| **Fecha** | **DATETIME** | **Timestamp.** |
| **Total** | **DECIMAL** | **Importe a cobrar.** |

### 

### **Tabla:** DetalleVenta

### ***Renglones.***

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **VentaId** | **INT (FK)** | **Ticket padre.** |
| **ProductoId** | **INT (FK)** | **Ítem vendido.** |
| **Cantidad** | **DECIMAL** | **Unidades.** |
| **PrecioUnitario** | **DECIMAL** | **Precio al público al momento de la venta.** |
| **CostoHistorico** | **DECIMAL** | **Vital para calcular la ganancia neta histórica.** |

### 

### **Tabla:** Comprobantes

### ***Fiscal.***

### 

| Campo | Tipo | Justificación / Razón de ser |
| :---- | :---- | :---- |
| **Id** | **INT (PK)** | **Identificador.** |
| **VentaId** | **INT (FK)** | **Relación 1:1.** |
| **Numero** | **VARCHAR** | **Numeración.** |
| **FechaEmision** | **DATETIME** | **Fecha fiscal.** |
| **Total** | **DECIMAL** | **Monto final.** |
| **DatosEmisor** | **JSON** | **Snapshot datos del negocio.** |
| **DatosComprador** | **JSON** | **Snapshot datos cliente.** |

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAoIAAAF1CAYAAAB8j4spAAAQAElEQVR4AeydBaDUxdrG3wElDEJCmkOjhCiSihxMxABMLMDW67X16lU/r3ptr3ntRLELxEIxwASRbiQVkJCQDoFvf+NdPRy3z+6e3bMPOmd3/9PPzH/mmXfeeafUdv0TAkJACAgBISAEhIAQyEkESpn+CQEhIASEQA4hoKoKASEgBP5EQETwTyz0TQgIASEgBISAEBACOYWAiGAONLeqKASEgBAQAkJACAiBUAiICIZCRc+EgBAQAkJACGQvAiq5EIgZARHBmKFSQCEgBISAEBACQkAIlCwERARLVnuqNrmKgOotBISAEBACQiABBEQEEwBNUYSAEBACQkAICAEhUJwIJCtvEcFkIal0hIAQEAJCQAgIASGQZQiICBZjg23atMmmTZtmU6dOtenTp9vmzZuLsTTKOhwCW7du9e0zZcoUmzFjhq1fvz5cUD0XAilEQEkLASEgBJKPgIhg8jGNKcVff/3VPvvsM/vxxx9tw4YNNm/ePPv8889tzZo1McVXoPQgAOn7+OOPbf78+bZx40bfXp9++qmtXbs2PQVQLkJACAgBISAEUohASokgkpSnnnrK7r///ox1DzzwgH355Ze2ffv2FMK8Y9IQv7ffftsqV65s3bp1s7Zt2/rPChUq2FtvveUJBzHS4bZs2WIDBgzI2Pah79BGkOR0thHYr1u3zl5//XXbddddrWvXrr6d+Kxbt64988wztmrVKoKlzJH/f//734xum4cfftgmTZqUMgyUsBAQAkJACKQWgZQSwddee80TnH79+lmmutNPP92Qzo0fPz61SAdS/+2337xE6eKLL7YePXpYx44drUyZMgEfs7Jly1qnTp3s8MMPN/wXLlxohPeeKfoDsYLo7L///hnbPvSbM844w+bOneu30FMExQ7JsoBZsmSJ3XPPPXbggQfaQQcdZOXKlfNhaK82bdpY37597d///rffMk5FO1GGG2+80Y4//viMbpvevXvbmDFjbOnSpR4f/RECQqBICCiyEEg7Aikjgtu2bbNKlSpZ48aNbY899shYV7VqVevQoYN99dVXKQV/9erVhlRr2LBh9sQTT9iee+4ZMr9atWrZo48+ah999JGNGDEipVvFkI3atWtbXl5exrYPfadKlSrWp08f+/DDD0NilsyH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MSRJSE45FczzZOajtNKLAJJJ2hJTP4cffrg398O1e+kthXITAkJACAgBIRAbAiKCseGUklDOOX9HMNeJQRacK36pVkoqmvWJxlcBpIC0KVJApL/xxVZoISAEhIAQEALpQ0BEMH1YKychIASEgBAQAkIgGxDIoTKKCOZQY6uqQkAICAEhIASEgBAoiEBWEUFO12KOY8tvv9m27dsL1kPfhYAQiIIA781vW7fab79tNU6nRwmea96qrxAQAkIgJxHIKiL4408L7MLL/2nnXXSlzZkzLycbTJUWAoki8OBDT9n1N99ht999r23Z8luiySieEBACQkAIlCAEsooIVq9WzXYpW8ac29ny6tctWjMothDIMQR6HNnNNm/ZavXyGlqZMjvnWO1VXSEgBISAEAiFQFYRwV12KW9NGta143t2N5lZCdWceiYEwiPQqGFDa5SXZ1067h8+kHyEQAlGQFUTAkLgrwhkFRGk+Kf0OdEOPKA9X+WEgBCIA4GyAWn62f36mKTpcYCmoEJACAiBEo5AVhBBFNtXrVplAwYMsJcGDrR77r7bPv/8c+OathLePqpekRBQZBDYunWrLV682G6//Xb7eOj7du+99xrX4HH4Cn85ISAEhIAQyF0EMp4IQgJnzJhhH374oXFbA5PYTTfdZFu2bLFPPvnEli5dmrutp5oLgSgIbNiwwbjCkPfnwgsvtFtvu83fizxhwgT74osvjPuQoyQhbyEgBIRA9iCgksaNQEYTQcjeW2+9ZT/88IN16dLFWrRoYc45K1++vHGNV/369T0Z/P77703SjbjbXhFKOAIrV660t99+25AIHnfccVa5cmVf4+rVq1uvXr38Yurjjz+22bNn++f6IwSEgBAQArmHQMYSQSaxRx991Pbaay87+OCDrU6dOju0TunSpT0x5D7XZcuW2aBBg/zEtkMg/RACOYrAxIkT7bnnnrNOnTr5RVTFihV3QKJs2bLGPdf77befffPNN4aEEOn7DoGy44dKKQSEgBAQAkVAIOOIIJPR3Llz7dVXX7WePXt6srfrrruGrWLVqlXtiCOOMCa6gQMH2po1a8KGlYcQKOkIIEUfOXKkjR071v72t79Zw4YNw56wd85ZvXr1vHTwq6++suHDh2sxVdI7iOonBISAECiEQEYRwbVr19p3331nI0aMsOOPP97y8vIKFdfMQjwpVaqU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rFmzxsaNG2foJGZek6lEQkAICAEhIASEQMYjkCUFTBoR5Nq3Zs2a2UEHHZT1rmvXrrbrrrvajBkzsqQZVUwhIASEgBAQAkJACMSPQNKI4NatW6106dLmnMt6V6pUKV+X3377LX5EFUMICIFcRUD1FgJCQAhkHQJJI4JZV3MVWAgIASEgBISAEBACOY6AiGBROoDiCgEhIASEgBAQAkIgixEQEczixlPRhYAQEAJCIL0IKDchUNIQEBEsaS2q+ggBISAEhIAQEAJCIEYERARjBErBchUB1VsICAEhIASEQMlFQESw5LataiYEhIAQEAJCQAjEi0COhRcRzLEGV3WFgBAQAkJACAgBIRBEIGVEcNu2bXbffffZ6tWrbd26dfbBBx/Yzz//bBs3brTNmzcbdge5lg5bfRhuHjt2rL+VBD/CEOfDDz80/PnNJ34bNmzw8X/88UdbtWqVj8MtI6TFJ+6XX36xu+++21588UVbvny5TZ061aezfv16o1yEDab5zDPPGL+3bNkSxESfQkAI5BYCqq0QEAJCIGcRSBkRBFGugnvzzTdtyZIltnDhQoPE8fuzzz6zL774wpYuXeoJIiRv+vTp9v7779sLL7zgP8eMGWMYdp41a5a9++679t1339nrr79ugwcP9t+HDh1qkyZNsiFDhtjHH39sU6ZMMYjjsGHDPEGsW7euNW7c2JPAAQMGGETz1Vdfta+++spfhTdo0CB/cwg3opDXzJkzKXKJcZDbElMZVUQICAEhIASEgBBICQIpJYJ5eXle8vfTTz9Z7dq1PQHbe++9jecLFiww7ieG6CGd++GHH6xp06YGKWvSpInhD0n75JNPrH79+v62Ep4dddRRtmLFCttrr728dBG/Vq1a2eTJk61GjRoGodtzzz09CeTuY/zr1KnjJZMHHnigQUInTJjgr8EjHFJC8qBMKUG4mBJFelpMWStbISAEhIAQEAJCIEsQSCkRLFu2rB1xxBFegge5g5R9/fXXXkrXvn17L9E75phjrEyZMgYp22233axmzZpWvnx522WXXWyPPfbw5A6St3btWoPQlStXzvsjRUTiOHr0aBs3bpxVrFjRFi9e7EkiRHFCgOyxLUw6bAcTFmlgy5YtrVatWl7qCNEkvx49etiXX37pt4ithPyjziWkKqqGEBACQqDICCgBISAEQiOQMiLIti4ksF69enb11VcbRBB38cUX23HHHWfNmjUzvvMMid4FF1zgJYU33nijD9urVy874YQTvDvrrLPssMMOs9NPP92TRr5369bNcBdddJH17t3bjjzySCPOrbfeahDOc845x6dftWpVu+yyyyw/P99IhzQ7duxo5513nrVt29auvPJKa926tXXv3t1LHUPDpKdCQAgIASEgBISAECh5CKSMCJY8qFSj7EJApRUCQkAICAEhIASiISAiGA0h+QsBISAEhIAQEAKZj4BKmBACIoIJwaZIQkAICAEhIASEgBDIfgREBLO/DVUDIZCrCKjeQkAICAEhUEQERASLCKCiCwEhIASEgBAQAkIgWxHILiKYrSir3EJACAgBISAEhIAQyEAEkkoEdZtFBrawiiQEhIAQyGIEVHQhIARSi0DSiCCGnjHyvHXrVn+bSDZ/Ug9gp058ygkBISAEhIAQEAJCoCQikDQi2KhRI/vmm2/s888/t+HDh2e148o57kHmGruS2OiZXSeVTggIASEgBISAEEgXAkkjgg0aNDCui4MQNmzY0LLZ7b333nb00Uf7q+zS1RDKRwgIASEgBIRATiKgShcrAkkjgtRi9913NwhhtjuuqNtpp52okpwQEAJCQAgIASEgBEosAkklgiUWJVVMCAiBZCKgtISAEBACQiBDEBARzJCGUDGEgBAQAkJACAgBIZBuBNJDBNNdK+UnBISAEBACQkAICAEhEBUBEcGoECmAEBACQkAIxIuAwgsBIZAdCIgIZkc7qZRCQAgIASEgBISAEEg6AiKCSYc0VxNUvYWAEBACQkAICIFsQ0BEMNtaTOUVAkJACAgBIZAJCKgMJQKBrCGC27Ztsy+//NKef/55e+mll+yHH34oEQ2gSggBISAEhIAQEAJCoLgQyAoiyN2/gwYNsuXLl9vpp59u3bt3t88++8ymTp1q27dvLy7slK8QyDUEVF8hIASEgBAoYQhkPBGcN2+evf7668atJVxhV7p0aatSpYqdeuqp9u2339pXX31lK1euLGHNouoIASEgBISAEBACQiD1CEQmgqnPP2wOW7ZssREjRtjo0aNtv/32s8MOO8wggcEIEMP+/fsbW8affvqpTZ8+XdLBIDj6FAJCQAgIASEgBIRADAhkJBFcs2aNvfPOO7Zhwwbr0aOHNW/e3Jxzf6kOxLBr167Wvn17mzhxoo0cOdIT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>
