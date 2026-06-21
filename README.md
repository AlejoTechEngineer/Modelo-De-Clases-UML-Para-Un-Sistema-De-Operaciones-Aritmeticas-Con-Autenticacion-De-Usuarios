<div align="center">

# 🏪 PoliMarket — Sistema de Gestión Empresarial

> **Laboratorio No. 2 — Reutilización de Software**  
> Maestría en Arquitectura de Software · Temas Avanzados de Diseño Software  
> Politécnico Grancolombiano · 2026

</div>

---

---

## 📋 Tabla de Contenido

- [Descripción General](#-descripción-general)
- [Arquitectura del Sistema](#-arquitectura-del-sistema)
- [Modelo de Clases UML](#-modelo-de-clases-uml)
- [Modelo de Componentes](#-modelo-de-componentes)
- [Requisitos Funcionales](#-requisitos-funcionales)
- [Stack Tecnológico](#-stack-tecnológico)
- [Estructura del Proyecto](#-estructura-del-proyecto)
- [Instalación y Ejecución](#-instalación-y-ejecución)
- [Clientes Implementados](#-clientes-implementados)
- [Endpoints API REST](#-endpoints-api-rest)
- [Principios de Diseño Aplicados](#-principios-de-diseño-aplicados)
- [Decisiones Arquitectónicas](#-decisiones-arquitectónicas)
- [Autor](#-autor)

---

## 📌 Descripción General

**PoliMarket** es un sistema de información empresarial diseñado bajo una **arquitectura modular basada en componentes**, que integra las principales áreas de negocio de una organización comercial: Recursos Humanos, Ventas, Bodega, Proveedores y Entregas.

El sistema fue concebido con base en los principios fundamentales del **diseño orientado a objetos** y la **reutilización de software**, modelado en **UML 2.5** e implementado en **Node.js**. Su arquitectura permite que la lógica de negocio sea consumida de forma independiente por múltiples tipos de clientes —una API REST y una interfaz CLI— sin duplicación de código.

### Objetivos del Sistema

| Objetivo | Descripción |
|---|---|
| 🔐 Control de acceso | Autorizar vendedores mediante el área de RRHH antes de operar |
| 💰 Gestión de ventas | Registrar transacciones validando inventario y autorizaciones |
| 📦 Control de inventario | Verificar stock y gestionar movimientos de bodega |
| 🚚 Logística | Gestionar entregas y actualizar estado de pedidos |
| 🔄 Reabastecimiento | Solicitar pedidos a proveedores cuando el stock es bajo |

---

## 🏛️ Arquitectura del Sistema

El sistema implementa una **arquitectura de capas con enfoque basado en componentes (CBD - Component-Based Design)**:

```
┌─────────────────────────────────────────────────────────┐
│                     CAPA DE CLIENTES                    │
│         Cliente Web (API REST)   │   Cliente CLI        │
└────────────────────┬────────────────────────────────────┘
                     │ consume
┌────────────────────▼────────────────────────────────────┐
│                  CAPA DE COMPONENTES                    │
│  ComponenteAutorizacion    │  ComponenteRegistroVentas  │
│  ComponenteInventario      │  ComponenteReabastecimiento│
│                ComponenteGestionEntregas                │
└────────────────────┬────────────────────────────────────┘
                     │ accede
┌────────────────────▼────────────────────────────────────┐
│                CAPA DE DATOS (db.js)                    │
│     Vendedores · Clientes · Productos · Ventas ·        │
│                Proveedores · Entregas                   │
└─────────────────────────────────────────────────────────┘
```

### Flujo General del Sistema

```
Cliente (web | CLI)
      │
      ▼
 Solicitud HTTP / Entrada CLI
      │
      ▼
 Componente de Negocio correspondiente
      │
      ├──► Valida autorización (ComponenteAutorizacion)
      ├──► Verifica stock (ComponenteInventario)
      └──► Persiste en db.js
      │
      ▼
 Respuesta al Cliente
```

---

## 📐 Modelo de Clases UML

El sistema modela **8 clases del dominio** con sus atributos, métodos y relaciones:

### Clases Identificadas

| Clase | Responsabilidad | Atributos clave |
|---|---|---|
| `RecursosHumanos` | Gestionar autorización de vendedores | `id`, `responsable` |
| `Vendedor` | Actor principal del proceso de ventas | `id`, `nombre`, `email`, `autorizado` |
| `Cliente` | Persona que realiza compras | `id`, `nombre`, `email`, `direccion` |
| `Producto` | Artículos disponibles para venta | `id`, `nombre`, `precio`, `stock` |
| `Venta` | Transacción comercial del sistema | `id`, `fecha`, `total`, `estado` |
| `Bodega` | Control de inventario físico | `id`, `ubicacion`, `capacidad` |
| `Proveedor` | Empresa externa de suministro | `id`, `nombre`, `contacto`, `telefono` |
| `Entrega` | Distribución de productos al cliente | `id`, `fecha`, `estado`, `direccionDestino` |

### Relaciones del Modelo

```
RecursosHumanos  1 ──────────────► 0..* Vendedor       [autoriza]
Vendedor         1 ──────────────► 0..* Venta           [realiza]
Venta            0..* ───────────► 1    Cliente         [pertenece a]
Venta            1..* ───────────► 1..* Producto        [incluye]
Bodega           1 ──────────────► 0..* Proveedor       [registra]
Venta            ··············►        Bodega          [«consulta»]
Entrega          ··············►        Venta           [«consulta»]
Entrega          ··············►        Bodega          [«registra salida»]
```

> **Convención:** `──────►` Asociación · `··············►` Dependencia

---

## 🧩 Modelo de Componentes

El sistema define **10 componentes** distribuidos por área de negocio, siguiendo el principio de **alta cohesión y bajo acoplamiento**:

### Tabla de Componentes

| Área | Componente | Funcionalidades Expuestas |
|---|---|---|
| Recursos Humanos | `ComponenteAutorizacion` | `autorizarVendedor()`, `revocarAutorizacion()` |
| Recursos Humanos | `ComponenteGestionVendedores` | `listarVendedores()`, `consultarEstadoVendedor()` |
| Ventas | `ComponenteRegistroVentas` | `registrarVenta()`, `calcularTotal()`, `getDetalle()` |
| Ventas | `ComponenteConsultaVentas` | `listarClientes()`, `listarProductosDisponibles()` |
| Bodega | `ComponenteInventario` | `verificarDisponibilidad()`, `registrarEntrada()`, `registrarSalida()` |
| Bodega | `ComponenteReabastecimiento` | `solicitarReabastecimiento()`, `consultarProveedores()` |
| Proveedores | `ComponenteGestionProveedores` | `registrarProducto()`, `getProductosSuministrados()` |
| Proveedores | `ComponentePedidos` | `solicitarPedido()`, `consultarEstadoPedido()` |
| Entregas | `ComponenteGestionEntregas` | `registrarEntrega()`, `confirmarEntrega()` |
| Entregas | `ComponenteLogistica` | `consultarPedidosPendientes()`, `registrarSalidaBodega()` |

### Grafo de Dependencias entre Componentes

```
[ComponenteAutorizacion] ──────────► [ComponenteGestionVendedores]
         ▲
         │
[ComponenteRegistroVentas] ─────────► [ComponenteConsultaVentas]
         │                  └────────► [ComponenteInventario]
         │                                      │
         │                                      ▼
         │                         [ComponenteReabastecimiento]
         │                                      │
         │                           ┌──────────┘
         │                           ▼
         │                    [ComponentePedidos]
         │                    [ComponenteGestionProveedores]
         │
[ComponenteGestionEntregas] ─────────► [ComponenteLogistica]
         │                                      │
         └──────────────────┬───────────────────┘
                            ▼
               [ComponenteInventario]
               [ComponenteConsultaVentas]
```

---

## ✅ Requisitos Funcionales

| ID | Requisito | Componente Principal | Componentes de Apoyo |
|---|---|---|---|
| RF1 | Autorizar vendedor | `ComponenteAutorizacion` | — |
| RF2 | Registrar venta | `ComponenteRegistroVentas` | `ComponenteAutorizacion`, `ComponenteInventario` |
| RF3 | Verificar disponibilidad de productos | `ComponenteInventario` | — |
| RF4 | Solicitar reabastecimiento | `ComponenteReabastecimiento` | `ComponenteInventario` |
| RF5 | Registrar entrega | `ComponenteGestionEntregas` | `ComponenteInventario` |

---

## 🛠️ Stack Tecnológico

| Tecnología | Versión | Uso |
|---|---|---|
| **Node.js** | 18+ | Runtime del servidor |
| **Express.js** | 4.x | Framework API REST (Cliente Web) |
| **PlantUML** | 2.5 | Generación de diagramas UML |
| **JavaScript (ES6+)** | — | Lenguaje de implementación |
| HTML5 | — | Interfaz web estática (`public/index.html`) |

> El sistema **no requiere base de datos externa**. La persistencia en esta versión se gestiona en memoria a través del módulo `db.js`, diseñado para ser reemplazable por una base de datos real (PostgreSQL, MongoDB, etc.) sin modificar los componentes de negocio.

---

## 📁 Estructura del Proyecto

```
LABORATORIO NO. 2/
└── polimarket/
    ├── components/                          # Capa de lógica de negocio
    │   ├── ComponenteAutorizacion.js        # RF1 - Autorización vendedores (RRHH)
    │   ├── ComponenteRegistroVentas.js      # RF2 - Registro de ventas
    │   ├── ComponenteInventario.js          # RF3 - Control de stock (Bodega)
    │   ├── ComponenteReabastecimiento.js    # RF4 - Reabastecimiento (Bodega/Proveedores)
    │   └── ComponenteGestionEntregas.js     # RF5 - Gestión de entregas
    │
    ├── clients/                             # Capa de acceso / clientes
    │   ├── web/
    │   │   └── servidor.js                 # Cliente 1: API REST con Express (puerto 3000)
    │   └── cli/
    │       └── cliente-cli.js              # Cliente 2: CLI interactivo por terminal
    │
    ├── public/
    │   └── index.html                      # Dashboard web del sistema
    │
    ├── uml/                                # Artefactos UML generados
    │   ├── diagrama-clases.puml            # Código PlantUML - Diagrama de clases
    │   ├── diagrama-clases.bmp             # Imagen exportada - Diagrama de clases
    │   ├── diagrama-componentes.puml       # Código PlantUML - Diagrama de componentes
    │   └── diagrama-componentes.BMP        # Imagen exportada - Diagrama de componentes
    │
    ├── db.js                               # Base de datos en memoria
    ├── package.json                        # Dependencias y scripts NPM
    └── package-lock.json                   # Lock de versiones
```

---

## 🚀 Instalación y Ejecución

### Prerrequisitos

- **Node.js** v18 o superior → [Descargar](https://nodejs.org/)
- **npm** v9 o superior (incluido con Node.js)

### Pasos de Instalación

```bash
# 1. Descomprimir el código fuente
# Extraer polimarket.zip en el directorio deseado

# 2. Acceder al directorio del proyecto
cd polimarket

# 3. Instalar dependencias
npm install
```

### Ejecución — Cliente Web (API REST)

```bash
npm run start:web
```

Una vez iniciado, acceder desde el navegador:

```
http://localhost:3000
```

El dashboard mostrará el estado del sistema: contadores de vendedores, productos, ventas y entregas pendientes, junto con el inventario rápido.

### Ejecución — Cliente CLI

```bash
npm run start:cli
```

Se desplegará un menú interactivo en la terminal:

```
============================================================
           BIENVENIDO AL SISTEMA POLIMARKET
============================================================

Selecciona una opción:
  1. Autorizar vendedor         (RF1 - RRHH)
  2. Registrar venta            (RF2 - Ventas)
  3. Ver disponibilidad bodega  (RF3 - Bodega)
  4. Solicitar reabastecimiento (RF4 - Bodega)
  5. Registrar entrega          (RF5 - Entregas)
  0. Salir
```

---

## 🔌 Endpoints API REST

Base URL: `http://localhost:3000`

| Método | Endpoint | RF | Descripción |
|---|---|---|---|
| `POST` | `/rrhh/autorizar` | RF1 | Autorizar un vendedor |
| `POST` | `/ventas/registrar` | RF2 | Registrar una venta |
| `GET` | `/bodega/disponibilidad/:id` | RF3 | Consultar stock de producto |
| `POST` | `/bodega/reabastecer` | RF4 | Solicitar reabastecimiento |
| `POST` | `/entregas/registrar` | RF5 | Registrar una entrega |

### Ejemplos de uso con cURL

```bash
# RF1 - Autorizar vendedor
curl -X POST http://localhost:3000/rrhh/autorizar \
  -H "Content-Type: application/json" \
  -d '{"vendedorId": "V001"}'

# RF2 - Registrar venta
curl -X POST http://localhost:3000/ventas/registrar \
  -H "Content-Type: application/json" \
  -d '{"vendedorId": "V001", "clienteId": "C001", "productos": [{"id": "P001", "cantidad": 2}]}'

# RF3 - Verificar disponibilidad
curl http://localhost:3000/bodega/disponibilidad/P001

# RF4 - Solicitar reabastecimiento
curl -X POST http://localhost:3000/bodega/reabastecer \
  -H "Content-Type: application/json" \
  -d '{"productoId": "P001", "cantidad": 50}'

# RF5 - Registrar entrega
curl -X POST http://localhost:3000/entregas/registrar \
  -H "Content-Type: application/json" \
  -d '{"ventaId": "VT001", "direccion": "Calle 123 # 45-67, Bogotá"}'
```

---

## 🎯 Clientes Implementados

### Cliente 1 — API REST con Express

| Atributo | Detalle |
|---|---|
| Plataforma | Web / HTTP |
| Puerto | 3000 |
| Framework | Express.js |
| Interfaz visual | Dashboard en `public/index.html` |
| RF consumidos | RF1, RF2, RF3, RF4, RF5 |

### Cliente 2 — CLI Interactivo

| Atributo | Detalle |
|---|---|
| Plataforma | Terminal / Consola |
| Protocolo | Entrada estándar (stdin) |
| Framework | Node.js nativo |
| RF consumidos | RF1, RF2, RF3, RF4, RF5 |

> Ambos clientes consumen **exactamente los mismos componentes** sin modificación alguna a la lógica de negocio, evidenciando el principio de **reutilización de software** como eje central de este laboratorio.

---

## 🧠 Principios de Diseño Aplicados

### 1. Separación de Responsabilidades (SoC)
Cada componente encapsula una única área funcional. Los clientes no conocen la lógica interna; solo invocan la interfaz pública de los componentes.

### 2. Bajo Acoplamiento
Los clientes (web, CLI) dependen de los componentes a través de sus interfaces expuestas, no de su implementación. Esto permite reemplazar o extender un componente sin afectar a los demás.

### 3. Alta Cohesión
Cada componente agrupa funcionalidades estrechamente relacionadas. Por ejemplo, `ComponenteInventario` concentra exclusivamente la lógica de stock: verificación, entrada y salida.

### 4. Reutilización de Software (CBD)
El mismo componente es consumido por el cliente web y el cliente CLI. Esta reutilización elimina duplicación de código y garantiza consistencia del comportamiento del sistema en cualquier plataforma.

### 5. Open/Closed Principle (OCP)
La arquitectura permite **agregar nuevos clientes** (móvil, desktop, microservicio) o **nuevos componentes** sin modificar los existentes.

### 6. Inversión de Dependencias
Los clientes dependen de abstracciones (funciones exportadas de los componentes), no de implementaciones concretas de base de datos o frameworks.

---

## 📊 Decisiones Arquitectónicas

### ¿Por qué arquitectura basada en componentes y no microservicios?

Para el alcance académico de este laboratorio, el enfoque CBD permite demostrar los principios de reutilización, separación de responsabilidades y desacoplamiento de forma clara, sin la complejidad operacional de un entorno distribuido (redes, tolerancia a fallos, service discovery).

La estructura, sin embargo, está diseñada para **escalar hacia microservicios**: cada componente puede ser extraído como un servicio independiente con mínima refactorización.

### ¿Por qué db.js en memoria en lugar de una base de datos real?

La capa de persistencia fue deliberadamente desacoplada de la lógica de negocio. `db.js` actúa como un **Data Access Object (DAO) simulado**, lo que permite:
- Ejecutar el sistema sin dependencias externas
- Sustituir la implementación por PostgreSQL, MongoDB u otro motor sin tocar los componentes
- Enfocarse en el diseño arquitectónico, no en la administración de infraestructura

### ¿Por qué dos clientes con stacks completamente diferentes?

Demuestra de forma práctica que la **lógica de negocio es independiente del mecanismo de entrega** (*delivery mechanism*). Este principio, central en arquitecturas limpias (Clean Architecture, Hexagonal), garantiza que el sistema no está atado a ninguna tecnología de frontend o protocolo de comunicación.

---

## 📄 Artefactos UML Generados

| Artefacto | Herramienta | Formato | Descripción |
|---|---|---|---|
| `diagrama-clases.puml` | PlantUML 2.5 | Código fuente | 8 clases con atributos, métodos y relaciones |
| `diagrama-clases.bmp` | PlantUML 2.5 | Imagen BMP | Diagrama de clases exportado |
| `diagrama-componentes.puml` | PlantUML 2.5 | Código fuente | 10 componentes distribuidos por área |
| `diagrama-componentes.BMP` | PlantUML 2.5 | Imagen BMP | Diagrama de componentes exportado |

---

## 📚 Referencias

- Booch, G., Rumbaugh, J., & Jacobson, I. (2005). *El lenguaje unificado de modelado (UML): Guía del usuario* (2ª ed.). Pearson Educación.
- Sommerville, I. (2016). *Ingeniería de software* (10ª ed.). Pearson Educación.
- Pressman, R. S., & Maxim, B. R. (2020). *Ingeniería de software: Un enfoque práctico* (9ª ed.). McGraw-Hill.
- Szyperski, C. (2002). *Component Software: Beyond Object-Oriented Programming* (2nd ed.). Addison-Wesley.
- Object Management Group. (2017). *Unified Modeling Language Specification Version 2.5*. https://www.omg.org/spec/UML/
- PlantUML. (2024). *PlantUML Language Reference Guide*. https://plantuml.com/
- Node.js Foundation. (2024). *Node.js Documentation*. https://nodejs.org/
- Express.js. (2024). *Express - Node.js web application framework*. https://expressjs.com/

---

## 👤 Autor

**Alejandro De Mendoza**  
Ingeniero Informático — UNIR · Ingeniero Senior de Plataformas e Integraciones · Especialista en Inteligencia Artificial — UNIR · Maestría en Arquitectura de Software — Politécnico Grancolombiano  

---

*Desarrollado para la asignatura Temas Avanzados de Diseño Software · Ing. Lina María Montoya Suárez · Bogotá D.C., 2026*

---

## Autor

**Alejandro De Mendoza**  
Ingeniero Informático · Especialista en IA · Especialista en Ingeniería de Software · Máster en Arquitectura de Software

[![GitHub](https://img.shields.io/badge/GitHub-AlejoTechEngineer-181717?style=for-the-badge&logo=github)](https://github.com/AlejoTechEngineer)
