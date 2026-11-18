# 🫧 Lavadero Espuma

Reservas online para un lavadero de autos (negocio ficticio) con dos boxes de lavado. El cliente carga sus vehículos, elige el servicio, el día y la hora, y el sistema se encarga de lo demás: calcula el precio y la duración según el tipo de vehículo, asigna un box libre, aplica el programa de fidelidad y bloquea a quien falta seguido.

Proyecto Final — hecho con el método **spec-first con IA**: primero se acordaron las reglas con la clienta (spec), después el plan técnico, después los tests y recién al final el código.

> 🌐 Demo: `https://TU-SITIO.netlify.app` *(completar después del deploy)*

## 📄 Documentación

- [`docs/spec.md`](docs/spec.md): entrevista con la dueña y las **8 reglas del negocio** (E1 a E8)
- [`docs/plan.md`](docs/plan.md): arquitectura, capas, modelo de datos y API
- [`docs/tareas.md`](docs/tareas.md): tareas derivadas de la spec y el test que cubre cada una

## ⚙️ Reglas en una mirada

| Regla | Resumen |
|---|---|
| E1 Horario | Lun a vie 8–18 h · sáb 8–13 h · dom cerrado · bloques de 30 min |
| E2 Anticipación | Entre 2 horas y 14 días |
| E3 Vehículos | Patente ABC123 o AB123CD · hasta 3 por cliente · 1 reserva pendiente por vehículo |
| E4 Precio | Auto ×1 · SUV ×1,25 (+30 min) · Pick-up ×1,40 (+30 min) · redondeo a la centena |
| E5 Boxes | 2 boxes, asignación automática; sin box libre → 409 |
| E6 Fidelidad | Cada 5 lavados realizados, uno gratis |
| E7 Faltas | Cancelar con menos de 3 h o no venir = falta · 2 faltas en 30 días → bloqueo de 15 días |
| E8 Cuentas | Registro con email único · admin inicial por variables de entorno |

### Preguntas de control

| Situación | Resultado |
|---|---|
| Tres clientes piden el mismo horario a la vez | Box 1, Box 2 y un 409 |
| Lavado completo para una SUV | $13.800, 90 min |
| Cliente con 5 lavados realizados | Su siguiente reserva cuesta $0 |
| Segunda cancelación tardía del mes | Bloqueo de 15 días |

## 🧱 Tecnologías

| | Tecnología | En mi compu | En internet |
|---|---|---|---|
| Cliente | React 19 + Vite | `localhost:5173` | Netlify |
| Servidor | Node.js + Express 5 | `localhost:3000/api` | Render |
| Datos | PostgreSQL | local | Neon |
| Tests | Vitest + Supertest | — | — |

```
client/   interfaz (views/, components/, lib/)
server/
  src/domain/   reglas puras: horario, vehículos, fidelidad, faltas, tiempo, reloj
  src/routes/   endpoints REST
  src/db/       esquema, conexión y datos iniciales
  tests/        domain.test.js (39) · api.test.js (38)
docs/     spec, plan y tareas
```

## ▶️ Correrlo localmente

Requisitos: Node 20 o más nuevo y PostgreSQL.

```bash
createdb espuma
createdb espuma_test          # solo para tests: se vacía en cada corrida

# servidor
cd server
npm install
cp .env.example .env          # completar DATABASE_URL
npm run dev                   # http://localhost:3000/api/health

# cliente (otra terminal)
cd client
npm install
cp .env.example .env
npm run dev                   # http://localhost:5173
```

Al arrancar, el servidor crea las tablas, los 3 servicios, los 2 boxes y el usuario de recepción definido en `INITIAL_ADMIN_EMAIL` / `INITIAL_ADMIN_PASSWORD`.

### Tests

```bash
cd server
cp .env.test.example .env.test   # apuntar a espuma_test
npm test
```

→ **77 tests en verde.** El "reloj" del sistema se detiene y se adelanta dentro de los tests, así las reglas de 3 horas, 30 días y 15 días se prueban en milisegundos.

## 🚀 Publicarlo

1. **Neon**: crear el proyecto y copiar la cadena de conexión.
2. **Render** → New → Blueprint (usa `render.yaml`) o Web Service con *Root Directory* `server`. Cargar `DATABASE_URL`, `INITIAL_ADMIN_EMAIL`, `INITIAL_ADMIN_PASSWORD` y `CORS_ORIGIN` (la URL de Netlify).
3. **Netlify** → Import from Git (usa `netlify.toml`). Cargar `VITE_API_URL` = `https://<tu-app>.onrender.com/api`.
4. Después de eso, cada `git push` publica solo.

> Render gratis se duerme: el primer pedido después de un rato puede tardar unos 30 segundos.

## 🔒 Seguridad

- Los secretos nunca se suben al repo: viven en `.env` (ignorado por git) o en el panel de cada servicio.
- Las contraseñas se guardan con bcrypt y las sesiones se manejan con JWT.
- CORS limitado al frontend publicado.
- El choque de reservas en un mismo box lo impide la propia base de datos (restricción `EXCLUDE`), además del código.
