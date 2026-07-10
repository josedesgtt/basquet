# De la anécdota al arma letal — Estado del proyecto

Scrollytelling sobre la evolución del triple en la NBA. Construido 100% a medida (HTML/CSS/JS + D3.js + Scrollama) — **no se usa Flourish**, para mantener consistencia visual en toda la pieza, a pesar de que el guión original lo sugería en varios tramos.

---

## 1. Sistema de diseño

**Paleta:**
- Papel cancha (fondo): `#F2E8D5`
- Naranja pelota (acento principal / resaltado): `#D9642E`
- Azul tablero (texto / tinta): `#1B3654`
- Rojo programa (acento secundario / peligro): `#B23B32`
- Verde pileta (poco usado hasta ahora): `#3D8792`
- Gris muted (texto secundario): `#5b6b80` o `#8a97a8` según el tramo

**Tipografías (Google Fonts):**
- **Big Shoulders Display** (600/800) — títulos, cifras grandes, remates. Uppercase, a veces con `-webkit-text-stroke` en azul sobre relleno naranja.
- **Public Sans** (400/500/600) — cuerpo de texto.
- **Martian Mono** (400/500) — eyebrows, datos, ejes, mono en general.

**Texturas y tratamientos recurrentes:**
- **Halftone**: `radial-gradient(circle, rgba(27,54,84,0.13) 1.3px, transparent 1.3px)` tamaño 9px — fondo de casi todas las secciones.
- **Duotono en fotos reales**: `filter: grayscale(1) contrast(1.15) brightness(0.92)` + una capa/círculo con `mix-blend-mode: color` en gradiente azul→rojo (o azul→naranja para Curry). Se usó en Chris Ford, Miller, foto del draft, los 12 pívots.
- **Fondo blanco en fotos**: casi todas las fotos de jugadores son JPEG con fondo de estudio blanco "quemado" (no son PNG transparentes reales). Hay que procesarlas con PIL antes de usarlas: cualquier píxel con r,g,b > 235 pasa a alpha 0.
- **"Ficha de programa" (step-card)**: fondo papel cancha sólido, borde duro 2px + borde izquierdo grueso de color, `box-shadow` offset duro (no blur), rotación alterna leve por step (`nth-child(odd/even)`), numerador circular generado con contador CSS (`counter-reset`/`counter-increment`), se endereza y cambia de color al activarse (`.step.is-active`).
- **"Recorte pegado"**: fotos con marco duro + sombra offset + rotación leve, como una foto clavada a mano.

---

## 2. Estructura completa (Intro + 5 tramos + cierre)

### Intro
Masthead con título + bajada + fondo de líneas de cancha tenues (arco + círculo + línea vertical, ~7% opacidad, en SVG). Debajo, bloque de texto sobre Chris Ford (nov. 1979, primer triple de la historia) + su foto en duotono. Termina con scroll-cue.

### Transición intro → tramo 1
**Regla de oro del proyecto, aplicada en todas las transiciones:** el elemento que conecta dos tramos debe ser el **mismo elemento del DOM**, nunca una copia — si no, se siente como "algo se fue, apareció otra cosa" en vez de una transformación continua. Acá: las líneas de cancha viven *dentro* del propio gráfico sticky de tramo 1 desde el principio (no en una sección "puente" aparte). La línea vertical no se mueve, solo gana opacidad y se convierte en el eje Y. La pelota cae con física real (rebote vía `d3.easeBounceOut`, easing distinto en X que en Y, gira mientras cae).

### Tramo 1 — "El tiro que nadie quería" (1979-2000)
Line chart D3+Scrollama, intentos de triple por partido liga completa, dibujo progresivo con el scroll. Franja sombreada 1994-97 (línea acortada). Aside de Reggie Miller (formato "nota al pie", debajo del step 03: foto duotono chica + 3 números: 94-95: 195, 95-96: 168, 96-97: 229). Fondo de madera (`maderapiso.jpg`) recortado solo al área de trazado del gráfico, muy desaturado. Último step usa clase `.reflective` (no `:last-child`, para no romperse cuando se agregan más steps después).

### Transición tramo 1 → tramo 2
La pelota real del gráfico (no una copia) se agranda mucho, revela la cancha de tramo 2 detrás (trasladada matemáticamente para que el aro caiga exacto sobre el último punto de datos), se achica y se convierte en la marca del aro.

**Bugs técnicos importantes que costó encontrar (¡tenerlos en cuenta desde el diseño!):**
1. CSS `transform` en un elemento con atributo SVG `transform` de posición → el CSS gana y borra la posición. Solución: grupo externo solo con posición (atributo), grupo interno anidado solo con el escalado (CSS).
2. Animaciones CSS `@keyframes` en loop (como el brillo pulsante) le ganan a una simple transición de opacidad — hay que matar la animación explícitamente (`animation: none`, con clase dedicada tipo `.pre-drop`).
3. Selectores CSS por **ID** pesan más que cualquier combinación de clases — si algo no se apaga a pesar de tener la clase correcta, buscar si hay una regla con `#id` compitiendo. La forma más robusta de ganar siempre: setear el estilo inline vía `d3.select(...).style(...)`.

