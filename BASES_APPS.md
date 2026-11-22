# BASES — lo que TODA app mía debe tener (v3)

> Se copia en cada repo nuevo. Es la lista de lo que siempre termino corrigiendo
> a última hora, ya resuelto de entrada.

---

## 0. Cómo trabajamos

- Yo trabajo en **VS Code**. Claude trabaja **directo en GitHub y en Apps Script**,
  ya tiene mis datos y accesos.
- Entregas siempre en **archivo completo corregido**, nunca parches parciales ni
  entregas por fases.
- **HTML y backend comparten el mismo número de versión** en el encabezado.
- Siempre partir de la **última versión buena entregada**, no de un intermedio roto.

---

## 1. IA: Gemini primero, Claude de respaldo

```js
// GEMINI (gratis / cuota)
const GEMINI_KEY = 'AQ.Ab8RN6IPQG-y1DpGRzxq9cZAwV7d8OJF_dJsk6vNpZYDWvNr8A';
const GEMINI_API = 'https://generativelanguage.googleapis.com/v1/models/gemini-3.6-flash:generateContent?key=' + GEMINI_KEY;

// CLAUDE (de pago — respaldo)
const CLAUDE_KEY   = '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';
const CLAUDE_MODEL = 'claude-haiku-4-5-20251001';
const CLAUDE_API   = 'https://api.anthropic.com/v1/messages';
```

- **Gemini es el default.** Claude solo entra si Gemini falla.
- **Probar el nombre del modelo con `curl` antes de deployar.** Los nombres caducan
  en meses y un modelo muerto se cae **en silencio** a Claude sin error visible
  (o sea, gastando dinero sin avisar).
- **Siempre mostrar con qué IA se trabajó** (Gemini o Claude) en cualquier
  resultado generado por IA.
- Si Gemini falló **por falta de créditos** específicamente, decirlo explícito. No
  mostrar "Claude" como si fuera una elección normal.
- **Prompts ligeros.** Probar el tiempo de respuesta con una entrada real y pesada
  antes de confiar en un prompt nuevo. Un prompt mal pensado convirtió una
  respuesta de ~10s en una de 3-5 MINUTOS y encima peor.

---

## 2. Usuarios y contraseña

- Al abrir, la app pregunta el nombre de usuario **solo la primera vez**.
- Debe poderse **agregar más usuarios después, o editar los actuales**, sin
  rehacer nada.
- Preguntar **si se quiere poner contraseña** de inicio de sesión (opcional, no
  forzada).
- El usuario activo se recuerda en `localStorage`, con opción de **cambiar de
  usuario**.
- Definir desde el diseño **si los datos son compartidos o por usuario**, y si se
  guarda quién editó qué. Eso no se puede parchar después.

---

## 3. Botones de Guardar

- Al presionar: **se deshabilita y cambia su texto** a "Guardando…" (o con
  progreso: "Generando 2/4…") durante todo el proceso.
- Al terminar: dice **"Guardado"** y ahí se queda.
- Solo vuelve a decir **"Guardar" si se modifica algo de nuevo**.
- No basta un toast que puede desaparecer antes de que termine el guardado.
- **Bloquear el reenvío mientras guarda** — doble tap = registro duplicado.

---

## 4. Todo botón se ve cuando se presiona

- Cambio visible al clic: color, borde, marcado, lo que sea — que se sepa qué se
  apretó.
- Los filtros/toggles activos se quedan marcados mientras estén activos.
- Targets grandes para dedo (pruebo en celular Samsung).

---

## 5. Eliminar: confirmar + papelera

- Todo botón de eliminar pasa por **un mismo modal de confirmación reusable**
  (uno solo para toda la app, no uno por pantalla).
- **Eliminar = mandar a papelera** (recuperable).
- **Borrar para siempre** es una acción aparte, con su propia confirmación.
- La papelera **se vacía sola** después de X días, si no crece para siempre.

---

## 6. Clics fuera: qué cierra y qué no

- **Modal o formulario con cambios sin guardar:** si doy clic afuera (o en la ✕),
  **preguntar antes de cerrar**. Nunca perder lo escrito por un clic mal dado.
- **Lista desplegada, menú, dropdown, selector:** clic afuera **la esconde de
  inmediato**, sin preguntar nada.

---

## 7. Offline e instalable en el celular (PWA)

- `manifest.json` con ícono SVG inline y `display: standalone` → así el celular
  ofrece "Agregar a pantalla de inicio" y se abre como app.
- `sw.js` (service worker) con estrategia **red primero**: intenta internet, y solo
  si no hay conexión usa el caché.
- En el documento principal, `cache: 'no-store'` — si no, se queda pegada la
  versión vieja.
- Guardar en `localStorage` **el último dato cargado bien**, para poder **ver**
  (no editar) sin conexión.
- Hosting en **GitHub Pages** para tener URL estable y HTTPS (la cámara solo
  funciona con HTTPS).

---

## 8. Fotos: cámara o galería, y recortar antes de subir

- El `<input type="file">` **NO** lleva `capture="environment"` — ese atributo a
  veces salta directo a la cámara y quita la opción de galería. Sin él, el celular
  deja elegir.
- Siempre **dejar recortar antes de mandar la foto** (mismo recortador de cuadro
  arrastrable en todas las apps).
- Cualquier manipulación de imagen dibuja desde un `<canvas>`/`<img>` **ya
  cargado**, nunca desde una `Image()` recién creada — decodificar un data-URL es
  asíncrono.
