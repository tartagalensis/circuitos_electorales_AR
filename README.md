# circuitos_electorales_AR

[![Licencia: CC BY 4.0](https://img.shields.io/badge/Licencia-CC%20BY%204.0-lightgrey.svg)](LICENSE)

Cartografía de los **circuitos electorales de la República Argentina** en formato
GeoJSON, para los 24 distritos y dos cortes temporales: **2021** y **2025**.

El circuito electoral es la unidad territorial más chica con la que se publican
resultados en Argentina, pero no existe una cartografía oficial única ni completa:
cada provincia publica en formatos distintos y algunas no publican nada. Este
repositorio reúne, homogeneiza y —donde hizo falta— reconstruye esa capa, para que
se pueda mapear y analizar el voto a nivel de circuito sin rearmar el dato cada vez.

## Contenido

```
2021/                24 archivos GeoJSON, uno por distrito
  data/              fuentes originales (KML, JSON) y establecimientos de votación 2021
2025/                24 archivos GeoJSON, uno por distrito
conteo_circuitos.ipynb   recorre ambos años y arma las tablas de este README
```

Los nombres de archivo son idénticos en los dos años, así que comparar cortes es
cambiar la carpeta y nada más:

```python
gpd.read_file("2021/Tucuman.geojson")
gpd.read_file("2025/Tucuman.geojson")
```

## Campos

Los 48 archivos comparten exactamente el mismo esquema, en este orden:

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `circuito` | string | Código de circuito, 5 caracteres con ceros a la izquierda. Puede terminar en letra (`0388A`, `00047A`). |
| `codprov` | string | Código de distrito, 2 dígitos (`01`–`24`), según la tabla de abajo. |
| `coddepto` | string | Código de departamento o partido dentro del distrito, 3 dígitos. |

Sistema de referencia: **EPSG:4326** (CRS84, latitud/longitud en grados decimales).
Geometrías: `Polygon`, `MultiPolygon` y, en San Juan, algunas `GeometryCollection`.

Un mismo circuito puede aparecer en más de un *feature* cuando su territorio no es
contiguo, así que la cantidad de features es mayor o igual a la cantidad de circuitos.
Para contar circuitos usá siempre valores únicos de `circuito`.

## Circuitos por distrito

Circuitos únicos por año, excluyendo los marcadores sin código (ver
[Limitaciones conocidas](#limitaciones-conocidas)). La columna **Comunes** es cuántos
códigos de 2021 siguen existiendo en 2025, un indicador de cuán comparables son los
dos cortes en ese distrito.

| Código | Distrito | Archivo | 2021 | 2025 | Δ | Comunes |
| :---: | --- | --- | ---: | ---: | ---: | ---: |
| 01 | Ciudad Autónoma de Buenos Aires | `CABA` | 167 | 167 | 0 | 167 |
| 02 | Buenos Aires | `PBA` | 1066 | 1145 | +79 | 1057 |
| 03 | Catamarca | `Catamarca` | 155 | 156 | +1 | 155 |
| 04 | Córdoba | `Cordoba` | 622 | 634 | +12 | 621 |
| 05 | Corrientes | `Corrientes` | 170 | 174 | +4 | 170 |
| 06 | Chaco | `Chaco` | 153 | 153 | 0 | 153 |
| 07 | Chubut | `Chubut` | 115 | 121 | +6 | 115 |
| 08 | Entre Ríos | `EntreRios` | 318 | 318 | 0 | 318 |
| 09 | Formosa | `Formosa` | 107 | 150 | +43 | 106 |
| 10 | Jujuy | `Jujuy` | 146 | 146 | 0 | 146 |
| 11 | La Pampa | `LaPampa` | 89 | 89 | 0 | 89 |
| 12 | La Rioja | `LaRioja` | 146 | 146 | 0 | 145 |
| 13 | Mendoza | `Mendoza` | 192 | 192 | 0 | 192 |
| 14 | Misiones | `Misiones` | 89 | 109 | +20 | 89 |
| 15 | Neuquén | `Neuquen` | 156 | 172 | +16 | 156 |
| 16 | Río Negro | `RioNegro` | 151 | 158 | +7 | 149 |
| 17 | Salta | `Salta` | 315 | 315 | 0 | 315 |
| 18 | San Juan | `SanJuan` | 146 | 143 | −3 | 143 |
| 19 | San Luis | `SanLuis` | 136 | 169 | +33 | 129 |
| 20 | Santa Cruz | `SantaCruz` | 33 | 30 | −3 | 21 |
| 21 | Santa Fe | `SantaFe` | 525 | 558 | +33 | 43 ⚠️ |
| 22 | Santiago del Estero | `Santiago` | 257 | 257 | 0 | 257 |
| 23 | Tucumán | `Tucuman` | 274 | 282 | +8 | 274 |
| 24 | Tierra del Fuego | `TdF` | 25 | 63 | +38 | 15 |
| | **Total** | | **5553** | **5847** | **+294** | |

## Cómo usarlo

**Python (GeoPandas)**

```python
import geopandas as gpd

base = "https://raw.githubusercontent.com/tartagalensis/circuitos_electorales_AR/main"
gdf = gpd.read_file(f"{base}/2025/Tucuman.geojson")

print(gdf.shape)               # (287, 4)
print(gdf.circuito.nunique())  # 282
```

**R (sf)**

```r
library(sf)

base <- "https://raw.githubusercontent.com/tartagalensis/circuitos_electorales_AR/main"
gdf <- st_read(file.path(base, "2025", "Tucuman.geojson"))
```

**Todo el país en un solo objeto**

```python
import glob
import pandas as pd
import geopandas as gpd

pais = gpd.GeoDataFrame(
    pd.concat([gpd.read_file(f) for f in sorted(glob.glob("2025/*.geojson"))],
              ignore_index=True),
    crs="EPSG:4326",
)
```

**Para unir con resultados electorales**, armá la clave con los tres campos, no solo
con `circuito`: los códigos de circuito se repiten entre distritos.

```python
gdf["clave"] = gdf.codprov + gdf.coddepto + gdf.circuito
```

## Limitaciones conocidas

Son características del dato de origen, no errores de procesamiento. Se documentan
en lugar de corregirse en silencio.

- **Santa Fe 2025 usa otra convención de códigos.** El código 2025 equivale al de 2021
  multiplicado por diez (`00001` → `00010`), más 194 subcircuitos intermedios como
  `00115` o `00142`. Los dos años **no se unen directamente** en este distrito.
- **`coddepto` nulo en 29 features de 2021**: Neuquén (13), Salta (7), San Juan (3),
  Santiago del Estero (3), La Rioja (2) y Corrientes (1). El `codprov` de esos features
  sí está completo.
- **Polígonos sin circuito asignado.** Córdoba trae, en ambos años, cuatro polígonos con
  `circuito` igual a `sindatos`, `zonagris`, `zonagris/` o `zonaincon`; Formosa y Santa
  Cruz 2025 traen features con `S/D`. Cubren territorio real sin circuito asignado en la
  fuente y se conservan para no dejar huecos en el mapa. Filtralos con
  `gdf[gdf.circuito.str[0].str.isdigit()]`.
- **Precisión variable.** Las provincias reconstruidas tienen un margen de error mayor
  que las que publican cartografía oficial. Ver [Circuitos reconstruidos](#circuitos-reconstruidos).
- Los archivos de `2021/data/` son las fuentes crudas y **no** están homogeneizados.

## Circuitos reconstruidos

Varias provincias no publican la cartografía de sus circuitos, o la publican incompleta.
En esos casos los polígonos fueron dibujados a partir de otras fuentes (radios censales,
domicilios de establecimientos de votación, mapas en PDF, cartografía municipal) y tienen
un margen de error mayor.

> Sección en preparación: queda pendiente documentar, provincia por provincia, qué fuente
> se usó, con qué método se dibujó cada circuito y qué grado de confianza tiene el
> resultado. Los saltos de cobertura entre 2021 y 2025 en Formosa (+43), Tierra del Fuego
> (+38), San Luis (+33), Misiones (+20) y Neuquén (+16) corresponden en buena medida a
> este trabajo de reconstrucción.

## Fuentes

- [Mapa electoral — Dirección Nacional Electoral](https://mapa2.electoral.gov.ar/descargas/)
- [Datos electorales — Ministerio del Interior](https://www.argentina.gob.ar/interior/dine/datoselectorales/secciones)
- Mapa de establecimientos de votación (ver `2021/data/`)
- Organismos electorales y de cartografía provinciales, según la provincia

Los archivos originales sin procesar de 2025 no se versionan en este repositorio;
se descargan de las fuentes de arriba.

## Cómo citar

Si usás estos datos en un trabajo, citá el repositorio. GitHub ofrece la cita
autogenerada desde el botón **Cite this repository**, a partir de
[`CITATION.cff`](CITATION.cff).

**APA**

> Galeano, F. (2026). *circuitos_electorales_AR: circuitos electorales de la República
> Argentina* (Versión 2025.1) [Conjunto de datos]. GitHub.
> https://github.com/tartagalensis/circuitos_electorales_AR

**BibTeX**

```bibtex
@misc{galeano_circuitos_2026,
  author       = {Galeano, Franco},
  title        = {circuitos\_electorales\_AR: circuitos electorales de la
                  Rep\'ublica Argentina},
  year         = {2026},
  version      = {2025.1},
  howpublished = {\url{https://github.com/tartagalensis/circuitos_electorales_AR}},
  note         = {Conjunto de datos}
}
```

## Licencia

[Creative Commons Atribución 4.0 Internacional (CC BY 4.0)](LICENSE). Podés copiar,
redistribuir, adaptar y usar estos datos con cualquier fin, incluso comercial, siempre
que des el crédito correspondiente.

Las fuentes oficiales de las que derivan estos datos tienen sus propios términos de uso.

## Contribuir

Si detectás un circuito mal dibujado, un código equivocado o tenés cartografía oficial
de alguna provincia reconstruida, abrí un *issue* indicando distrito, año y circuito.