### Tramo 2 — "Moreyball"
Shot chart con geometría real de cancha NBA (arco a 23'9", esquinas a 22 pies con transición exacta a ~14 pies, pintura 16×19, área restringida 4 pies). Escala de color papel→naranja→rojo según % de tiros por zona. Mecanismo de contraste: la zona de media distancia se apaga con opacidad **y** `grayscale`, no solo con un borde dorado (el borde solo no se notaba bien). Hook-dot en el punto más alto del arco, se vuelve "Stephen Curry" en la transición a tramo 3.

### Tramo 3 — "Curry rompe la escala"
- **Draft**: texto + foto duotono (`draftcurry.jpeg`).
- **Bar chart "antes"**: 11 jugadores (10 pre-Curry + Curry), columnas verticales, foto de cada uno **sin marco ni círculo, sin duotono** (estilo "The Pudding", solo el PNG), la foto "monta" sobre la barra seguida su altura (`bottom: barHeight + gap`, calculado en JS con píxeles, no porcentajes). Fondo blanco de las fotos removido con PIL.
- **Line chart**: Curry vs. Harden/Klay/Korver/Terry/Ray Allen, crecimiento **continuo ligado al scroll** (`scrollama.onStepProgress`, no saltos discretos entre steps), reescala de eje Y en vivo (300→420), color azul→naranja.
- **Fotos aside**: `currytriple.jpg` (temporada de 272, duotono) y `739season.jpg` (tapa del Daily News del 73-9, **sin duotono**, es un artefacto impreso real). Viven en un panel lateral fijo al costado del gráfico, no flotando sobre puntos de datos (más robusto).
- **Pie charts**: reparto de puntos (dobles/triples/libres) Bulls 1995-96 vs. Warriors 2015-16 — sección aparte **después** del line chart, no adentro de un step-card. Números reales: Bulls 63.9%/18.8%/17.3%, Warriors 51.2%/34.2%/14.5%.
- **Texto de enganche + bar chart "después"**: mismo gráfico de barras de arriba se extiende con Harden, Klay, Kemba, Lillard, Dončić, Herro, Edwards, Haliburton, Wembanyama, Holmgren (sin fotos, no las tenemos). Al agregarse, **se recalculan todas las barras juntas** contra el nuevo máximo (Lillard con 185 pasa a ser el techo).

⚠️ **Pendiente de reordenar**: el bar chart "antes"+"después" vive en `tramo3_draft_prototipo.html`; el line chart + pie charts viven en `tramo3_scalebreak_prototipo.html`. El orden real correcto es: draft → barras antes → line chart → pie charts → enganche + barras después. Hay que unificar en un solo archivo respetando ese orden.

### Transición tramo 3 → tramo 4
Conceptualizada pero **no construida todavía**: mismo patrón de "la pelota real se agranda y revela lo que sigue".

### Tramo 4 — "Hasta los pívots tiran de afuera"
Se **abandonó** el plan original del guión (area chart apilado por posición) por falta de datos reales confiables. En su lugar: **scatter de 12 pívots reconocidos**, eje X = año de debut, eje Y = promedio de triples intentados por partido en toda la carrera. Marcadores = foto circular duotono de cada jugador (no puntos genéricos).

Datos reales verificados (Basketball-Reference / StatMuse):

| Jugador | Debut | 3PA/partido |
|---|---|---|
| Kareem Abdul-Jabbar | 1969 | 0.01 |
| Hakeem Olajuwon | 1984 | 0.10 |
| Patrick Ewing | 1985 | 0.11 |
| Shaquille O'Neal | 1992 | 0.02 |
| Dirk Nowitzki | 1998 | 3.42 |
| Dwight Howard | 2004 | 0.08 |
| Brook Lopez | 2008 | 2.60 |
| Nikola Jokić | 2015 | 2.80 |
| Karl-Anthony Towns | 2015 | 4.30 |
| Joel Embiid | 2016 | 3.50 |
| Chet Holmgren | 2023 | 3.83 |
| Victor Wembanyama | 2023 | 3.90 |

Jokić/Towns (mismo año, 2015) y Wemby/Holmgren (mismo año, 2023) tienen un jitter horizontal chico para que las fotos no se superpongan.

4 steps: (1) revela el cluster "casi cero" (Kareem, Hakeem, Ewing, Shaq, Dirk, Howard) sin resaltar; (2) resalta Dirk (la excepción) y después Howard (prueba que no cambió nada todavía); (3) revela y resalta Lopez + ficha con "31 vs. 387"; (4) revela los 5 restantes, todos resaltados.

Nombres de archivo de fotos usados: `kareem.png`, `face5.png` (¡es Hakeem, nombre raro!), `ewing.png`, `shaq.png`, `dirk.png`, `howard.png`, `lopez.png`, `jokic.png`, `towns.png`, `embiid.png`, `holmgreen.png` (typo, es Holmgren), `wemby.png`.

### Transición tramo 4 → tramo 5
Conceptualizada (mismo mecanismo de "cambia de paleta cálida a fría, sin cambiar la forma" del guión original, aplicado a los puntos del scatter en vez de a un area chart) — **no construida todavía**.

### Tramo 5 — "¿Se rompió el juego?"
También se **abandonó** el plan original (slider de shot charts 1995 vs. hoy) por ser demasiado parecido al mecanismo de tramo 2. En su lugar: **line chart del True Shooting % de la liga, 1979-80 a 2025-26** (47 temporadas reales, tabla completa de Basketball-Reference). Máximo histórico: **58.1%**, empatado entre 2022-23 y 2025-26.

Citas reales verificadas (no inventadas):
- Isiah Thomas: *"If you can shoot the three, and you can make a layup, and you can make a free-throw"* (podcast Come And Talk 2 Me, oct. 2025)
- Charles Barkley: *"I don't like jump-shooting teams"* (comentarios en TNT, ~2015-16)

4 steps: (1) solo ejes, sin línea; (2) la línea se dibuja hasta ~55% vía `stroke-dasharray`/`dashoffset`, tono gris, cita de Isiah Thomas; (3) la línea se completa al 100%, todavía sin resaltar, cita de Barkley + gancho a Wembanyama; (4) la línea se resalta (naranja, más gruesa), aparece el valor final "58.1% — máximo histórico".

### Transición tramo 5 → cierre
Un 5to step invisible: la línea se estira más allá del último dato en trazo punteado gris, perdiendo opacidad ("la curva sigue, no sabemos hacia dónde"). Ejes y grilla se apagan al mismo tiempo.

### Cierre
Sección no-sticky, scroll normal. Línea de tiempo horizontal 1979-2026 con 5 hitos: Chris Ford (1979), Daryl Morey (2003), Curry 402/73-9 (2016), Brook Lopez 387 (2017), TS% 58.1% (2023). Debajo, el texto de reflexión final **tal cual estaba escrito en el guión original** (no se cambió una palabra):

> *"El triple no cambió las reglas del básquet. Cambió la pregunta que se hacen los que lo juegan: ya no es '¿puedo entrar?', sino '¿desde dónde conviene tirar?'. Esa pregunta la empezó a responder una oficina de analistas, la llevó al extremo un base de 1,88 metros, y terminó reescribiendo hasta el rol de los pívots. Cuarenta años después de aquel primer tiro casi ignorado, la única certeza es que la línea se va a seguir corriendo. La pregunta que queda es hacia dónde."*

---

## 3. Archivos prototipados hasta ahora (todos por separado, falta unificar)

| Archivo | Contenido | Assets que necesita |
|---|---|---|
| `intro_tramo1_combinado.html` | Intro + tramo 1 completo + transición + tramo 2 completo | `chrisford.webp`, `ball.png`, `reggie.jpg`, `maderapiso.jpg` |
| `tramo3_draft_prototipo.html` | Draft + bar chart antes/después | `draftcurry.jpeg`, carpeta `players/` (11 fotos) |
| `tramo3_scalebreak_prototipo.html` | Line chart Curry + pie charts | `currytriple.jpg`, `739season.jpg` |
| `tramo4_pivots_prototipo.html` | Scatter de 12 pívots | carpeta `pivots/` (12 fotos) |
| `tramo5_prototipo.html` | Line chart TS% + cierre completo | ninguno (todo SVG) |

---

## 4. Lo que falta antes de armar el archivo único final

1. Construir la transición tramo 3 → tramo 4 (conceptualizada, no hecha).
2. Construir la transición tramo 4 → tramo 5 (conceptualizada, no hecha).
3. Reordenar el contenido de tramo 3 (draft → antes → line chart → pies → después) — hoy está repartido en 2 archivos en el orden equivocado.
4. Unificar todo en un solo documento, reconciliando IDs/clases duplicadas entre archivos y ajustando coordenadas donde haga falta (mismo proceso que ya hicimos para intro+tramo1+tramo2).
5. Considerar usar **Claude Code** para esta etapa de unificación — ya lo charlamos, tiene sentido para manejar tantos archivos/carpetas de una vez que se pase de prototipar piezas sueltas a ensamblar el final.
