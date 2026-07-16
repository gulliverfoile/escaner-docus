EXPLICACIÓN DEL PROTOTIPO DE ESCÁNER DE DOCUMENTOS (HTML + JS)

¿ES PARA MÓVIL?
Sí, este código funciona en cualquier navegador moderno que soporte 
getUserMedia (Chrome, Firefox, Edge, Safari). En móviles Android o 
iPhone se accede a la cámara trasera si está disponible. La parte del 
canvas se adapta al tamaño real del vídeo usando videoWidth/videoHeight, 
así que no debería descuadrarse si el CSS no interfiere. Si has tenido 
problemas de descuadre con canvas en móvil, suele ser por diferencias 
entre el tamaño lógico (CSS) y el físico (píxeles reales). Aquí usamos 
el tamaño físico directamente y luego mostramos la imagen con 
max-width:640px sin forzar dimensiones fijas.

¿QUÉ HACE?
1. Enciende la cámara y muestra la vista previa en un elemento <video>.
2. Cuando pulsas "Capturar y Procesar", toma un fotograma del vídeo y 
   lo dibuja en un canvas oculto.
3. Ese canvas se procesa píxel a píxel para convertir la imagen a un 
   blanco/negro limpio, simulando un fondo blanco perfecto y texto negro 
   nítido (sin grises sucios ni blanco satinado).
4. Muestra el resultado en pantalla y habilita un enlace de descarga en 
   formato PNG.

¿CÓMO LO HACE?
a) Acceso a la cámara: 
   - navigator.mediaDevices.getUserMedia con restricciones de resolución 
     (ideal 1280x720). El stream se asigna al <video> y se reproduce.

b) Captura del fotograma:
   - Se crea un canvas temporal con las dimensiones exactas del vídeo 
     (video.videoWidth, video.videoHeight).
   - Se dibuja el frame actual con drawImage.

c) Procesamiento (módulo processImage):
   - Se obtienen los datos de píxeles (ImageData).
   - Se convierte cada píxel a escala de grises usando la fórmula de 
     luminosidad (0.299*R + 0.587*G + 0.114*B). Se guarda en un array 
     Uint8Array llamado "gray".
   - Se construye una "imagen integral" (sumas acumuladas) para calcular 
     rápidamente la media local en un bloque de 15x15 píxeles.
   - Para cada píxel, se calcula la media de su vecindario (blockSize=15) 
     y se usa como umbral con un pequeño offset (C=-2). Si el valor del 
     píxel es mayor que (media + C), se pinta blanco (255); si no, negro (0).
   - Este es el "umbral adaptativo": el umbral varía según la iluminación 
     local, por lo que las sombras o los bordes oscuros no fastidian 
     la binarización.
   - El resultado es una imagen con fondo totalmente blanco y texto negro 
     puro, sin medios tonos.

d) Visualización y descarga:
   - Los datos procesados se colocan en el canvas de salida (outputCanvas).
   - Se genera un data URL en PNG y se asigna a una etiqueta <img> para 
     previsualizar.
   - Se crea un enlace <a> con el atributo download para guardar el PNG.

¿POR QUÉ ESTE ENFOQUE?
- No requiere instalar nada: un solo archivo HTML ya es funcional.
- Procesa en el navegador, sin enviar imágenes a un servidor 
  (privacidad y rapidez).
- El umbral adaptativo soluciona el problema del blanco satinado y el 
  negro borroso: aunque la foto tenga zonas más oscuras o claras, las 
  letras siempre quedan bien contrastadas.
- La imagen integral acelera el cálculo, haciéndolo viable incluso en 
  móviles sin bloquear la interfaz.
- La salida simula un escaneo real: fondo blanco uniforme, texto negro 
  sólido, lista para archivar o imprimir.
- Los parámetros (blockSize=15, C=-2) son ajustables según el tipo de 
  documento. Con un pequeño cambio se puede añadir un slider en la 
  interfaz para regular la sensibilidad.

POSIBLES MEJORAS INMEDIATAS (sin sobreingeniería)
- Dibujar un recuadro guía sobre el vídeo (usando un div con borde 
  semitransparente) para ayudar a centrar el documento.
- Detectar automáticamente los bordes del papel y enderezar la imagen 
  (con OpenCV.js o un algoritmo propio de detección de contornos).
- Añadir selección de modo: Texto, Color, Foto original.
- Generar un PDF directamente con jsPDF (importado por CDN).

Este esqueleto es modular: las funciones de cámara, procesamiento y UI 
están separadas y se comunican por eventos implícitos (callback del 
botón). Migrar a una arquitectura hexagonal más formal es cuestión de 
encapsular cada bloque en clases con interfaces claras.

Si en tu móvil el canvas se descuadra, comprueba que el <video> y el 
canvas no tengan estilos CSS que modifiquen su tamaño de forma diferente 
al tamaño intrínseco. La línea `tempCanvas.width = video.videoWidth` 
asegura que el canvas interno coincida con la resolución nativa del 
vídeo. La visualización final usa max-width para adaptarse a la pantalla.
