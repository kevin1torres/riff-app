# CLAUDE.md · Proyecto RIFF

Este archivo es el encargo completo. Léelo entero antes de escribir código.

**Referencias en esta carpeta:**
- `referencias/06_Design-system.pdf`: tokens y componentes.
- `referencias/07_Flujo-consumidor.pdf` y `referencias/08_Flujo-organizador.pdf`: cada pantalla y estado que hay que construir, con sus notas de heurísticas. Mira las páginas antes de construir cada fase.
- `referencias/09_Prototipo-web.pdf`: lo que ya existe en código.
- `referencias/01_Brief.pdf` y `referencias/02_Brief-UX.pdf`: contexto de producto.
- El código actual (`index.html`, `css/`, `js/`) es el punto de partida descrito abajo como `02_riff-app/`.
- Prototipo en Figma, si tienes acceso: https://www.figma.com/proto/80vaxDw05umaX0Ped7RyCk/RIFF---Wireframes--seg%C3%BAn-flujogramas---copia-?node-id=2012-118&starting-point-node-id=2012%3A74

---

## Rol y objetivo

Eres un desarrollador front end senior con buen ojo de diseño. Vas a construir **RIFF**, una web app mobile-first que centraliza y recomienda eventos de música en vivo en Bogotá, de cualquier género, formato de acto y tipo de venue. El usuario filtra por gustos y presupuesto, que declara a mano o trae al vincular Spotify o Apple Music, y descubre planes que hoy están dispersos entre redes sociales y páginas de venues. Lema: **"Cero fomo, siempre al tanto."**

Hay dos tipos de usuario:
1. **Consumidor (público):** descubre, filtra, guarda eventos y recibe recordatorios.
2. **Organizador, venue o artista:** verifica su cuenta, publica eventos y mide su alcance.

El diseño ya está cerrado en alta fidelidad (Figma «RIFF · Alta fidelidad»). Tu trabajo es llevarlo a código **fiel al diseño**, navegable de punta a punta y con datos simulados.

## Restricciones técnicas

- **Solo HTML, CSS y JavaScript nativos.** Sin frameworks (nada de React, Vue o Tailwind) y sin paso de build. Debe abrirse sirviendo la carpeta con cualquier servidor estático (`npx serve .` o la extensión Live Server).
- JavaScript con **ES modules** (`<script type="module">`), sin dependencias salvo dos permitidas por CDN:
  - **Leaflet** para el mapa de Bogotá, con teselas oscuras (CARTO Dark Matter) y su atribución visible.
  - **Google Fonts** para las tipografías.
- **SPA con router por hash** (`#/feed`, `#/evento/omar-courtz`…). Un solo `index.html`, con las pantallas renderizadas desde JS mediante funciones que devuelven template strings.
- **Estado persistente en `localStorage`**: sesión, tipo de perfil, gustos, presupuesto, eventos guardados, recordatorios, eventos publicados por el organizador y tutorial visto. Envuelve cada lectura y escritura en `try/catch`.
- Contenedor de app con ancho máximo de **430 px**, centrado en escritorio, con fondo de marca alrededor.
- Accesible: HTML semántico, `button` para acciones y `a` para navegación, `aria-current` en la tab bar, `aria-pressed` en guardar, foco visible, áreas táctiles de al menos 44 px, `alt` en imágenes, contraste WCAG AA y `prefers-reduced-motion`.
- Código en inglés y textos de interfaz en **español de Colombia**. Fechas como "jue 12 nov · 8:00 p. m." y precios en COP como "$189.000 – $577.000" (usa `Intl.NumberFormat('es-CO')` y `Intl.DateTimeFormat('es-CO')`).

## Estructura de archivos

```
riff/
├─ index.html
├─ css/
│  ├─ tokens.css        ← ya existe: reutilízalo tal cual
│  ├─ base.css          ← reset, tipografía, layout de app
│  ├─ components.css    ← botones, inputs, chips, cards, tab bar, toast, sheets…
│  └─ screens.css       ← ajustes por pantalla
├─ js/
│  ├─ main.js           ← arranque y router
│  ├─ router.js
│  ├─ store.js          ← estado + localStorage
│  ├─ data/events.js    ← eventos simulados
│  ├─ data/genres.js
│  ├─ components/       ← funciones que devuelven HTML (eventCard, chip, toast…)
│  └─ screens/
│     ├─ consumer/      ← una pantalla por archivo
│     └─ organizer/
└─ img/                 ← portadas de eventos (usa placeholders con gradiente si no hay imagen)
```

