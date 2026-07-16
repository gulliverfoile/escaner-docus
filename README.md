EXPLICACIÓN DE LOS DOS PROGRAMAS DE DIGITALIZACIÓN DE DOCUMENTOS

Autor: Asistente de IA (con la colaboración del usuario)
Fecha: Julio 2026
Licencia: Dominio público (haz lo que quieras)

═══════════════════════════════════════════════════════════════
1. ESCÁNER DE DOCUMENTOS (Versión Profesional)
   Archivo: index.html (con manifest.json y sw.js para PWA)
═══════════════════════════════════════════════════════════════

OBJETIVO:
  Capturar una fotografía de un documento en papel y convertirla
  en una imagen digital plana, perfectamente enderezada, con el
  fondo blanco y el texto negro nítido. Ideal para sustituir a
  la fotocopiadora tradicional.

QUÉ HACE:
  - Activa la cámara trasera del móvil (o webcam en PC).
  - Muestra una vista previa en vivo con un recuadro guía A4
    que se pone verde cuando el móvil está estable.
  - Al pulsar "Capturar" o "Auto":
    * Realiza una ráfaga de 3 fotos rápidas.
    * Las alinea entre sí (opcional, modo preciso) y las fusiona
      por mediana para eliminar el fantasma del pulso.
    * Detecta las 4 esquinas del documento (con un algoritmo
      clásico de contornos).
    * Aplica una transformación de perspectiva (homografía)
      para enderezar el documento y corregir la inclinación.
    * Normaliza la iluminación (elimina sombras) y binariza
      la imagen con el umbral de Otsu (automático).
    * Aplica un filtro de mediana para quitar ruido.
    * Muestra el resultado y permite descargarlo como PNG.
  - Funciona completamente offline (PWA) tras la primera carga.
  - Si no hay cámara, entra en modo simulación con una imagen
    de prueba.

CÓMO LO HACE (componentes principales):

  A. CameraService
     - Encapsula la API getUserMedia para obtener el stream de
       la cámara.
     - Emite eventos 'frame' con cada fotograma.
     - Captura frames individuales o ráfagas de 3 fotos.

  B. ImageProcessor (dentro de un Web Worker)
     - Normaliza iluminación con una imagen integral (box blur).
     - Calcula el umbral de Otsu para la binarización.
     - Implementa un detector de esquinas basado en Sobel +
       contornos + envolvente convexa + aproximación poligonal.
     - Realiza la homografía resolviendo un sistema lineal con
       un método iterativo.
     - Fusión de frames: alineación por SAD (baja resolución)
       y mediana píxel a píxel.

  C. UIController
     - Orquesta los módulos y la interfaz de usuario.
     - Controla la vista previa, los botones y el feedback
       (flash, vibración).
     - Modos: Captura manual, Auto (Otsu + perspectiva), y
       Precisión (alineación de frames).

LIMITACIONES CONOCIDAS:

  - La detección de esquinas falla con fondos muy texturizados,
    sombras muy duras, o cuando el documento no contrasta bien
    con el fondo. No es tan robusta como una red neuronal.
  - La homografía usa interpolación por vecino más cercano
    (puede producir escalones en bordes inclinados). No usa
    interpolación bilineal o bicúbica.
  - El procesamiento en el Worker, aunque no bloquea la UI,
    puede ser lento en móviles antiguos (varios segundos).
  - No exporta a PDF multipágina ni incluye OCR para texto
    seleccionable.
  - El Service Worker cachea solo los archivos esenciales, no
    permite sincronización posterior.
  - En condiciones de baja luz, la ráfaga puede empeorar si
    el tiempo de exposición es alto (fotos movidas).

═══════════════════════════════════════════════════════════════
2. COPIADOR GEOMÉTRICO DE DOCUMENTOS
   Archivo: conversor.html
═══════════════════════════════════════════════════════════════

OBJETIVO:
  Tomar una imagen (escaneada o fotografiada) y reconstruir
  el texto usando una fuente tipográfica que imite la original,
  copiando la geometría exacta de cada letra pero redibujándola
  con una fuente digital nítida. No usa OCR ni redes neuronales;
  se basa puramente en la forma de las letras (momentos de Hu).

