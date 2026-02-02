# 📝 Actividad evaluable  
## Análisis de un backend PHP sin framework (Proyecto GTask)

### Módulo
Desarrollo Web en Entorno Servidor (DWES)  
Programación Backend

### Tipo de actividad
Actividad **individual**, **evaluable**, de **análisis y razonamiento técnico**  
(no programación)

---

## 📖 Descripción de la actividad

En esta actividad analizarás un **backend funcional desarrollado en PHP sin framework**
(proyecto **GTask**) con el objetivo de **comprender cómo funciona realmente un backend web** cuando no existe un framework que oculte la complejidad.

El análisis se centrará en:

- conexión a base de datos
- enrutado y gestión de peticiones HTTP
- validación y control de tipos
- autenticación mediante sesiones
- control de acceso a recursos (autorización)
- manejo uniforme de respuestas y errores
- configuración CORS

El objetivo **no es programar**, sino **leer, entender y razonar** sobre el código existente,
para entender cómo realmente trabajan frameworks como **Laravel, Spring Boot o .NET Web API**.

---

## 🎯 Resultados de Aprendizaje (RA)

Esta actividad contribuye a los siguientes resultados de aprendizaje del módulo:

### RA principal
**RA:**  
Desarrolla aplicaciones web en entorno servidor analizando y aplicando principios de arquitectura, acceso a datos, validación, autenticación y control de la lógica de negocio.

### RA secundarios
- Analiza el flujo de datos entre cliente, servidor y base de datos.
- Identifica mecanismos de validación y control de errores en aplicaciones backend.
- Reconoce técnicas de autenticación y autorización en aplicaciones web.
- Valora la importancia de la separación de responsabilidades en el código servidor.

---

## 📏 Criterios de Evaluación (CE)

La actividad evalúa parcialmente los siguientes criterios:

- **CE 1:** Analiza el flujo de ejecución de aplicaciones web en entorno servidor.  
- **CE 2:** Identifica y justifica mecanismos de validación y control de errores.  
- **CE 3:** Reconoce técnicas de acceso a datos y sus implicaciones en seguridad y mantenimiento.  
- **CE 4:** Analiza mecanismos de autenticación y control de acceso a recursos.  

> ⚠️ Esta actividad no cubre la totalidad de ningún RA, sino una parte de ellos.

---

## 📂 Material proporcionado

Se facilitará al alumnado el proyecto:

```

gtask-php-app-2025_26

```

Ficheros clave a analizar:

- `app/public/index.php`
- `app/src/Support.php`
- `app/src/Database.php`
- `app/config/config.php`
- `app/src/Controllers/AuthController.php`
- `app/src/Controllers/TaskController.php`

---

## 🧩 Tareas a realizar

### Parte A — Mapa de arquitectura (obligatoria)

Describe el flujo general de una petición HTTP desde que llega al servidor hasta que se devuelve una respuesta JSON.

Ejemplo de flujo:

```

Request HTTP → Router → Controller → Base de Datos → Response JSON

```

Incluye:
- fichero donde se enrutan las peticiones
- controlador responsable
- acceso a base de datos
- formato de respuesta

---

### Parte B — Conexión a base de datos (obligatoria)

Indica claramente:

- dónde se crea la conexión a la base de datos
- cómo se construye el DSN
- cómo se obtienen los valores de configuración
- qué ocurre si la conexión falla

---

### Parte C — Enrutado y gestión de peticiones (obligatoria)

Explica:

- cómo se distingue el método HTTP (`GET`, `POST`, `PUT`, `DELETE`)
- cómo se procesa la ruta
- cómo se decide qué controlador y método ejecutar

Incluye una **tabla de endpoints**, al menos:

| Endpoint | Método | Auth | Controller::método |
|--------|--------|------|--------------------|
| `/api/register` | POST | No | AuthController::register |
| `/api/login` | POST | No | AuthController::login |
| `/api/me` | GET | Sí | AuthController::me |
| `/api/tasks` | GET | Sí | TaskController::index |
| `/api/tasks` | POST | Sí | TaskController::create |
| `/api/tasks/{id}` | PUT | Sí | TaskController::update |
| `/api/tasks/{id}` | DELETE | Sí | TaskController::delete |

---

### Parte D — Validación y control de tipos (obligatoria)

Analiza las validaciones existentes:

- campos obligatorios
- longitud de cadenas
- valores permitidos (`status`, `priority`, etc.)
- formatos de fecha
- códigos de error HTTP utilizados

Incluye **ejemplos de datos inválidos** y la respuesta esperada del servidor.

---

### Parte E — Autenticación y control de acceso (obligatoria)

Explica:

- cómo se gestiona la sesión del usuario
- qué información se guarda en `$_SESSION`
- cómo se bloquea el acceso a endpoints protegidos
- cómo se evita que un usuario acceda a recursos de otro

Indica en qué ficheros se realiza cada comprobación.

---

### Parte F — CORS, respuestas y errores (obligatoria)

Analiza:

- cómo se gestionan las cabeceras CORS
- cómo se responde a peticiones `OPTIONS`
- formato uniforme de respuestas JSON
- estructura de errores

---

### Parte G — Propuesta de mejoras (obligatoria)

Propón **dos mejoras realistas**, justificadas técnicamente.

Ejemplos:
- separar reglas de negocio en servicios
- mejorar gestión de errores
- introducir tokens en lugar de sesiones
- paginación y filtros
- rate limiting

---

## 📤 Entrega

- **Formato:** PDF / Markdown / DOCX
- **Nombre del archivo:**

```

Apellido_Nombre_Analisis_GTask_Backend.pdf

```

- **Entrega:** a través de Moodle

---

## ✅ Checklist de autoevaluación

- [ ] Identifico la conexión a BD y su configuración
- [ ] Explico el enrutado de peticiones
- [ ] Incluyo tabla de endpoints
- [ ] Analizo validaciones y tipos
- [ ] Explico autenticación y control de acceso
- [ ] Analizo CORS y manejo de errores
- [ ] Propongo 2 mejoras razonadas
- [ ] Referencio ficheros concretos del proyecto

---

## 📊 Rúbrica de evaluación (10 puntos)

### 1. Arquitectura y flujo (2 puntos)
- Excelente (2): flujo completo, claro y coherente
- Adecuado (1–1.5): correcto pero incompleto
- Insuficiente (0–0.5): confuso o superficial

### 2. Conexión a base de datos (1.5 puntos)
- Excelente (1.5): identifica config, PDO y errores
- Adecuado (0.75–1)
- Insuficiente (0–0.5)

### 3. Enrutado y endpoints (2 puntos)
- Excelente (2): explicación clara + tabla completa
- Adecuado (1–1.5)
- Insuficiente (0–0.5)

### 4. Validación y tipos (2 puntos)
- Excelente (2): validaciones bien analizadas y justificadas
- Adecuado (1–1.5)
- Insuficiente (0–0.5)

### 5. Autenticación y autorización (2 puntos)
- Excelente (2): distingue autenticación y control de acceso
- Adecuado (1–1.5)
- Insuficiente (0–0.5)

### 6. CORS, errores y mejoras (0.5 puntos)
- Completo (0.5)
- Parcial (0.25)
- Ausente (0)

---

## 🧠 Nota final para el alumnado

> No se trata de aprender a programar sin framework,  
> sino de entender **qué problemas resuelve un framework moderno**  
> y por qué es importante saber leer y mantener código existente.



---