Parte del prototipo que ya existe en `02_riff-app/`: tiene `tokens.css`, `app.css`, `app.js` y tres pantallas (splash, perfil y feed). Conserva los tokens y el estilo visual, y reorganiza el código a la estructura de arriba.

## Design system (respétalo al pie de la letra)

**Color** (tokens de `css/tokens.css`):

| Token | Valor | Uso |
|---|---|---|
| `--bg-base` | `#100310` | Fondo |
| `--surface-default` | `rgba(99,56,102,.3)` | Cards y campos |
| `--surface-deep` | `#633866` | Superficie profunda |
| `--brand-primary` | `#E199D1` | Botón primario, selección, enlaces |
| `--brand-secondary` | `#765BA3` | Chips seleccionados, íconos |
| `--brand-accent` | `#CB6E74` | Acento, alertas suaves |
| `--status-danger` | `#933A50` | Error, destructivo, "+18" |
| `--text-primary` / `--text-secondary` | `#FFF` / `rgba(255,255,255,.65)` | Texto |
| `--text-on-primary` | `#100310` | Texto sobre botón primario |
| Gradiente de marca | `#FFF → #E199D1 → #CB6E74` | Logotipo y momentos clave |

Fondo de pantalla: `#100310` con manchas radiales difusas de violeta y rosa (como en las pantallas del Figma). Las barras superiores y la tab bar llevan efecto vidrio (`backdrop-filter: blur`).

**Tipografía:**
- **Unbounded 700** para logotipo y títulos, **siempre en minúsculas** ("tu feed", "mi calendario"). Tamaños: H1 32, H2 24, H3 20, H4 17.
- **Lunasima 400/700** para cuerpo: 16, 14 y 12 px.
- **Anton** para cifras y fechas: métricas a 48 px, días del calendario a 32 px.
- Usa las clases `.t-h1 … .t-fig-20` que ya están en `tokens.css`.

**Radios** 12 / 16 / pill. **Espaciado** en múltiplos de 4 px, con gutter de 16 px.

