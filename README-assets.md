# Guía de imágenes — Frito Piurano

Todo lo visual se controla desde el objeto **`ASSETS`** al inicio del `<script>` del HTML.
Si un campo `img` es `null`, se usa el emoji (el juego funciona sin imágenes). Si pones una
imagen y no carga, cae automáticamente al emoji.

## Cómo apuntar a una imagen
En `img` puedes poner cualquiera de estas formas:

- `"ajo"` → se resuelve a `assets/ajo.png` (usa `base` + `ext`)
- `"ajo.webp"` → `assets/ajo.webp`
- `"img/ajo.png"` → ruta relativa tal cual
- `"https://..."` o `"data:..."` → URL absoluta tal cual

Cambia la carpeta/extensión por defecto con:
```js
ASSETS.base = 'assets/';   // carpeta
ASSETS.ext  = '.png';      // extensión por defecto
```

## Imágenes individuales (cambia el emoji por foto)

| Grupo | Claves | Dónde aparece |
|---|---|---|
| `ASSETS.ingredients` | `cerdo, ajo, cebolla, aji, chicha, yuca, limon, culantro, aceite, pimiento, menestra, pescado, caldo, fideos, lechuga` | Nivel 1 (escoger) y Nivel 3 (orden) |
| `ASSETS.pieces` | `frito, yuca, salsa, chifles` | Nivel 4 (piezas para servir) |
| `ASSETS.badges[0..3]` | 4 insignias | HUD, recompensas y pantalla final |
| `ASSETS.finalPlate` | plato final | Pantalla final |

Ejemplo:
```js
ingredients: {
  ajo: { name:'Ajo', emoji:'🧄', img:'ajo' },   // usará assets/ajo.png
  ...
}
```

## Secuencias progresivas (la imagen cambia según el avance)

En `ASSETS.sequences`, cada lista es un arreglo de imágenes en orden. Vacío = usa emoji/CSS.

| Secuencia | Cuándo avanza | Orden esperado |
|---|---|---|
| `pelar` | Nivel 2, barra de "pelar ajos" | de entero a pelado |
| `cortar` | Nivel 2, barra de "cortar cebolla" | de entero a cortado |
| `picar` | Nivel 2, barra de "picar ají" | de entero a picado |
| `mezclar` | Nivel 2, barra de "mezclar aderezo" | de suelto a mezclado |
| `cocina` | Nivel 3, según la cocción | frame 0 = crudo … último = listo |
| `plato` | **Nivel 4, al servir** | índice 0 = plato vacío; +1 por cada componente colocado |

Ejemplo de lo que pediste (el plato cambia al ir sirviendo):
```js
sequences: {
  plato: [
    'plato_preparado_1',   // plato vacío
    'plato_preparado_2',   // tras colocar el frito
    'plato_preparado_3',   // + yuca
    'plato_preparado_4',   // + salsa criolla
    'plato_preparado_5',   // + chifles (plato completo)
  ],
  cocina: ['olla_1','olla_2','olla_3','olla_4'],
}
```

Notas:
- En el Nivel 4, si `plato` tiene imágenes se usa el modo "imagen única que cambia"
  (no se sueltan emojis sobre el plato). Si está vacío, se sueltan las piezas como antes.
- El índice se calcula por **cantidad de componentes colocados** (0,1,2,3,4), así que
  conviene una imagen por estado.

## Música de la pantalla final (YouTube opcional)

Por defecto suena una melodía criolla sintetizada. Para usar una canción de YouTube,
pon el link (o el ID) en `ASSETS.music.youtube`:

```js
music: { youtube: 'https://www.youtube.com/watch?v=XXXXXXXXXXX' }
// también vale: 'https://youtu.be/XXXXXXXXXXX', '.../embed/XXXXXXXXXXX' o solo 'XXXXXXXXXXX'
```

Cómo funciona y qué tener en cuenta:
- Se usa el **reproductor oficial de YouTube (IFrame API)**, incrustado oculto. No se descarga
  ni se extrae el audio (YouTube no lo permite); por eso se reproduce dentro de su propio player.
- Suena al pulsar **"Música criolla"** en la pantalla final (los navegadores exigen un clic para
  reproducir, y ese botón lo provee). El botón de 🔊/🔇 también silencia esta música.
- Requiere conexión a internet y que **el video permita ser incrustado** (algunos lo bloquean).
- Si el entorno bloquea el iframe (por ejemplo, ciertas vistas previas con sandbox), cae
  automáticamente a la melodía sintetizada después de ~3 s. Para una prueba fiable, abre el
  archivo directamente en el navegador o súbelo a tu hosting.
- El video se repite en bucle automáticamente.

Override en tiempo de ejecución:
```html
<script>
  window.__GAME_ASSETS__ = { music: { youtube: 'https://youtu.be/XXXXXXXXXXX' } };
</script>
```

## Override sin tocar el archivo
Puedes inyectar/override la config antes de que cargue el script:
```html
<script>
  window.__GAME_ASSETS__ = {
    base: 'mis-imagenes/',
    sequences: { plato: ['plato_preparado_1','plato_preparado_2','plato_preparado_3','plato_preparado_4','plato_preparado_5'] }
  };
</script>
```

## Formato recomendado
- PNG/WebP con fondo transparente para ingredientes y piezas.
- Cuadradas (~256×256) para ingredientes; el plato puede ser rectangular y se recorta en círculo.
