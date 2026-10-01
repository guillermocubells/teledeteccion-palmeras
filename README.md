# Identificar palmeras desde el aire

Estudio de viabilidad técnica: **¿se puede identificar cada especie de palmera desde
imagen cenital, con qué fuente y con qué límite?** Ámbito: entorno urbano español,
usando exclusivamente datos públicos y gratuitos.

---

## Conclusión, primero

| Especie | Detección | Identificación de especie | Fuente mínima suficiente |
|---|---|---|---|
| *Phoenix canariensis* | Resuelta | Alta confianza | Ortofoto PNOA 25 cm |
| *Trachycarpus fortunei* | Resuelta vía LiDAR | Media-alta, apoyada en altura | PNOA-LiDAR + ortofoto 10–25 cm |
| Detección de picudo rojo | **No viable desde el aire** | — | — (ver §7) |

Tres conclusiones que ordenan todo lo demás:

1. **El satélite comercial no hace falta, y el gratuito no sirve.** En ciudad europea,
   la ortofoto aérea pública supera a cualquier satélite accesible.
2. **El discriminante es la morfología, no el espectro.** Esto invierte la intuición
   heredada de la teledetección agrícola.
3. **El LiDAR resuelve el caso difícil.** La altura y la firma vertical de la nube de
   puntos separan especies que la ortofoto no llega a resolver.

Todo lo necesario para una primera exploración es público, gratuito y descargable hoy.

---

## 1. El satélite está descartado

| Sensor | Resolución | Coste |
|---|---|---|
| Sentinel-2 (visible + NIR) | 10 m/px | Gratis |
| Sentinel-2 (red edge, SWIR) | 20 m | Gratis |
| Sentinel-1 (radar SAR) | ~5×20 m | Gratis |
| Sentinel-3 (OLCI) | 300 m | Gratis |
| Planet Labs | 3 m, diario | Pago |
| SkySat | 50 cm | Pago |
| WorldView-3 / Pléiades Neo | 30 cm | Pago, caro |

Un píxel de Sentinel-2 son 100 m². En jardín urbano **una palmera es un píxel**,
contaminado por acera, césped, tierra y coches. No hay nada que analizar.

Sentinel sirve para detectar *masas* de palmeral o cobertura perenne a escala de
paisaje. No para individuos.

**Lo único que aportaría el satélite, si alguna vez hiciera falta: series temporales.**
La identificación de especie por satélite no se hace por forma sino por **firma
espectro-temporal** — cómo cambia el brillo de cada banda a lo largo del año
(brotación, floración, senescencia, contenido de agua en hoja). Con revisita de 5 días
eso funciona para cultivos y para especies forestales templadas en clases amplias. A
nivel de individuo en jardín, no.

---

## 2. Por especie

### *Phoenix canariensis* — el caso fácil

Copa de **8–12 m**, estructura radial densa, hoja pinnada, crece aislada, sombra muy
característica. Vista cenitalmente es una estrella plumosa casi inconfundible.

El conteo automático de palmeras es, además, industria madura: las plantaciones de
palma aceitera del sudeste asiático se inventarían con redes convolucionales sobre
imagen submétrica con tasas de acierto muy altas. **El discriminante es la morfología,
no el espectro.**

| Fuente | Copa en píxeles | ¿Identificable? |
|---|---|---|
| Sentinel-2 (10 m) | ~1 px | No |
| Planet (3 m) | ~3×3 px | Detectas algo, no qué |
| SkySat (50 cm) | ~20×20 px | Sí, género *Phoenix* |
| WorldView-3 (30 cm) | ~35×35 px | Sí, corona resuelta |
| **PNOA (25 cm, aéreo)** | ~40×40 px | Sí — y gratis |

**Límite honesto:** *canariensis* vs. *dactylifera* no es fiable solo con píxeles —
mismo género, misma arquitectura. Se sesga bien con dos *priors*:

1. **Tamaño de copa** — la canariensis adulta es notablemente mayor.
2. **Contexto** — alineación en avenida, rotonda o jardín de chalet = ornamental, casi
   seguro canariensis; alineación agrícola en zona árida = dactylifera.

En jardín de clima continental la *dactylifera* es poco frecuente, así que el confusor
es de bajo impacto.

### *Trachycarpus fortunei* — el caso difícil

Copa de solo **2–3,5 m**. Un orden de magnitud menos de superficie que la canariensis.
Ahí se cae todo.

| Fuente | Copa en píxeles | ¿Sirve? |
|---|---|---|
| Sentinel-2 (10 m) | subpíxel | No, ni detección |
| Planet (3 m) | ~1 px | No |
| WorldView-3 (30 cm) | ~8×8 px | Marginal: mancha redonda |
| PNOA 25 cm | ~10×10 px | Marginal |
| **Ortofoto urbana 10 cm** | ~25×25 px | Sí, se resuelve el abanico |
| Dron (2–3 cm) | cientos | Trivial |

