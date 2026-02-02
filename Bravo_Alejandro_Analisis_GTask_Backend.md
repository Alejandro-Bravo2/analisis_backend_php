# Análisis de un backend PHP sin framework (Proyecto GTask)

**Alumno:** Alejandro Bravo Calderón
**Módulo:** Desarrollo Web en Entorno Servidor (DWES)
**Fecha:** Febrero 2026

---

## Parte A - Mapa de arquitectura

### Flujo de una petición HTTP

Todo sigue este recorrido:

```
Cliente (navegador / Postman)
       |
       v
Nginx (puerto 8080)
       |
       v  redirige todo a index.php
       |
index.php
       |
       ├── Inicia sesión PHP
       ├── Carga las dependencias
       ├── Aplica CORS
       ├── Lee la configuración y conecta con la BD
       ├── Crea los controladores
       ├── Lee el método HTTP y la ruta
       └── Decide qué controlador ejecutar
              |
              v
     Controller (Auth o Task)
              |
              ├── Valida los datos
              ├── Hace la consulta a PostgreSQL
              └── Devuelve JSON
```

### Ficheros y qué hace cada uno

| Fichero | Qué hace |
|---------|----------|
| `nginx.conf` | Recibe las peticiones y redirige a `index.php` con `try_files` |
| `app/public/index.php` | Es el punto de entrada de todo. Carga dependencias, aplica CORS, parsea la ruta y llama al controlador que toque |
| `app/src/Support.php` | Funciones de ayuda: respuestas JSON, leer el body, CORS y comprobar si estás logueado |
| `app/config/config.php` | Configuración de la BD con variables de entorno |
| `app/src/Database.php` | Crea la conexión PDO a PostgreSQL |
| `app/src/Controllers/AuthController.php` | Registro, login, logout y ver usuario actual |
| `app/src/Controllers/TaskController.php` | CRUD de tareas |

### Sobre el patrón que usa

Creo que esto es lo que se llama **Front Controller**: todas las peticiones pasan por un solo archivo (`index.php`) y desde ahí se decide qué hacer. Es parecido a lo que hace Laravel o Spring Boot con el DispatcherServlet, pero aquí está hecho a mano con bloques `if`.

La ventaja es que la inicialización (sesión, CORS, conexión a BD) se hace en un sitio y no hay que repetirla.

---

## Parte B - Conexión a base de datos

### Dónde se crea

La conexión se crea en el constructor de la clase `Database` en `app/src/Database.php`. Luego en `index.php` se instancia así:

```php
$database = new Database($config['db']);
```

### Cómo se monta el DSN

Usa `sprintf` para construir la cadena de conexión:

```php
$dsn = sprintf(
    'pgsql:host=%s;port=%s;dbname=%s',
    $config['host'],
    $config['port'],
    $config['name']
);
```

Eso genera algo como `pgsql:host=db;port=5432;dbname=app`.

### De dónde salen los valores

De `app/config/config.php`, que lee variables de entorno con `getenv()`. Si no existen, tiene valores por defecto:

- `DB_HOST` → `db` (nombre del contenedor de PostgreSQL en Docker)
- `DB_PORT` → `5432`
- `DB_NAME` → `app`
- `DB_USER` → `user`
- `DB_PASSWORD` → `password`

Estas variables se configuran en el `docker-compose.yml`, dentro del servicio `php`.

### Qué pasa si falla la conexión

En el constructor de `Database.php` se pone `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`, así que si falla la conexión PHP lanza una excepción. No hay ningún try-catch, así que supongo que el servidor devuelve un error 500 directamente.

También pone `PDO::FETCH_ASSOC` para que los resultados vengan como arrays asociativos.

---

## Parte C - Enrutado y gestión de peticiones

### Cómo distingue el método HTTP

En `index.php` lee el método con:

```php
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
```

Básicamente usa `$_SERVER['REQUEST_METHOD']` que tiene el verbo HTTP (GET, POST, PUT, etc).

### Cómo procesa la ruta

Lo que hace es:

1. Saca la ruta con `parse_url()` para quedarse solo con el path (sin query strings)
2. Le quita la barra del final con `rtrim`
3. Parte la ruta en trozos con `explode('/', ...)`

