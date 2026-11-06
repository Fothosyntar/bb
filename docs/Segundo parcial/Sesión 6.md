# Bitácora Universitaria: Ingeniería Mecatrónica

## 5. Sesión 6: Visión y color
| Campo | Detalle |
| :--- | :--- |
| **Fecha** | 2 de octubre del 2026 |
| **Autora** | Carmen Leyva López |
| **Asignatura** | Introducción a la mecatrónica |

Se usó la cámara de la computadora para detectar un objeto de color amarillo.
El programa hace esto:

1. Prende la cámara.
2. Toma una imagen tras otra.
3. Cambia la imagen de color normal (BGR) a otro formato llamado HSV.
4. Separa solo los píxeles que son amarillos.
5. Limpia la imagen para quitar ruido.
6. Busca las figuras u objetos que quedaron.
7. Dibuja un contorno, un rectángulo, un círculo y el centro del objeto.

## ¿Por qué se usa HSV?
Porque en HSV es más fácil para separar los colores.
El tono H indica qué color es.
La saturación o S, indica qué tan fuerte es el color.
El brillo o V, indica qué tan claro u oscuro está.

------------------------------------------

La primera parte del código hace lo básico:
---------------------------------------------------

- Abre la cámara.
- Convierte la imagen a HSV.
- Define el rango del color amarillo:
  H entre 20 y 35
  S entre 120 y 255
  V entre 120 y 255
- Crea una máscara: una imagen en blanco y negro donde lo blanco
  es lo que sí es amarillo.
- Muestra tres ventanas:
    - mask: la máscara en blanco y negro.
    - original: la imagen normal de la cámara.
    - hsv: la imagen original pero solo con la parte amarilla.

Resultado:
El objeto amarillo se detectó, pero la máscara tenía ruido:
aparecían puntitos blancos que no eran parte del objeto.

---------------

Segunda parte: limpiar y dibujar formas
-----------------------------------------

El segundo código mejora el primero.

- Primero dilata: hace más grande lo blanco para unir partes y tapar huecos.
- Luego erosiona: vuelve a reducir para quitar ruido.

Esto se hace con un kernel (un sello en cruz) o ventana de 5x5 píxeles.
Al final queda una máscara más limpia.

Para cada objeto detectado dibuja:

- Contorno en verde.
- Rectángulo que lo encierra en azul.
- Círculo mínimo que lo encierra en amarillo.
- Centro del objeto en rojo, con sus coordenadas (x, y).

Las coordenadas del centro sirven para saber dónde está el objeto.

## Código: 

```py
import cv2
import numpy as np

cap = cv2.VideoCapture(0)  # prueba 0/1/2 si tienes varias cámaras
if not cap.isOpened():
    raise RuntimeError("No se pudo abrir la cámara")

# Opcional: fija resolución (si la cámara lo permite)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)

while True:
    ok, frame = cap.read()
    if not ok:
        break

    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

    low = np.array([20, 120, 120], dtype=np.uint8)
    high = np.array([35, 255, 255], dtype=np.uint8)
    filtro = cv2.inRange(hsv, low, high)

    filtrada = cv2.bitwise_and(frame, frame, mask=filtro)
    cv2.imshow("mask", filtro)
    cv2.imshow("original", frame)
    cv2.imshow("hsv", filtrada)
    if cv2.waitKey(1) & 0xFF == ord('q'):  # presiona 'q' para salir
        break

cap.release()
cv2.destroyAllWindows()

import cv2
import numpy as np

cap = cv2.VideoCapture(0)  # prueba 0/1/2 si tienes varias cámaras
if not cap.isOpened():
    raise RuntimeError("No se pudo abrir la cámara")

# Opcional: fija resolución (si la cámara lo permite)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)

kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))

while True:
    ok, frame = cap.read()
    if not ok:
        break

    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

    low = np.array([20, 120, 120], dtype=np.uint8)
    high = np.array([35, 255, 255], dtype=np.uint8)
    filtro = cv2.inRange(hsv, low, high)
    erosion = cv2.dilate(filtro, kernel, iterations=2)
    erosion = cv2.erode(erosion, kernel, iterations=2)
    filtrada = cv2.bitwise_and(frame, frame, mask=filtro)
    contours, _ = cv2.findContours(erosion, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    cv2.imshow("mask", filtro)
    cv2.imshow("erosion", erosion)
    cv2.imshow("original", frame)

    for c in contours:
        area = cv2.contourArea(c)
        if area < 200:  # filtra ruido muy pequeño
            continue

        # Dibujo del contorno:
        cv2.drawContours(filtrada, [c], -1, (0, 255, 0), 2)

        # Rectángulo axis-aligned:
        x, y, w, h = cv2.boundingRect(c)
        cv2.rectangle(filtrada, (x, y), (x + w, y + h), (255, 0, 0), 2)

        # Círculo mínimo envolvente:
        (cx, cy), rad = cv2.minEnclosingCircle(c)
        cv2.circle(filtrada, (int(cx), int(cy)), int(rad), (0, 255, 255), 2)

        # Centroides con momentos:
        M = cv2.moments(c)
        if M["m00"] != 0:
            cx_m = int(M["m10"] / M["m00"])
            cy_m = int(M["m01"] / M["m00"])
            cv2.circle(filtrada, (cx_m, cy_m), 4, (0, 0, 255), -1)
            cv2.putText(filtrada, f"({cx_m},{cy_m})", (cx_m + 6, cy_m - 6),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 255), 1)
    cv2.imshow("hsv", filtrada)
    if cv2.waitKey(1) & 0xFF == ord('q'):  # presiona 'q' para salir
        break

cap.release()
cv2.destroyAllWindows()

```

Conclusión
------------------------------------------------------------
Usar HSV es buena idea para detectar colores.
Pero después de separar el color hay que limpiar la imagen,
porque siempre aparece algo de ruido.
Con contornos y momentos se puede saber dónde está el objeto,
qué tamaño tiene y cómo dibujarlo.
Esto es la base para hacer seguimiento de objetos, contar piezas
o controlar un mecanismo con la cámara.
