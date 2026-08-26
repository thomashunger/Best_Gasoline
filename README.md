# ⛽ Gasolineras Tracker

**Panel de precios de combustible en tiempo real + histórico semanal, en dos archivos HTML autocontenidos. Sin backend, sin build, sin dependencias que instalar.**

Encuentra las gasolineras más baratas en un radio de 5 km, compáralas por los 4 combustibles más comunes, y sigue la evolución del precio más barato a lo largo de la semana.

---

## ✨ Features

- **Tabla de precios en vivo** — Gasolina 95 E5, Gasolina 98 E5, Gasóleo A y Gasóleo Premium, ordenables por columna, con la estación más barata de cada combustible resaltada.
- **Radio de 5 km desde tu ubicación** — usa la geolocalización del navegador (con tu permiso) y calcula distancias reales por Haversine; si no das permiso, cae a unas coordenadas de referencia fijas.
- **Actualización automática** — al abrir la página y cada 30 minutos mientras se mantenga abierta.
- **Cadena de fuentes con fallback en cascada** — si una fuente de datos falla, prueba la siguiente en vez de dejar la tabla vacía (ver [Fuentes de datos](#-fuentes-de-datos)).
- **Histórico semanal del precio más barato** — cada actualización registra estación, combustible, precio y hora exacta; el segundo dashboard lo dibuja como gráfico de escalones (los precios oficiales solo cambian ~1 vez al día, así que una línea "suave" mentiría sobre la tendencia).
- **Deduplicado inteligente** — no se guarda un punto nuevo en el histórico si el precio más barato no ha cambiado, para que la línea de tiempo muestre cambios reales y no ruido.
- **Almacenamiento compartido con fallback local** — si ambos archivos se abren dentro de Claude, el histórico se sincroniza entre quien los abra; si se abren como archivos sueltos, cada navegador guarda su propio histórico local automáticamente, sin errores ni pantallas en blanco.

## 📸 Cómo se ve

| Tabla de precios | Histórico semanal |
|---|---|
| Ranking ordenable, precio más barato resaltado por columna | Gráfico de escalones + tabla de registros |

## 🗂 Estructura del proyecto

```
gasolineras-app/
├── gasolineras-tabla.html       # Dashboard 1: ranking de gasolineras cercanas
├── gasolineras-historico.html   # Dashboard 2: evolución semanal del precio más barato
└── README.md
```

Cada archivo es completamente autónomo (HTML + CSS + JS en el mismo fichero). Ábrelos con doble clic, súbelos a cualquier hosting estático, o compártelos dentro de Claude para que el histórico quede sincronizado entre varias personas.

## 🚀 Uso

1. Abre `gasolineras-tabla.html`. Al cargar, intenta geolocalizarte y consulta los precios reales dentro de un radio de 5 km.
2. Ordena la tabla por el combustible que te interese, o pulsa **"Registrar precio más barato"** para forzar un punto en el histórico.
3. Abre `gasolineras-historico.html` para ver la evolución de la última semana: mínimo, registro más reciente, y el detalle de cada cambio (estación, combustible, hora).

No hace falta servidor, `npm install`, ni configuración: son dos archivos HTML.

## 🔌 Fuentes de datos

El registro oficial de precios de carburantes ([Ministerio para la Transición Ecológica](https://sedeaplicaciones.minetur.gob.es/ServiciosRESTCarburantes/PreciosCarburantes/EstacionesTerrestres/)) no envía cabeceras CORS, así que un navegador no puede consultarlo directamente. La app prueba, en orden, hasta encontrar una fuente que responda:

1. **Mirror diario** — una réplica de los mismos datos oficiales republicada vía GitHub Pages (sí envía CORS), actualizada una vez al día por Actions.
2. **API oficial vía pasarelas CORS** — varios relays públicos (allorigins, corsproxy.io, thingproxy, codetabs) que reenvían la petición con las cabeceras necesarias.
3. **Snapshot congelado** — un último conjunto de datos guardado dentro del propio archivo, usado solo si las dos fuentes anteriores fallan a la vez, para que la tabla nunca se quede vacía.

Cada actualización deja constancia de qué fuente se usó (`mirror`, `proxy` o `fallback`) junto al registro en el histórico.

## ⚠️ Limitaciones honestas

- **No hay scraping real en segundo plano.** Todo se ejecuta en el navegador de quien tenga la página abierta; si nadie la abre durante días, no se generan nuevos registros.
- **Los precios oficiales se actualizan ~1 vez al día**, no en tiempo real minuto a minuto — por eso el histórico usa deduplicado y un gráfico de escalones en vez de una curva continua.
- **El histórico compartido depende de dónde se abra el archivo.** Dentro de Claude, ambos dashboards leen el mismo almacenamiento compartido. Fuera de Claude (archivo descargado, hosting propio), cada navegador guarda su histórico en `localStorage`, sin sincronizar entre dispositivos — para eso haría falta un backend real.
- **El radio de 5 km está calibrado para Barcelona** (municipios 881/Barcelona y 875/Badalona como respaldo vía la API oficial). El mirror diario cubre todo el territorio nacional automáticamente si cambias las coordenadas.

## 🛠 Personalización

Ambos archivos son legibles de arriba a abajo. Lo más probable que quieras tocar:

- `FALLBACK_COORDS` (en `gasolineras-tabla.html`) — coordenadas de referencia si se deniega la geolocalización.
- El radio de búsqueda (`5.2` km) en los filtros de distancia.
- `dedupeWindowMs` / la ventana de 20h — cuánto tiempo debe pasar sin cambios antes de permitir un registro repetido.
- La ventana del histórico (`7*24*60*60*1000`, una semana) en `gasolineras-historico.html`.

## 🧾 Créditos

- Datos oficiales: Ministerio para la Transición Ecológica y el Reto Demográfico (MITECO).
- Mirror diario: [IvanAraque/precios-carburantes](https://github.com/IvanAraque/precios-carburantes) (GitHub Pages, actualizado vía Actions).
- Gráficos: [Chart.js](https://www.chartjs.org/).

## 📄 Licencia

Proyecto personal / DIY. Úsalo, modifícalo y compártelo libremente.