O sea que `/api/tasks/5` se convierte en `['api', 'tasks', '5']`. A partir de ahí sabe que el recurso es `tasks` y el id es `5`.

### Cómo decide qué controlador usar

Es un bloque de `if` manuales. Primero mira que la ruta empiece por `api`, si no devuelve 404. Luego compara el recurso y el método:

- Si es `register`, `login`, `logout` o `me` → va a `AuthController`
- Si es `tasks` → primero llama a `require_auth()` para comprobar que estás logueado, y luego va a `TaskController`

Si nada coincide, devuelve 404.

### Tabla de endpoints

| Endpoint | Método | Auth | Controller::método |
|----------|--------|------|--------------------|
| `/` | GET | No | Sirve la página web (`views/app.php`) |
| `/api/register` | POST | No | `AuthController::register` |
| `/api/login` | POST | No | `AuthController::login` |
| `/api/logout` | POST | No | `AuthController::logout` |
| `/api/me` | GET | Sí | `AuthController::me` |
| `/api/tasks` | GET | Sí | `TaskController::index` |
| `/api/tasks` | POST | Sí | `TaskController::create` |
| `/api/tasks/{id}` | GET | Sí | `TaskController::show` |
| `/api/tasks/{id}` | PUT/PATCH | Sí | `TaskController::update` |
| `/api/tasks/{id}` | DELETE | Sí | `TaskController::delete` |

En `/api/me` la autenticación no se comprueba en `index.php` como las otras, sino dentro del propio método `me()`.

---

## Parte D - Validación y control de tipos

### Registro (AuthController::register)

| Campo | Validación | HTTP | Mensaje |
|-------|-----------|------|---------|
| `name` | Obligatorio, máx 100 caracteres | 422 | "Nombre, email y contraseña son obligatorios." / "El nombre no puede superar 100 caracteres." |
| `email` | Obligatorio, formato válido con `filter_var`, único en BD | 422 / 409 | "El email no es valido." / "El email ya está registrado." |
| `password` | Obligatorio, mín 6 caracteres | 422 | "La contraseña debe tener al menos 6 caracteres." |

### Login (AuthController::login)

En el login se valida que el email y la contraseña no estén vacíos y que el email tenga formato válido. Para comprobar la contraseña se usa `password_verify` contra el hash que hay en la BD. Si no coincide devuelve 401.

### Creación de tareas (TaskController::create)

| Campo | Validación | HTTP | Mensaje |
|-------|-----------|------|---------|
| `title` | Obligatorio, máx 200 caracteres | 422 | "El título es obligatorio." |
| `description` | Opcional, máx 1000 caracteres | 422 | "La descripción no puede superar 1000 caracteres." |
| `status` | Opcional (por defecto `pending`), solo `pending` o `completed` | 422 | "El estado no es valido." |
| `priority` | Opcional (por defecto 0), entre 0 y 5 | 422 | "La prioridad debe estar entre 0 y 5." |
| `due_date` | Opcional, formato `YYYY-MM-DD` | 422 | "La fecha limite debe tener formato YYYY-MM-DD." |

### Actualización de tareas (TaskController::update)

Las mismas validaciones que al crear, pero solo se validan los campos que envías. Si no mandas ningún campo, devuelve 422.

### Ejemplos de datos inválidos

Si intentas registrarte sin contraseña:
```json
// POST /api/register
{"name": "Alejandro", "email": "alex@test.com"}

// Respuesta 422:
{"error": "Nombre, email y contraseña son obligatorios."}
```

Si creas una tarea con prioridad 8 (fuera de rango):
```json
// POST /api/tasks
{"title": "Mi tarea", "priority": 8}

// Respuesta 422:
{"error": "La prioridad debe estar entre 0 y 5."}
```

### Códigos HTTP que usa

Los que he visto son: 200 para respuestas normales, 201 cuando se crea algo (registro o tarea nueva), 401 si no estás logueado, 404 si la ruta o tarea no existe, 409 cuando el email ya está cogido, y 422 para errores de validación. También usa 204 para las peticiones OPTIONS de CORS y 400 si el JSON viene mal.

---

## Parte E - Autenticación y control de acceso

### Cómo funciona la sesión

El sistema usa **sesiones de PHP** para saber quién está logueado. El flujo es:

