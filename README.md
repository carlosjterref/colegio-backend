# Colegio Yermo y Parres — Backend (API REST)

API REST del **Sistema de Información Académico** del Colegio Yermo y Parres.
Gestiona la autenticación por rol, las calificaciones, la mensajería interna,
las publicaciones institucionales y el registro de estudiantes.

> Este repositorio contiene **únicamente el backend**.
> El cliente que lo consume está en [colegio-frontend](https://github.com/carlosjterref/colegio-frontend).

---

## Tecnologías

| Componente | Tecnología |
|------------|------------|
| Servidor | Node.js + Express 4 |
| Base de datos | MySQL (MariaDB / XAMPP) |
| Autenticación | jsonwebtoken (JWT) + bcryptjs |
| Driver BD | mysql2 (con pool de conexiones) |
| Otros | cors, dotenv, nodemon (desarrollo) |

## Estructura

```
├── config/db.js          # Pool de conexiones MySQL
├── middleware/auth.js    # verificarToken y soloRol (autenticación y autorización)
├── routes/               # Endpoints por recurso
│   ├── auth.js               alumnos.js      docentes.js
│   ├── acudientes.js         materias.js     notas.js
│   ├── comunicaciones.js     noticias.js     circulares.js
│   ├── inscripciones.js      festivos.js
├── scripts/              # Esquema SQL y utilidades
│   ├── esquema.sql               # Estructura completa (11 tablas)
│   ├── cargar-esquema.js         # Ejecuta el esquema contra la BD configurada
│   ├── crear-admin.js            # Administrador + noticia de ejemplo
│   ├── plantel-docentes.js       # 45 docentes del plantel
│   ├── datos-prueba.js           # Materias, notas y mensajes de ejemplo
│   └── ...                       # Encriptar/resetear contraseñas, migración
├── server.js             # Punto de entrada
└── .env.example          # Plantilla de variables de entorno
```

## Requisitos

- **Node.js 18** o superior (usa `fetch` nativo)
- **MySQL** (se recomienda XAMPP)

## Instalación

```bash
# 1. Instalar dependencias
npm install

# 2. Configurar variables de entorno
cp .env.example .env      # edita .env con tus datos

# 3. Crear la base de datos y las tablas
node scripts/cargar-esquema.js

# 4. (Opcional) Cargar datos de prueba
node scripts/crear-admin.js
node scripts/plantel-docentes.js

# 5. Iniciar el servidor
npm run dev               # o: node server.js
```

La API queda disponible en **<http://localhost:3000/api>**.
Verifica con: `GET /api/health`

## Variables de entorno

```env
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=
DB_NAME=proyecto
DB_PORT=3306

JWT_SECRET=una_clave_larga_y_aleatoria
JWT_EXPIRES_IN=8h

PORT=3000
```

> El archivo `.env` **no se versiona**. `config/db.js` también acepta las variables
> `MYSQL*` que inyectan algunos servicios de despliegue.

## Endpoints

Base: `/api` · Las rutas protegidas requieren `Authorization: Bearer <token>`.

| Método | Endpoint | Acceso | Descripción |
|--------|----------|--------|-------------|
| POST | `/auth/login` | Público | Inicia sesión (usuario, correo o documento) y devuelve el JWT |
| PUT | `/auth/password` | Autenticado | Cambia la contraseña propia |
| GET | `/alumnos` · `/alumnos/:id` | Autenticado | Consulta de estudiantes |
| POST/PUT/DELETE | `/alumnos` | Docente | Gestión de estudiantes |
| GET | `/docentes/plantel` | **Público** | Plantel docente |
| GET/POST/PUT/DELETE | `/docentes` | Autenticado / Docente | Gestión de docentes |
| GET/POST/PUT/DELETE | `/acudientes` | Docente | Gestión de acudientes |
| GET | `/materias` · `/materias/:id/alumnos` | Autenticado | Materias y alumnos inscritos |
| GET | `/notas/alumno/:id` · `/notas/materia/:id` | Autenticado / Docente | Consulta de notas |
| POST/PUT/DELETE | `/notas` | Docente | Registro y edición de notas |
| GET | `/comunicaciones/recibidos` · `/enviados` | Autenticado | Bandeja de mensajes |
| POST | `/comunicaciones` | Autenticado | Enviar mensaje |
| PATCH | `/comunicaciones/:id/leer` | Autenticado | Marcar como leído |
| GET | `/noticias` · `/circulares` | **Público** | Contenidos publicados |
| POST/PUT/DELETE | `/noticias` · `/circulares` | Administrador | Gestión de contenidos |
| POST | `/inscripciones` | **Público** | Registra estudiante + acudiente (transaccional) |
| GET | `/festivos` | **Público** | Festivos de Colombia (**consumo de API externa**) |
| GET | `/health` | **Público** | Estado de la API |

## Consumo de API externa

`routes/festivos.js` consume la API pública [Nager.Date](https://date.nager.at)
para obtener los días festivos de Colombia y los expone en `/api/festivos`.
Incluye **caché en memoria** (12 h) y manejo de errores, evitando saturar el servicio externo.

## Seguridad

- Contraseñas con **hash bcrypt**; nunca se almacenan en texto plano.
- **JWT** firmado con expiración; el rol viaja dentro del token.
- **Autorización por rol** con el middleware `soloRol`.
- Operaciones sensibles usan la identidad **del token**, no datos del cliente.
- **Consultas parametrizadas** en todo el acceso a datos (previene inyección SQL).
- **Transacciones** en operaciones compuestas (p. ej. el registro de inscripciones).

## Modelo de datos

Once tablas: `administrador`, `docente`, `alumno`, `padreacudiente`, `materia`,
`materia_alumno`, `nota`, `comunicacion`, `comunicacion_receptor`, `noticia` y `circular`.
Las relaciones muchos a muchos se resuelven con tablas intermedias, con integridad
referencial y borrado en cascada.

## Repositorios relacionados

| Repositorio | Contenido |
|-------------|-----------|
| [proyecto-ficha83](https://github.com/carlosjterref/proyecto-ficha83) | Proyecto completo (frontend + backend) |
| [colegio-frontend](https://github.com/carlosjterref/colegio-frontend) | Cliente / sitio web |

---

**Autor:** Carlos Terreros — Proyecto Ficha 83