QUÉ HACE:
  - Carga una imagen (arrastrando, seleccionando archivo o
    usando una muestra sintética).
  - Preprocesa la imagen (normalización de iluminación y
    binarización) en un Worker.
  - Aísla cada letra o componente conectada (mancha negra).
  - Calcula los 7 momentos invariantes de Hu de cada letra.
  - Compara esos momentos con una base de datos interna de
    glifos de tres fuentes (Arial, Times New Roman, Courier New).
  - Selecciona el carácter y la fuente más cercanos (distancia
    euclidiana mínima entre vectores de Hu).
  - Reconstruye el documento sobre un lienzo blanco, dibujando
    cada letra con la fuente elegida en la misma posición y
    tamaño aproximado que la original.
  - Muestra el resultado y permite descargar como PNG.

CÓMO LO HACE (componentes principales):

  A. GlyphDatabase
     - Al iniciar la página, renderiza en un canvas cada
       carácter (a-z, A-Z, 0-9, puntuación) en cada una de las
       tres fuentes.
     - Para cada glifo, calcula sus momentos de Hu después de
       recortar el bounding box apretado.
     - Almacena los vectores de 7 números en un diccionario.

  B. HuMoments (clase estática)
     - Calcula los 7 momentos invariantes de Hu a partir de una
       imagen binaria.
     - Implementa momentos geométricos, centrales y normalizados,
       y las fórmulas algebraicas de Hu.

  C. ConnectedComponents
     - Extrae todas las componentes conexas (negras) de la imagen
       binarizada mediante BFS.
     - Devuelve cada componente recortada con su bounding box.

  D. DocumentReconstructor
     - Toma las componentes clasificadas (carácter, fuente) y
       dibuja el resultado en un canvas.

LIMITACIONES CONOCIDAS:

  - La base de datos de fuentes es muy limitada (solo 3 fuentes).
    Si la fuente original no se parece a ninguna de ellas, las
    letras no coincidirán bien y el documento reconstruido
    parecerá escrito en una fuente distinta.
  - Los momentos de Hu son invariantes a escala y rotación, pero
    no a distorsiones de perspectiva o estilos decorativos muy
    marcados. Letras manuscritas muy irregulares tendrán vectores
    de Hu dispersos y la coincidencia será baja.
  - El umbral de similitud (distancia máxima para considerar una
    coincidencia válida) es fijo (0.5) y puede necesitar ajuste
    según el tipo de documento.
  - No se procesan signos de puntuación raros ni caracteres
    acentuados de forma especial (dependen de la disponibilidad
    en la base de datos; se incluyeron vocales acentuadas en
    la generación pero la cobertura no es exhaustiva).
  - No se detectan líneas ni recuadros (solo texto). Las formas
    geométricas se perderán o aparecerán como letras erróneas.
  - El rendimiento al procesar muchos caracteres puede ser
    notable (cientos de letras implican cientos de cálculos de Hu
    y comparaciones contra la base de datos). Se ejecuta en el
    hilo principal, por lo que la interfaz puede congelarse
    brevemente en documentos muy densos.
  - No hay exportación a PDF editable ni a formatos vectoriales.
    Solo se obtiene un PNG.

═══════════════════════════════════════════════════════════════
RESUMEN COMPARATIVO
═══════════════════════════════════════════════════════════════

  | Característica         | Escáner           | Copiador Geométrico |
  |------------------------|-------------------|---------------------|
  | Entrada                | Cámara en vivo    | Archivo de imagen   |
  | Procesamiento clave    | Normalización,    | Momentos de Hu,     |
  |                        | homografía, Otsu  | matching de glifos  |
  | Salida                 | PNG plano y limpio| PNG con texto       |
  |                        |                   | reconstruido        |
  | Usa IA / redes neuron. | No                | No                  |
  | Dependencias externas  | Ninguna           | Ninguna             |
  | Modo offline           | Sí (PWA)          | Sí (único HTML)     |
  | Fidelidad tipográfica  | La original       | Aproximada (3 fuentes) |
  | Robustez ante mala luz | Buena (Otsu)      | Buena (normalización) |
  | Tiempo de proceso      | 1-3 segundos      | 2-5 segundos (varía) |

═══════════════════════════════════════════════════════════════
NOTA FINAL
═══════════════════════════════════════════════════════════════

Ambos programas están escritos en HTML, CSS y JavaScript
vanilla, sin frameworks. Utilizan APIs modernas del navegador
(Web Workers, getUserMedia, Canvas 2D) y son autocontenidos
(un solo archivo). La arquitectura está pensada para ser modular
y extensible, con separación en servicios y eventos, facilitando
su evolución futura (por ejemplo, añadir nuevos formatos de
salida, más fuentes, o módulos de IA si se desea).

Este proyecto demuestra que es posible crear herramientas de
digitalización funcionales sin depender de librerías de pago
ni de conexión a internet, usando solo el navegador y un poco
de ingenio geométrico.
