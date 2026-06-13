# Liberia — Quebrada Cementerio 10 jun 2026

Visor web de inspección aérea de la Quebrada Cementerio, Liberia, Costa Rica.  
**58 fotos georeferenciadas** capturadas con dron en aproximadamente 22 minutos de vuelo.

🌐 **Ver en línea:** [https://asoto59g.github.io/Inspeccion-Queb-Cementerio-Liberia/](https://asoto59g.github.io/Inspeccion-Queb-Cementerio-Liberia/)

---

## 📋 Flujo de trabajo completo

### 1. Captura con dron

- **Equipo:** Dron con cámara de resolución suficiente para fotogrametría (ej. DJI Mini / Mavic / Phantom).
- **Patrón de vuelo:** Cuadrícula (grid) sobre el cauce y riberas de la quebrada con superposición lateral y frontal mínima del **75 %** para asegurar buena cobertura para la reconstrucción fotogramétrica.
- **Altura de vuelo:** Definida según la resolución de suelo (GSD) deseada. A menor altura, mayor resolución.
- **Parámetros recomendados:**
  - Superposición frontal: 80 %
  - Superposición lateral: 75 %
  - Velocidad: automática según la app de vuelo
- **Resultado:** Carpeta de fotos JPG con datos GPS EXIF embebidos en cada imagen.

---

### 2. Georeferenciación con `fullorthorect.py`

El script `fullorthorect.py` es una herramienta en Python que lee los metadatos GPS de cada foto (latitud, longitud, altitud) almacenados en los campos EXIF y genera un archivo de puntos de control (GCP — Ground Control Points) o directamente posiciona las imágenes en el sistema de coordenadas geográfico.

**Pasos:**

1. Colocar todas las fotos del vuelo en una carpeta local (ej. `fotos/`).
2. Ejecutar el script apuntando a esa carpeta:

```bash
python fullorthorect.py --input fotos/ --output salida/
```

3. El script extrae de cada imagen:
   - Latitud y longitud (grados decimales WGS84)
   - Altitud sobre el nivel del mar
   - Orientación (yaw/pitch/roll si están disponibles)
4. Genera un archivo de georeferenciación (`.csv` o `.txt`) compatible con software de fotogrametría como **ODM (OpenDroneMap)**, **Agisoft Metashape** o **QGIS**.

**Resultado:** Archivo de puntos de control que vincula cada foto a su posición geográfica real.

---

### 3. Generación del ortofotos / mosaico

Con las fotos georeferenciadas se puede usar **OpenDroneMap (WebODM)**, **Agisoft Metashape** u otro software de fotogrametría para:

1. Reconstruir la nube de puntos 3D.
2. Generar el **modelo digital de superficie (DSM)**.
3. Exportar el **ortomosaico** en formato GeoTIFF (`.tif`) con proyección geográfica.

**Parámetros de exportación recomendados:**
- Sistema de Referencia: **WGS84 / EPSG:4326** o UTM local
- Formato: **GeoTIFF (`.tif`)**
- Compresión: LZW o DEFLATE para reducir tamaño

---

### 4. Generación de tiles XYZ con QGIS

Los tiles XYZ son pequeñas imágenes cuadradas (256×256 px) organizadas en carpetas por nivel de zoom que los navegadores web pueden cargar de forma eficiente.

**Pasos en QGIS:**

1. Abrir QGIS y cargar el archivo **GeoTIFF** del ortomosaico.
2. Ir al menú: **Procesamiento → Caja de herramientas → Herramientas de raster → Generar teselas XYZ (directorio)**.
3. Configurar los parámetros:
   - **Capa de entrada:** El ortomosaico GeoTIFF
   - **Zoom mínimo:** 10
   - **Zoom máximo:** 22 *(mayor zoom = más detalle pero más archivos)*
   - **Formato de salida:** JPG o PNG
     > ⚠️ **Importante:** Si el ortomosaico tiene bordes transparentes (sin datos), usar **PNG** para conservar la transparencia. Si se usa JPG, los bordes vacíos se rellenan con color sólido.
   - **Directorio de salida:** Carpeta `tiles/` dentro del proyecto
4. Ejecutar el proceso. Dependiendo del tamaño del ortomosaico y el zoom máximo, puede tardar varios minutos.

**Resultado:** Carpeta `tiles/` con la siguiente estructura:

```
tiles/
  10/
    x/
      y.jpg
  11/
    x/
      y.jpg
  ...
  22/
    x/
      y.jpg
```

> ℹ️ QGIS genera las imágenes con el sistema de coordenadas XYZ estándar (coordenada Y no invertida), compatible directamente con Leaflet.

---

### 5. Publicación como visor web con Leaflet

El archivo `index.html` de este repositorio contiene un visor de mapas construido con **[Leaflet.js](https://leafletjs.com/)** que:

- Carga los tiles desde la carpeta `tiles/{z}/{x}/{y}.jpg` de forma nativa y eficiente.
- Ofrece dos mapas base intercambiables: **OSM Standard** y **Google Hybrid**.
- Muestra la capa de la quebrada desde el archivo `Quebpanteon.geojson`.
- Incluye controles de **zoom**, **medición** y **geolocalización del usuario**.
- Tiene título y subtítulo descriptivos del vuelo.
- Incluye meta tags de **Open Graph** para que al compartir el enlace en WhatsApp, Facebook y Twitter se muestre la imagen de previsualización (`preview.jpg`).

---

### 6. Publicación en GitHub Pages

1. Subir al repositorio los archivos:
   - `index.html`
   - `Quebpanteon.geojson`
   - `preview.jpg`
   - Carpeta `tiles/` completa
2. En la configuración del repositorio en GitHub: **Settings → Pages → Branch: main → / (root) → Save**.
3. En pocos minutos la página estará disponible en:  
   `https://<usuario>.github.io/<nombre-repositorio>/`

> ⚠️ GitHub rechaza archivos individuales mayores de **100 MB**. Por eso se usa la carpeta `tiles/` (muchos archivos pequeños) en lugar del archivo `.mbtiles` (un solo archivo grande).

---

## 🗂️ Archivos del repositorio

| Archivo | Descripción |
|---|---|
| `index.html` | Visor web interactivo con Leaflet.js |
| `Quebpanteon.geojson` | Trazo vectorial de la quebrada |
| `preview.jpg` | Imagen de previsualización para redes sociales |
| `tiles/` | Tiles XYZ generados desde QGIS (zoom 10–22) |

---

## 🛠️ Tecnologías utilizadas

- [Leaflet.js 1.9.4](https://leafletjs.com/) — Visor de mapas web
- [leaflet-measure](https://github.com/ljagis/leaflet-measure) — Control de medición
- [Leaflet.Locate](https://github.com/domoritz/leaflet-locatecontrol) — Control de geolocalización
- [OpenStreetMap](https://www.openstreetmap.org/) — Mapa base estándar
- Google Maps Hybrid — Mapa base satelital
- [QGIS](https://qgis.org/) — Generación de tiles XYZ
- [GitHub Pages](https://pages.github.com/) — Hosting web gratuito
