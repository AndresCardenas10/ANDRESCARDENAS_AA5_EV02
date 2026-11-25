# Servicio web de registro e inicio de sesión – Pruebas con Postman

**Evidencia:** GA7-220501096-AA5-EV02 – API (testing con Postman)
**Base:** código de la evidencia GA7-220501096-AA5-EV01 (sin cambios en la lógica)
**Proyecto formativo:** SGB – Sistema de Gestión para Barbería
**Aprendiz:** Andrés Cárdenas · **Ficha:** 3186668
**Programa:** Tecnología en Análisis y Desarrollo de Software (SENA)

## Descripción

Servicio web (API REST) que permite **registrar** un usuario y **iniciar sesión**.
Recibe un `usuario` y una `contrasena` en formato JSON:

- Si la autenticación es correcta responde **"Autenticación satisfactoria"**.
- Si no lo es responde **"Error en la autenticación"**.

La contraseña nunca se guarda en texto plano: se almacena su *hash* con bcrypt
(coherente con el requisito de seguridad del proyecto SGB).

## Tecnologías

| Tecnología | Uso |
|---|---|
| Node.js (≥ 20) | Entorno de ejecución de JavaScript |
| Express 5 | Framework para crear el servidor y las rutas |
| MySQL / MariaDB (XAMPP) | Base de datos |
| mysql2 | Conexión a la base de datos (consultas parametrizadas) |
| bcryptjs | Cifrado (hash) de contraseñas |
| dotenv | Lectura de la configuración desde `.env` |
| Git / GitHub | Control de versiones |

## Requisitos previos

- Node.js 20 o superior (`node -v`)
- XAMPP con el módulo **MySQL** iniciado (y **Apache** si se usa phpMyAdmin)
- Git

## Instalación y ejecución

1. **Crear la base de datos.** En phpMyAdmin (`http://localhost/phpmyadmin`) ir a la
   pestaña **Importar** y cargar el archivo `database/crear-bd.sql`.
2. **Configurar el entorno.** Copiar `.env.example` como `.env` (si no existe) y
   ajustar los valores si es necesario. Con XAMPP por defecto no hace falta cambiar nada.
3. **Instalar dependencias:**
   ```bash
   npm install
   ```
4. **Iniciar el servicio:**
   ```bash
   npm start
   ```
   Salida esperada:
   ```
   Conexion a la base de datos: OK
   Servidor escuchando en http://localhost:3000
   ```
   Para desarrollo (se reinicia solo al guardar cambios): `npm run dev`.

## Endpoints

URL base: `http://localhost:3000`

Todas las peticiones envían y reciben **JSON** (`Content-Type: application/json`).

### `POST /api/register` – Registrar usuario

Cuerpo de la petición:

```json
{
  "usuario": "andres",
  "contrasena": "Clave123"
}
```

| Situación | Código HTTP | Respuesta |
|---|---|---|
| Registro correcto | `201 Created` | `{"exito": true, "mensaje": "Usuario registrado correctamente"}` |
| El usuario ya existe | `409 Conflict` | `{"exito": false, "mensaje": "El usuario ya existe"}` |
| Faltan datos o no son texto | `400 Bad Request` | `{"exito": false, "mensaje": "Debe enviar usuario y contrasena"}` |
| Usuario > 50 caracteres o contraseña < 6 caracteres | `400 Bad Request` | `{"exito": false, "mensaje": "<motivo>"}` |
| Falla del servidor o de la base de datos | `500` | `{"exito": false, "mensaje": "Error interno del servidor"}` |

### `POST /api/login` – Iniciar sesión

Cuerpo de la petición:

```json
{
  "usuario": "andres",
  "contrasena": "Clave123"
}
```

| Situación | Código HTTP | Respuesta |
|---|---|---|
| Usuario y contraseña correctos | `200 OK` | `{"exito": true, "mensaje": "Autenticación satisfactoria"}` |
| Usuario inexistente **o** contraseña incorrecta | `401 Unauthorized` | `{"exito": false, "mensaje": "Error en la autenticación"}` |
| Faltan datos | `400 Bad Request` | `{"exito": false, "mensaje": "Error en la autenticación: debe enviar usuario y contrasena"}` |
| Falla del servidor o de la base de datos | `500` | `{"exito": false, "mensaje": "Error interno del servidor"}` |