1. En `index.php` se llama a `session_start()`, que crea o recupera la sesión. Esto le manda una cookie `PHPSESSID` al navegador
2. Cuando haces login o registro, se guardan tus datos en `$_SESSION['user']`
3. En las siguientes peticiones, PHP lee la cookie y recupera la sesión automáticamente

### Qué se guarda en $_SESSION

Un array con tres cosas:

```php
$_SESSION['user'] = [
    'id'    => $userId,
    'name'  => $name,
    'email' => $email,
];
```

La contraseña no se guarda en la sesión, solo el id, nombre y email.

### Cómo se protegen los endpoints

Hay una función `require_auth()` en `Support.php` que comprueba esto:

```php
function require_auth(): array
{
    if (empty($_SESSION['user'])) {
        json_error('No autenticado.', 401);
    }
    return $_SESSION['user'];
}
```

Si no hay sesión, devuelve 401 y corta la ejecución. Si hay sesión, te devuelve los datos del usuario. Esta función se usa en `index.php` antes de las rutas de tareas, y tambien dentro de `AuthController::me()`.

### Cómo se evita acceder a tareas de otro usuario

En `TaskController.php` todas las consultas SQL llevan `WHERE user_id = :user_id`, así que solo puedes ver, modificar o borrar tus propias tareas. El `$userId` no viene de lo que manda el cliente en el JSON, sino que se saca de la sesión en `index.php` y se le pasa al controlador. Así que no puedes inventarte un id para acceder a las tareas de otro.

Si intentas acceder a una tarea que no es tuya o que no existe, en ambos casos te devuelve 404. No te dice si la tarea existe para otro usuario, simplemente que no la encuentra.

---

## Parte F - CORS, respuestas y errores

### CORS

CORS se gestiona en `apply_cors()` de `Support.php`. Basicamente lee las variables de entorno `CORS_ENABLED` y `ALLOWED_ORIGINS` para saber si tiene que poner las cabeceras CORS o no. Si el origen de la petición está en la lista permitida, pone `Access-Control-Allow-Origin` con ese origen.

Para las peticiones `OPTIONS` (que creo que son las que hace el navegador antes de la petición real) devuelve 204 y corta.

### Respuestas JSON

Hay dos funciones en `Support.php` para las respuestas:

- `json_response()` pone el código HTTP, la cabecera JSON y hace `json_encode`. Luego hace `exit` para cortar.
- `json_error()` llama a `json_response` pero mete la clave `error`. Así los errores siempre tienen el formato `{"error": "mensaje"}`.

Un ejemplo de login correcto:
```json
{
    "message": "Login correcto.",
    "user": {
        "id": 1,
        "name": "Alejandro",
        "email": "alex@test.com"
    }
}
```

Y un error sería algo así:
```json
{"error": "El título es obligatorio."}
```

Las tareas vienen con todos los campos de la BD:
```json
{
    "tasks": [
        {
            "id": 1,
            "title": "Mi tarea",
            "description": null,
            "status": "pending",
            "due_date": null,
            "priority": 0,
            "created_at": "2026-02-01 10:30:00",
            "updated_at": "2026-02-01 10:30:00",
            "completed_at": null
        }
    ]
}
```

Las respuestas de éxito no siempre tienen la misma estructura (unas llevan `message`, otras `user`, otras `tasks`), pero los errores sí son siempre `{"error": "mensaje"}`.

---

## Parte G - Propuestas de mejora

### Mejora 1: Usar tokens en vez de sesiones

Las sesiones se guardan en el servidor, y si hubiera mas de un servidor no se compartirían entre ellos. Se podría usar **JWT** en vez de sesiones, que es lo que vimos en clase. Basicamente el servidor te da un token firmado cuando haces login y tú lo mandas en la cabecera `Authorization` en cada petición. Habría que cambiar `require_auth()` para que lea el token en vez de `$_SESSION`, y que el login devuelva el token en vez de crear sesión.

### Mejora 2: Separar la lógica en capas

Los controladores hacen todo: validan, hacen la lógica y ejecutan las consultas SQL. En Spring Boot esto se separa con `@Service` y `@Repository`, cada clase hace una cosa. Se podría hacer algo parecido aquí, por ejemplo meter las consultas SQL en una clase `TaskRepository` y la lógica en un `TaskService`. Así los controladores quedan mas limpios.