Con 8×8 px ves *que hay una planta redonda*. No ves la estructura radial que te
permitiría decir "palmera" y no "arbusto esférico".

**Dos agravantes propios de la especie:**

- **Vive escondida.** Es la palmera de jardín pequeño, patio y medianera. Rodeada de
  muros, setos y aleros que contaminan cada píxel del entorno. La canariensis está
  despejada en rotondas y paseos.
- **A veces está bajo dosel.** En el Tesino, norte de Italia, Galicia y cornisa
  cantábrica coloniza el sotobosque de castañares y robledales. En verano, desde
  arriba, es invisible.

---

## 3. Los dos discriminantes que sí funcionan

### Altura vía LiDAR

Restando el modelo digital del terreno al de superficie (**nDSM** = DSM − MDT) tienes
altura por objeto, sin geometría de sombras ni suposiciones:

| Especie | Altura | Copa | Forma |
|---|---|---|---|
| *Trachycarpus fortunei* | 4–10 m | 2–3 m | Cilindro alto y estrecho |
| *Chamaerops humilis* | 1,5–3 m | 2–3 m | Mata achaparrada |
| *Phoenix canariensis* | 10–18 m | 8–12 m | Columna con disco enorme |

Además, la nube de puntos de una palmera tiene **firma vertical propia**: tronco casi
sin retornos y, de golpe, un disco denso en la corona. Una frondosa reparte los retornos
por todo el volumen de copa. Se separa con reglas geométricas simples, sin modelo
entrenado.

Esto es lo que resuelve el problema del *Trachycarpus*: aunque la copa no se resuelva en
la ortofoto, la relación **altura / diámetro de copa** lo separa limpiamente de
*Chamaerops*, que es su principal confusor.

### Sombra — alternativa si falta LiDAR

Con el ángulo solar de la toma, la longitud de sombra da la altura.

- Tronco alto y esbelto + copa pequeña simétrica → *Trachycarpus*.
- Copa igual de pequeña **sin sombra de tronco** → *Chamaerops* (multicaule, contorno
  irregular).

Es medición geométrica pura y es sorprendentemente robusta.

### Ventana fenológica de invierno — solo para naturalización

La *Trachycarpus* es perennifolia dentro de bosque caducifolio. En enero el dosel está
desnudo y ella sigue verde:

- Con imagen submétrica de invierno ves los individuos de sotobosque que en agosto no
  existían.
- Con Sentinel-2 de invierno no llegas al individuo, pero sí detectas manchas de verdor
  perenne en bosque caducifolio.

Útil para vigilancia de especie invasora. No aplica al caso urbano.

---

## 4. Confusores a gestionar

| Confusor | Con qué se confunde | Cómo se separa |
|---|---|---|
| *Chamaerops humilis* | *Trachycarpus* | Multicaule: varias coronas amontonadas, contorno irregular, sin sombra de tronco |
| *Washingtonia* joven | *Trachycarpus* | Abanico también, pero acaba siendo mucho mayor; altura |
| *Cycas revoluta* | *Trachycarpus* pequeño | A esta resolución es ruido puro; altura <2 m lo descarta |
| *Phoenix dactylifera* | *P. canariensis* | Tamaño de copa + contexto |
| Frondosa de copa redonda | Cualquiera | Firma vertical LiDAR: retornos repartidos en volumen vs. disco en ápice |

La distinción básica sí es trivial desde arriba: **hoja pinnada (estrella plumosa) vs.
palmada (disco de abanico)**. Separa *Phoenix* de todo el resto.

---

## 5. El stack de datos públicos

En ciudad europea el satélite es la peor herramienta disponible. Estas fuentes gratuitas
lo superan.

**1. Ortofoto aérea.** El PNOA da 25 cm de cobertura nacional, pero varias comunidades
autónomas vuelan su propia ortofoto urbana a **5–15 cm**, y los ayuntamientos grandes
por su cuenta. A 8 cm una *Trachycarpus* son ~30×30 px y se le cuentan los segmentos del
abanico. Conviene comprobar qué vuelo autonómico o municipal cubre la zona antes de
conformarse con los 25 cm nacionales.

**2. LiDAR.** PNOA-LiDAR publica cobertura nacional en teselas de 2×2 km, formato LAZ,
gratis. Las coberturas recientes son bastante más densas que las primeras: **verificar
densidad de puntos y año de vuelo por zona antes de planificar** — es la variable que
más condiciona si el discriminante de altura va a funcionar.

**3. Imagen de calle.** Identificar desde el lateral es infinitamente más fácil: el
tronco fibroso y peludo de la *Trachycarpus* es inconfundible, y desde arriba es
invisible. **Mapillary** tiene licencia abierta y API, y es usable dentro del pipeline.
Street View **no**: sus condiciones no permiten descarga masiva ni almacenar derivados;
solo sirve para verificación manual puntual.