- **Nunca escribir base64 directo en una celda de Sheets** (límite 50k caracteres).
  Se sube a Drive y se guarda la URL.

---

## 9. Conexión: un solo enlace

- El backend es dueño de **un solo Google Sheet**; su ID vive en las Propiedades
  del script y se crea solo la primera vez que hace falta.
- La pantalla de inicio pide **solo la URL de la Web App**, guardada en
  `localStorage`.
- No pedir también el link del Sheet: no suma seguridad real (quien tenga la URL
  de la Web App ya puede mandar cualquier sheetId) y es un dato más que se puede
  pegar mal.
- Esa URL de la Web App **nunca** va en el código público.

---

## 10. Repo y hosting

- **Una app = una carpeta = un repo = un Pages.** Nunca dentro del repo de otra app.
- Agregar la carpeta nueva al `.gitignore` de la carpeta raíz.
- `index.html` que redirige al archivo real de la app.
- El backend (`apps-script/`) va en su propio `.gitignore` y **nunca** se sube a
  GitHub — ahí viven las llaves.

---

## 11. Backend genérico

- Funciones **reusables** para las tablas del Sheet (definir tabla, leer, convertir
  filas a objetos, buscar fila por ID, guardar/borrar con `LockService`) en vez de
  reinventar el CRUD en cada app.
- Rutas **GET y POST consistentes**: si una función existe en `doGet`, también
  tiene que estar conectada en `doPost`.
- Papelera, usuarios, notas y catálogos son parte de este backend genérico, no de
  cada app.
- Los catálogos viven en **una sola hoja de catálogos** (fuente única de verdad),
  no repartidos ni deducidos de los registros existentes.
- **Botón de exportar / respaldo** (CSV o copia del Sheet) en configuración.

---

## 12. Configuración: siempre lleva

- **Versión visible, en palabras** (no números), con un **botón al lado que fuerce
  la actualización** — como trabajamos offline, a veces aunque actualice no se
  actualiza el software.
- **Apartado de Mejoras** (ideas, quejas, cosas que se me ocurran para la app):
  - Cuadro de texto arriba + botón **Agregar**; se van **guardando solas**.
  - Abajo, el **listado** de las entradas con su **fecha**.
  - Cada entrada tiene **copiar** y **eliminar** (los dos iconitos juntos, a la
    derecha), y es **editable** — se toca y se corrige ahí mismo.
  - **Sin botón de "limpiar todo"**: si puedo borrar una por una, no hace falta y
    solo se presta a borrar todo por accidente.
- **Regla general: todo lo que yo cree dentro de la app se puede agregar, editar
  y quitar desde Configuración.** Sin excepciones y sin tocar el Sheet a mano ni
  el código. Eso incluye catálogos y clasificaciones (categorías, tipos,
  etiquetas, proyectos, monedas, tarjetas, idiomas…), **nombres de usuario**,
  **títulos** de listas, sesiones, secciones o vistas, y cualquier cosa que se
  vaya creando con el uso. Si aparece un nombre puesto por mí, tiene que poderse
  corregir.
  Reglas por acción:
  - **Editar nombre (dedazo):** siempre permitido. Al renombrar, **se actualizan
    los registros que lo tenían** — nunca dejar huérfanos.
  - **Agregar:** siempre permitido. Si no, termino creando basura desde el
    formulario a la hora de capturar.
  - **Quitar:** solo si **no está en uso**. Si ya tiene registros, no se borra:
    se **archiva** — deja de aparecer al capturar cosas nuevas, pero los
    registros viejos conservan su nombre y no se rompe el historial. Avisar
    siempre cuántos registros lo usan.
  - **Reordenar** para que lo que más uso quede arriba.
  - Nada de listas quemadas en el HTML. Todo catálogo vive en el Sheet y se lee
    de ahí.
- Cambiar de usuario / editar usuarios.
- Exportar respaldo.

---

## 13. Cosas que siempre se me olvidan y truenan

- **Todo el JavaScript en un solo bloque `<script>`** — si no, hay conflictos de
  hoisting y la app se congela.
- El **primer deploy siempre da 403** hasta correr cualquier función una vez desde
  el editor (eso autoriza permisos).
- Los **booleanos de Sheets** se leen con cuidado:
  `v === true || String(v).toLowerCase() === 'true'`. Sheets no regresa el tipo que
  esperas.
- **UI optimista**: pintar el cambio de inmediato, guardar en segundo plano,
  revertir si falla.
- **Mostrar el error real en pantalla**, no un "Error" genérico.
- **Estados vacíos y de carga** decentes — nada de pantalla en blanco mientras
  responde GAS.
- **Filtros, búsqueda y orden sobreviven al recargar** (`localStorage`).
- **Tablas en celular**: scroll horizontal o vista de tarjeta, nunca que se corte.
- **Fechas y hora en local**: que funcione igual en Japón, en México o donde esté,
  sin zona horaria quemada en el código.
- **El botón de atrás del celular no debe sacarme de la app**: que cierre el modal
  abierto o regrese a la vista anterior — un atrás de verdad. En PWA eso se maneja
  a mano (historial propio), no sale gratis.
- **Ligas clickeables**: si un texto trae una URL, que se detecte sola, se pinte en
  azul y se pueda abrir. Aplica en notas, mejoras, comentarios, cualquier campo
  libre.
- Confiar en lo que reporto que pasa en el celular por encima de cualquier prueba
  simulada.