> Por seguridad, cuando el usuario no existe y cuando la contraseña es incorrecta se
> devuelve **el mismo mensaje**, para no revelar qué usuarios están registrados.

### `GET /` – Comprobación

Devuelve un JSON indicando que el servicio está activo. Sirve para verificarlo desde el navegador.

## Pruebas con Postman (EV02)

La colección `pruebas/AA5_EV02.postman_collection.json` contiene **10 pruebas** con
verificaciones automáticas (código de estado, campo `exito` y mensaje):

| # | Método y ruta | Caso | Esperado |
|---|---|---|---|
| 01 | `GET /` | Servicio activo | 200 |
| 02 | `POST /api/register` | Registro correcto | 201 |
| 03 | `POST /api/register` | Usuario repetido | 409 |
| 04 | `POST /api/register` | Contraseña corta | 400 |
| 05 | `POST /api/register` | Falta la contraseña | 400 |
| 06 | `POST /api/login` | Login correcto | 200 – "Autenticación satisfactoria" |
| 07 | `POST /api/login` | Contraseña incorrecta | 401 – "Error en la autenticación" |
| 08 | `POST /api/login` | Usuario inexistente | 401 – "Error en la autenticación" |
| 09 | `POST /api/login` | JSON mal escrito | 400 |
| 10 | `GET /api/no-existe` | Ruta inexistente | 404 |

Cómo ejecutarlas:

1. Iniciar MySQL y el servicio (`npm start`).
2. Vaciar la tabla antes de empezar (en phpMyAdmin, pestaña SQL):
   `TRUNCATE TABLE aa5_ev01_auth.usuarios;`
3. En Postman: **Import** → seleccionar la colección → enviar las peticiones **en orden**
   (01 a 10) y revisar la pestaña **Test Results**.

La lista completa de endpoints, cuerpos y respuestas está en `ENDPOINTS.txt`.

> Si su MySQL no usa el puerto 3306, ajuste `DB_PORT` en el archivo `.env`.

## Estructura del proyecto

```
ANDRESCARDENAS_AA5_EV02/
├── database/
│   └── crear-bd.sql            Script de la base de datos y la tabla usuarios
├── src/
│   ├── config/
│   │   └── database.js         Conexión (pool) a MySQL
│   ├── controllers/
│   │   └── auth-controller.js  Lógica de registro e inicio de sesión
│   ├── routes/
│   │   └── auth-routes.js      Definición de las rutas /register y /login
│   └── app.js                  Configuración de Express
├── pruebas/
│   └── AA5_EV02.postman_collection.json   Colección de Postman (10 pruebas)
├── server.js                   Punto de entrada (enciende el servidor)
├── ENDPOINTS.txt               Lista de endpoints de la API (entregable)
├── .env.example                Plantilla de configuración
├── .gitignore                  Archivos que Git no debe subir
├── package.json                Dependencias y scripts
├── ENLACE_REPOSITORIO.txt      Enlace al repositorio (entregable)
└── README.md                   Este documento
```

## Estándares de codificación

Se siguen los estándares definidos en el plan de trabajo del proyecto SGB:

- Variables y funciones en `camelCase`, con verbo al inicio (`registrarUsuario`).
- Constantes en `MAYUSCULAS_CON_GUION_BAJO`.
- Archivos en `kebab-case` (`auth-controller.js`).
- `const` / `let` (nunca `var`), comparación estricta `===` y plantillas de texto.
- Comentario de encabezado en cada módulo y comentarios explicativos en la lógica.

## Seguridad aplicada

- Contraseñas almacenadas como hash bcrypt (nunca en texto plano).
- Consultas SQL parametrizadas (`?`) para evitar inyección SQL.
- Restricción `UNIQUE` en `usuario` para impedir duplicados.
- Mensaje de error único para usuario inexistente o contraseña incorrecta.
- El archivo `.env` con datos locales no se sube al repositorio.

## Control de versiones

- Herramienta: **Git**, con repositorio remoto en **GitHub**.
- Flujo: *GitHub Flow* (rama `feature/pruebas-postman` integrada a `main`).
- Mensajes de commit: *Conventional Commits* (`feat:`, `chore:`, `test:`, `docs:`).
- El código de la API proviene de la evidencia AA5-EV01
  (repositorio `ANDRESCARDENAS_AA5_EV01`).

## Autor

Andrés Cárdenas – Aprendiz SENA, ficha 3186668.
