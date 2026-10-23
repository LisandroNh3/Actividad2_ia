# AstroLogistics · Evaluación II

Solución local de los dos problemas: **Estabiliza el Reactor** y **AstroLogistics Ltd.** React + Vite + Reactstrap en el frontend, Node.js + Express en el backend. La explicación y las preguntas de defensa están en `EXPLICACION_Y_DEFENSA.txt`.

## Requisitos y ejecución

Instalar Node.js 22.12 o posterior (recomendado Node 24 LTS). Desde esta carpeta:

```sh
npm install
npm run dev
```

Abrir la dirección que muestre Vite, normalmente **http://127.0.0.1:5173**. La API funciona en el puerto 3001. En PowerShell, si `npm` está bloqueado por la política de scripts, usar `npm.cmd` en todos los comandos, por ejemplo `npm.cmd run dev`.

Para ejecutar la versión compilada:

```sh
npm run build
npm start
```

Abrir **http://localhost:3001**. Para comprobar la API: `npm test`. Para detener los servidores: Ctrl+C. No abrir `index.html` con doble clic: necesita Vite o Express.

## Estructura

- `server/app.js`: rutas, inventario, sesiones y reglas del negocio.
- `server/index.js`: arranque del servidor.
- `src/main.jsx`: componentes App, Reactor y Logistica, estados y efectos.
- `src/api.js`: comunicación con la API y manejo de errores HTTP.
- `src/styles.css`: estilos y adaptación a móviles.
- `vite.config.js`: proxy de desarrollo; evita necesitar CORS.
- `tests/api.test.js`: pruebas HTTP de las reglas del negocio.

## Decisiones del caso de negocio

Cada misión inicia con 500 créditos y 3 minutos. Se reserva stock al agregar, se devuelve al reducir/eliminar o expirar y se mantiene descontado al confirmar. Esto resuelve la combinación del enunciado que pide carga temporal, descuento inmediato del stock y devolución al eliminar. Confirmar convierte la reserva en definitiva y bloquea modificaciones. Iniciar otra misión cancela las reservas activas de la anterior, pero conserva el consumo de misiones confirmadas.

La API recalcula precios, totales y saldo: no acepta valores financieros del navegador. El contador visual usa el vencimiento informado por el servidor; el servidor comprueba el plazo antes de procesar las operaciones. Las reservas vencidas se liberan al llegar la siguiente petición de API. No se incluyó un peso en kilogramos ni una capacidad ficticia porque el enunciado no proporciona esos valores; las restricciones implementadas son stock, presupuesto y tiempo.

Estado en memoria, permitido por el enunciado: reiniciar el servidor restaura el inventario y elimina las misiones. El reactor es una variable global del servidor, como pide el ejercicio: su reinicio afecta a todos los navegadores. Las misiones usan un UUID guardado en `sessionStorage` por pestaña; sirve para identificar la demostración y **no constituye autenticación**. El catálogo se refresca al cargar y después de las operaciones; cambios de otros navegadores se validan siempre en el backend, aunque el catálogo local esté desactualizado.

## API

Enviar `Content-Type: application/json` y, para carga, `X-Mision-Id` con el ID devuelto por `POST /api/mision`.

| Método y ruta | Cuerpo | Resultado |
| --- | --- | --- |
| POST `/adjust` | `{ "temperatura": 50 }` | Pista y booleano `estabilizado` |
| POST `/reboot` | — | Nueva temperatura aleatoria entre 1 y 100 |
| POST `/api/mision` | — | Nueva misión con 500 créditos y vencimiento |
| GET `/api/inventario` | — | Array de suministros y stock libre |
| GET `/api/carga` | — | Créditos restantes, items, total y estado |
| POST `/api/carga/agregar` | `{ "equipoId": 201, "cantidad": 1 }` | Reserva unidades |
| PUT `/api/carga/actualizar-unidades/:id` | `{ "cantidad": 2 }` | Establece cantidad absoluta |
| DELETE `/api/carga/eliminar/:id` | — | Elimina y devuelve unidades |
| POST `/api/carga/confirmar` | — | Bloquea el manifiesto |

HTTP: 201 creación, 200 éxito, 400 entrada inválida o restricción de negocio, 401 misión ausente, 404 recurso ausente, 409 carga confirmada, 410 misión expirada.

## Recorrido para revisar

1. Iniciar misión. Agregar oxígeno: stock 5 → 4, total 50, saldo 450.
2. Subir a 2: stock 3, total 100, saldo 400. Bajar a 1 y desembarcar: stock 5 y saldo 500.
3. Agregar 2 sondas y 4 tanques: total 500. Pulsar + en tanques: error de créditos sin cambiar estado.
4. Desembarcar las sondas, subir tanques a 5 y pulsar +: error de stock. El catálogo marca agotado.
5. Confirmar con carga: mensaje de éxito y controles bloqueados.
6. Iniciar otra misión y dejar transcurrir 3 minutos: sesión cerrada, carga liberada; se invita a iniciar otra.
7. Reactor: probar 50 y seguir las pistas; al acertar se desactiva el formulario. Reiniciar borra el historial.

## Entrega y colaboración

El repositorio remoto del equipo no se crea ni se publica automáticamente. Un integrante debe crearlo e invitar a sus compañeros; cada integrante debe realizar y comprender sus aportes reales. Subir el código y `package-lock.json`, excluyendo `node_modules` y `dist`. Usar ramas por tarea, commits descriptivos y revisión antes de integrar. No inventar participación de otros integrantes. Verificar desde otro clon con `npm ci`, `npm test` y `npm run build`.

## Verificación realizada

Las cinco pruebas HTTP automatizadas pasan y la compilación de producción con Vite finaliza correctamente. Express sirve el HTML, JavaScript y CSS compilados con HTTP 200. No se realizó una revisión visual en navegador porque no había uno conectado disponible en la sesión.