**4. Inventarios municipales de arbolado.** Varias ciudades publican como dato abierto
su arbolado viario y de zonas verdes, con especie, coordenadas, perímetro de tronco y
fecha de plantación.

> Ojo al uso correcto: es **arbolado público**. Su valor no es encontrar ejemplares,
> sino **excluir** los que ya están censados y disponer de un **set de entrenamiento
> etiquetado gratis** para el clasificador.

**5. Parcelario.** El Catastro publica geometría de parcela y edificación, referencia
catastral, superficie, uso y dirección postal mediante servicios INSPIRE, descargables
por municipio. Sirve para acotar el área de búsqueda antes de descargar un solo píxel de
ortofoto — es el filtro más barato del pipeline. La **titularidad no es un dato público**.

---

## 6. Lo que se complica en ciudad

No todo mejora al bajar a entorno urbano:

- **Cañón urbano** — sombras duras de fachadas, oclusión por aleros y toldos.
- **Patios interiores de manzana** — punto ciego perfecto: no se ven desde la calle y
  desde arriba, a menudo, tampoco. Justo donde más *Trachycarpus* hay.
- **Píxeles mixtos** con asfalto y cubiertas.
- **Vegetación en maceta y terraza** — falsos positivos de altura baja sobre edificación.

---

## 7. Lo que no se puede hacer desde el aire

**Detectar picudo rojo (*Rhynchophorus ferrugineus*) a tiempo.** Los síntomas visibles
en cenital —copa asimétrica, hojas caídas, ápice colapsado— aparecen cuando la palmera
ya está gravemente infestada, a menudo perdida. La detección precoz operativa se hace
con sensores acústicos, trampas de feromonas, perros detectores e inspección visual
entrenada. El multiespectral y el térmico desde dron tienen resultados prometedores en
investigación, no operativos.

El declive detectado comparando ortofotos de dos vuelos distintos sí es una señal útil
—pero es una señal tardía, no un sistema de alerta temprana.

Nota de precisión, porque se confunde constantemente: **el picudo rojo y el
*Paysandisia archon* no son la misma plaga ni atacan al mismo hospedador.** El picudo es
hospedador-específico en campo y su diana abrumadora es *Phoenix canariensis*; el
*Trachycarpus* se considera hospedador marginal o nulo. El *Paysandisia* sí ataca a
*Trachycarpus* y *Chamaerops*.

---

## Un pipeline mínimo

El orden importa más que las herramientas. Filtrar antes de descargar baja el cómputo
dos órdenes de magnitud:

```
1. Parcelario (Catastro, INSPIRE) — gratis, ligero
   └─ acotar el área de interés
2. Solo sobre ese subconjunto: descarga de ortofoto + LiDAR por teselas
3. LiDAR → nDSM (DSM − MDT) = altura por objeto
   └─ candidatos: objetos vegetales aislados de 2–20 m
4. Ortofoto sobre cada candidato → clasificación de especie
5. Cruce con inventario municipal de arbolado
   └─ NO para encontrar: para EXCLUIR lo ya censado
      y para ENTRENAR el clasificador con etiquetas reales
6. Salida: punto GPS + referencia catastral + especie + grado de confianza
```

Dos detalles que no son opcionales:

- **Emitir grado de confianza por individuo**, no solo etiqueta. Un clasificador que
  solo dice "canariensis" es inauditable.
- **Validar a mano una muestra** contra imagen de calle antes de creerse la tasa de
  acierto global.

---

## Fuentes

- [Centro de Descargas del CNIG](https://centrodedescargas.cnig.es/CentroDescargas/) —
  ortofoto PNOA, PNOA-LiDAR, modelos digitales del terreno
- [PNOA — Plan Nacional de Ortofotografía Aérea](https://pnoa.ign.es/)
- [Sede Electrónica del Catastro](https://www.sedecatastro.gob.es/) — servicios INSPIRE
  de parcelario y edificaciones
- [INE](https://www.ine.es/) — Atlas de Distribución de Renta de los Hogares, a nivel de
  sección censal
- [datos.gob.es](https://datos.gob.es/) — catálogo donde localizar los inventarios
  municipales de arbolado
- [Mapillary](https://www.mapillary.com/) — imagen de calle con licencia abierta y API

---

## Estado

Estudio de viabilidad cerrado; pipeline **no implementado**. Lo que hay aquí es el
análisis de qué fuente resuelve qué, y dónde están los límites reales — que es
exactamente la parte que suele faltar antes de empezar a descargar teselas.

Las cifras de resolución, altura y tamaño de copa son órdenes de magnitud de trabajo,
no medidas de campo propias. Verifícalas contra tu zona antes de construir sobre ellas.

## Licencia

[MIT](LICENSE).