**Componentes que debes construir como funciones reutilizables:**
- **Button:** primario (relleno `#E199D1`, texto oscuro), secundario (superficie), ghost, deshabilitado, peligro y acento. Alto de 48 px y radio de 12.
- **Input:** estados normal, foco (borde rosa), error (borde y mensaje junto al campo, con ícono) y válido (check). Lleva label arriba y ayuda debajo.
- **Chip:** píldora de 44 px, seleccionable (con check) y removible (con ×).
- **Badge:** género, "Bar", "+18", "En revisión", "Publicado", "Rechazado", "Dentro de tu presupuesto" y "Fuera de tu presupuesto".
- **Event card** (vertical, con portada y botón de guardar en la card) y **event row** (horizontal, para listas y resultados).
- **Search bar, nav bar** (botón volver más título centrado), **tab bar** (Feed · Mapa · Calendario · Perfil, siempre en la misma posición) y **toast** (feedback inmediato con "Deshacer").
- **Pin del mapa** (normal y seleccionado, con color por género), **cluster** (número en Anton), **ubicación del usuario** y **controles del mapa** (vidrio).
- **Bottom sheet** con tres alturas: mínima, media y completa. Se arrastra y también responde a teclado.
- **Selector segmentado** (Venue · Artista · Organizador), **notification card** (según tus gustos, sorpréndeme, recordatorio, organizador) y **skeleton loader**.
- **Metric tile** (Anton), **barra de progreso por pasos** y **slider de rango doble** para el presupuesto en COP (de $0 a $300.000+), sincronizado con chips de atajo: Gratis · Hasta 50 mil · 50–150 mil · 150–300 mil · Sin límite.
- Botones de terceros: **Continuar con Spotify** (verde #1DB954) y **Continuar con Apple Music** (rojo/rosa), con sus logos.

## Datos simulados (`js/data/events.js`)

Crea al menos estos eventos, con id, título, artista, géneros, formato, tipo de venue, venue, dirección, coordenadas, fecha, rango de precio (o "Precios en Ticketmaster"), clasificación de edad, descripción, URL de venta y ticketera:

| Evento | Fecha | Venue | Precio | Género / formato |
|---|---|---|---|---|
| Omar Courtz: Por si mañana no estoy (segunda fecha, +12) | jue 12 nov · 8:00 p. m. | Movistar Arena (Dg. 61c # 26-36) | $189.000 – $577.000 · tuboleta.com | Urbano · Arena |
| Penyair: Desde Abajo Tour | dom 29 nov · 6:00 p. m. | Teatro Jorge Eliécer Gaitán | $98.200 – $271.400 | Rap · Teatro |
| Berliner Philharmoniker | mié 14 oct · 8:00 p. m. | Teatro Mayor Julio Mario Santo Domingo | $44.300 – $774.450 | Clásica · Orquesta |
| El Kalvo: 20 años de rap rolo | sáb 17 oct · 8:00 p. m. | Teatro Jorge Eliécer Gaitán | $88.800 – $99.900 | Rap |
| Slipknot + Marilyn Manson | jue 16 sep 2027 · 6:00 p. m. | Vive Claro | Precios en Ticketmaster | Metal · Arena |
| BAUM Festival 2027 | vie 12 – sáb 13 feb 2027 | Corferias | Precios en Ticketmaster | Electrónica · Festival |
| Bambulé · Viernes de Reggaetón | vie 16 oct · 8:00 p. m. | Bambulé, Cra. 13 # 83-47, Zona Rosa | $40.000 – $60.000 | Reggaetón · DJ set · Bar |
| Bambulé · Perreo de Halloween | sáb 31 oct · 9:00 p. m. | Bambulé | — | Reggaetón · Club |

Añade de 6 a 10 eventos más de escena local e independiente (bares en Chapinero, Video Club, jazz, salsa, cumbia, indie, eventos gratis, un DJ set, una orquesta juvenil) para que el feed, el mapa y los filtros tengan densidad real. Son datos de prueba: márcalos así en el código.

Géneros: Electrónica, Rap, House, Cumbia, Salsa, Indie, Jazz, Rock, Reggaetón, Urbano, Trap latino, Pop latino, Clásica y Metal. Formatos de acto: DJ set, En vivo, Acústico y Festival. Tipos de venue: Club, Bar, Teatro, Al aire libre y Arena.

## Flujo consumidor (pantallas y estados)

Sigue el orden y los estados de `07_Flujo-consumidor.pdf`. Cada pantalla lleva una nota de heurísticas de Nielsen: cúmplelas.

1. **Splash:** logotipo "riff" con gradiente y "todo en un mismo lugar". Pasa sola a la siguiente pantalla en unos 1,8 s; también se puede tocar para saltarla.
2. **00 · Selecciona tu perfil:** "¿cómo quieres usar riff?" con dos cards grandes: *Quiero descubrir eventos* y *Soy organizador, venue o artista*. Abajo, el aviso de Términos y Política de tratamiento de datos.
3. **01 · Crear cuenta:** nombre, apellido y correo o celular. Valida en tiempo real (correo o celular de 10 dígitos). El botón queda deshabilitado hasta que todos los campos son válidos. Incluye el enlace "¿Ya tienes cuenta? Inicia sesión". Hay un estado de error y otro de campos válidos.
4. **02 · Iniciar sesión:** correo o celular, más un código de 6 dígitos simulado.
5. **03 · Vincula tu música:** botones *Continuar con Spotify*, *Continuar con Apple Music* y *Prefiero definirlos manualmente*, con "Omitir" visible y del mismo peso. Lleva el aviso de privacidad "Solo leemos tus artistas y géneros más escuchados. Nunca publicamos nada."
   - **Spotify:** pantalla de autorización simulada, luego "importando tus gustos" con progreso y checklist, y después "¡listo, ya te conocemos!" con top artistas y géneros preseleccionados y **editables**.
   - **Apple Music:** diálogo de permiso del sistema simulado, luego importando y luego vinculado.
   - **Manual:** "tus gustos", con chips de género seleccionables.
6. **03 · Tu presupuesto:** slider doble más chips de atajo. Texto: "Te mostraremos si cada evento está dentro o fuera de tu presupuesto." Botones *Ir a mi feed* y *Omitir por ahora*.
7. **04 · Feed:** "hola, [nombre]", "tu feed", búsqueda, botón de filtros y campana con punto de no leídas. Secciones: *recomendado para ti* (carrusel horizontal), *cerca de ti* y *esta semana*. Estados: **cargando** (skeleton), **sin conexión** ("se fue la señal" con *Reintentar*) y **tutorial de bienvenida en 3 pasos** (busca y filtra · guarda sin entrar · explora el mapa), que se muestra solo la primera vez.
8. **05 · Buscar y filtrar:** búsquedas recientes, chips de género, formato de acto y tipo de venue. Abajo: *Limpiar* y *Aplicar filtros (n)* con el contador en vivo. Incluye la variante "escribir", con resultados mientras se teclea.
9. **06 · Resultados:** chips de filtros activos (removibles), conteo y lista de event rows. Estados: **sin resultados** ("no encontramos eventos" con *Quitar todos los filtros*) y **cargando**.
10. **07 · Detalle de evento:** portada grande, título en minúsculas, badges, fecha, venue con distancia y "Ver en Google Maps", y precio con nota "Incluye servicio · precios en COP". Muestra la badge **Dentro/Fuera de tu presupuesto** con el rango del usuario, la descripción, la acción de guardar y la de compartir (hoja nativa con `navigator.share` y respaldo de copiar enlace). CTA: *Comprar en página oficial →*. Antes de salir aparece el diálogo **"¿Salir de RIFF?"**: "Vas a abrir la página oficial de venta en [ticketera]. RIFF no procesa pagos ni guarda tus datos de compra." con *Cancelar* y *Abrir página*. Debe funcionar con todos los eventos.
11. **08 · Guardado y recordatorio:** "¡evento guardado!", mini card, recordatorio (1 día antes por defecto) editable en una hoja (1 hora, 3 horas, 1 día, 2 días o sin recordatorio), *Volver al feed* y toast con *Deshacer*.
12. **09 · Mi calendario:** mes navegable con días en Anton, con los eventos guardados marcados y el día de hoy. Debajo, la lista "mis eventos guardados". Estado vacío: "aún no tienes planes".
13. **10 · Mapa de Bogotá:** Leaflet oscuro, pines por género, clusters, ubicación del usuario y bottom sheet "12 eventos cerca de ti" con búsqueda. Estados: hoja media (lista), hoja completa, **pin seleccionado** (mini card), zoom en un cluster, **sin permiso de ubicación** (Bogotá centrada más un aviso), **zona sin eventos** (*Buscar en todo Bogotá*) y **cómo llegar** (hoja que abre Google Maps, Apple Maps o Waze).
14. **11 · Mi perfil:** avatar con iniciales, nombre, correo y opción de editar. Filas: *Mis gustos musicales*, *Presupuesto* (editable con el mismo selector), *Cuentas vinculadas* (vincular o desvincular en un paso, mostrando el proveedor) y *Ayuda* (incluye repetir el tutorial). *Cerrar sesión* pide confirmación. Pie: "RIFF v1.0 · Hecho en Bogotá".
15. **12 · Notificaciones:** agrupadas en *hoy* y *esta semana*. Tipos: *Según tus gustos*, **Sorpréndeme** (algo distinto, como clásica para un fan del rap) y *Recordatorio*. Al tocar una, abre el evento.

**Recomendación (simple y explicable):** ordena por coincidencia de género con los gustos, luego por presupuesto y luego por cercanía o fecha. Guardar un evento suma peso a sus géneros. "Sorpréndeme" elige un evento de un género que el usuario **no** tiene.

## Flujo organizador, venue o artista

Sigue `08_Flujo-organizador.pdf`.

1. **01 · Crear cuenta de organizador:** nombre, correo o celular, y tipo de cuenta con selector segmentado (Venue · Artista · Organizador). Incluye estados de error y válido.
2. **02 · Verifica tu negocio:** subida simulada de documentos (Permiso de funcionamiento, Cámara de comercio, RUT) con estados subido, subiendo (con progreso) y pendiente. Texto: "La revisión puede tardar hasta 48 horas". Estado **rechazada**, con el motivo ("tu cámara de comercio fue expedida hace más de 30 días") y *Corregir y reenviar*.
3. **03 · Perfil público:** portada, avatar, badge "Verificado", nombre, tipo, zona, descripción, ubicación, galería de fotos y redes. Botón *Guardar y continuar*.
4. **04 · Iniciar sesión (organizador).**
5. **05 · Tu panel:** *+ Publicar nuevo evento*, resumen de 7 días (Vistas · Guardados · Clics a venta, en Anton) y "mis eventos", cada uno con estado (Publicado, En revisión, Rechazado) y acciones Métricas, Editar, Duplicar y Cancelar. Cancelar pide confirmación y explica a cuántas personas se avisará.
6. **06 · Nuevo evento, en 4 pasos** con barra de progreso:
   1. **Lo básico:** "Duplicar un evento anterior", nombre, organizadores (con @mención), portada de 1080 × 1080.
   2. **Fecha y lugar:** dirección con vista previa en el mapa y selector de fecha y hora.
   3. **Clasificación y precio:** chips de género, formato y tipo de venue, precio desde y hasta en COP, y edad (Todo público · +14 · +18).
   4. **Venta y permisos:** link de venta oficial validado, permiso PMU de la Alcaldía con tooltip y restricciones opcionales (máximo 200 caracteres). Botón *Enviar a revisión*.

   El paso actual se guarda en `localStorage`. *Siguiente* solo se activa con los campos obligatorios completos, y *Atrás* conserva lo escrito.
7. **07 · Tu evento está en revisión:** línea de tiempo Enviado → En revisión → Publicado, y "(Prototipo) Simular aprobación →".
8. **08 · ¡Evento publicado!:** el evento aparece en el feed, el mapa y los filtros del consumidor. Hazlo de verdad: agrégalo al store y que se vea desde el lado consumidor.
9. **09 · Métricas:** tiles (vistas, guardados, clics a venta), gráfica de línea de 7 días hecha a mano en SVG (vistas contra guardados), mejor día y "tasa de guardado" en porcentaje.
10. **10 · Notificaciones del organizador:** evento aprobado, hito alcanzado ("¡30 guardados!") y recordatorio para actualizar datos.

## Calidad y comportamiento

- Transiciones suaves entre pantallas (fade o slide de 200–250 ms), desactivadas si `prefers-reduced-motion`.
- La tab bar solo aparece en Feed, Mapa, Calendario y Perfil, y siempre en la misma posición.
- El botón volver del sistema y de la nav bar funcionan, porque todo pasa por el router.
- Cada acción tiene feedback inmediato (toast, estado del botón o skeleton). Toda acción destructiva pide confirmación o ofrece Deshacer.
- Sin errores en consola. Rendimiento: imágenes con `loading="lazy"` y tamaños definidos.
- Botón oculto de "Reiniciar demo" en Perfil › Ayuda, que limpia `localStorage`.

## Cómo entregar el trabajo

Trabaja por fases y muéstrame el resultado al final de cada una antes de seguir:
1. Base: estructura, router, store, tokens y todos los componentes en una página `#/componentes` que funcione como catálogo.
2. Onboarding consumidor: splash, perfil, crear cuenta, iniciar sesión, vincular música (las tres rutas) y presupuesto.
3. Feed, búsqueda y filtros, resultados y detalle con el aviso de salida.
4. Guardado, calendario, notificaciones y perfil.
5. Mapa de Bogotá con todos sus estados.
6. Flujo organizador completo, conectado con el lado consumidor.
7. Pulido: accesibilidad, estados vacíos y de error, animaciones y revisión contra los PDFs, pantalla por pantalla.

**Criterio de terminado:** se puede recorrer cada pantalla y cada estado de los dos PDFs de flujo desde la app, sin tocar el código, y cada una se ve como en el diseño.
