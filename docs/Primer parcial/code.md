# Bitácora Universitaria: Ingeniería Mecatrónica


**Autora:** Carmen Leyva López  
**Asignatura:** Introducción a la Mecatrónica  
**Periodo:** Agosto – Septiembre 2026


---


## Resumen


> **¡Saludos! Soy Carmen.**
> Decidí estudiar Ingeniería Mecatrónica por mi fascinación por los autómatas y los pequeños mecanismos que dan vida a objetos cotidianos, como los que encuentras dentro de una cafetera.
>
> Mi intención con esta bitácora es documentar los avances, retos y aprendizajes obtenidos a lo largo de la carrera.


![Foto de Carmen](../recursos/imgs/me.jpg)


---


## 0. Inicialización del repositorio y publicación en GitHub


Para registrar la bitácora en Git por primera vez y vincularla con GitHub desde Visual Studio Code (`Terminal > Nueva terminal`), se llevan a cabo los siguientes pasos:


---


### 0.1 Inicializar el repositorio local


```bash
git clone
```


---


### 0.2 Descargar la carpeta de archivos


```bash
ls
```


---
### 0.2.1 Abrir la carpeta
Preparar el código en visual estudio code, en formato md. 


### 0.3 Preparar todos los archivos


```bash
git add .
```


---


### 0.4 Guardar el primer commit


```bash
git commit -m "Primera versión de la bitácora universitaria"
```


---


### 0.5 Subir los archivos por primera vez a GitHub


```bash
git push
```


---


> **Nota:**  
> Antes de ejecutar el paso 5, asegúrate de haber creado el repositorio remoto en GitHub y de haberlo vinculado con:
>
> ```bash
> git remote add origin https://github.com/tu-usuario/tu-repositorio.git
> ```


## 1. Sesión 2: Intermitencia y Capacitores


| Campo | Detalle |
| :--- | :--- |
| **Fecha** | 28 de agosto del 2026 |
| **Autora** | Carmen Leyva López |
| **Asignatura** | Introducción a la mecatrónica |


### 1.1 Descripción


Durante esta sesión, ensamblamos el circuito basándonos en el esquema de conexión del LM3909. Evaluamos cómo la variación en voltaje afecta directamente el tiempo de parpadeo y, por consiguiente, la intensidad de luz del led.


Usamos el osciloscopio para registrar el voltaje y el parpadeo del LED. La amplitud de la onda indica el voltaje aplicado, mientras que la distancia horizontal entre pulsos (periodo) permite calcular la frecuencia de parpadeo. El brillo del LED no se lee directamente en el osciloscopio, pero se puede inferir a partir de la corriente y del ciclo de trabajo: a mayor ciclo de trabajo o corriente, mayor intensidad luminosa.


Referencia visual (intervalos cortos):


![Osciloscopio](../recursos/imgs/osil.png)


### 1.2 Evidencia del ensamble


![Montaje en protoboard](../recursos/imgs/led1.jpg)
![LED parpadeando](../recursos/imgs/led2.jpg)


*Figura 1. Montaje en protoboard y LED parpadeando.*


---


## 2. Sesión 3: Conexión inalámbrica y microcontrolador ESP32


| Campo | Detalle |
| :--- | :--- |
| **Fecha** | 4 de septiembre del 2026 |
| **Autora** | Carmen Leyva López |
| **Asignatura** | Introducción a la mecatrónica |

En está práctica aprendimos a usar el micro controlador ESP32 y a conectarlo vía bluetooth al led.

### 2.1 Observaciones


- El nombre del dispositivo Bluetooth se define en `Mi_tel.begin("Edu")`; puede cambiarse por el que se prefiera.
- Es necesario instalar en el celular una aplicación de terminal Bluetooth (como *Serial Bluetooth Terminal en android*) para enviar los comandos "ON" y "OFF".
- Se recomienda verificar la conexión física del LED: si se conecta directamente al pin 32 y a tierra.

### 2.2 Código utilizado


```c++
#include <BTAddress.h>
#include <BTAdvertisedDevice.h>
#include <BTScan.h>
#include <BluetoothSerial.h>


BluetoothSerial Mi_tel;


void setup() {
  Mi_tel.begin("Edu");
  Mi_tel.setTimeout(20);
  Serial.begin(9600);
  pinMode(32,OUTPUT);
}


void loop() {
  if(Mi_tel.available()){
    String mensaje= Mi_tel.readStringUntil('\n');
    mensaje.trim();
    if(mensaje == "OFF"){
      digitalWrite(32,1);
    }
    if(mensaje == "ON"){
      digitalWrite(32,0);
    }
  }
}
```


### 2.4 Evidencia audiovisual


[Video 1: Demostración de conexión intercalando leds](https://www.youtube.com/embed/1JN5oAWgr-M)


### 2.5 Evidencia fotográfica


![Montaje en protoboard](../recursos/imgs/blo.jpg)


*Figura 2. Montaje en protoboard*


---


## 3. Sesión 4: Motor DC y Servo


| Campo | Detalle |
| :--- | :--- |
| **Fecha** | 11 de septiembre del 2026 |
| **Autora** | Carmen Leyva López |
| **Asignatura** | Introducción a la mecatrónica |


### 3.1 Descripción


En esta sesión programamos el control de un robot con dos motores DC y un servomotor usando Arduino. Los motores DC se manejan a través de un puente H (L293D/L298N), usando los pines 6 y 7 para el primer motor y los pines 3 y 4 para el segundo. Los pines 5 y 2 se configuran como habilitadores y se colocan en `HIGH` para permitir el giro de los motores. El servomotor se conecta al pin 9 y se controla con la librería `Servo.h`.


Definimos cuatro funciones (`adelante()`, `atras()`, `der()` e `izq()`) que establecen los estados de los pines para que el robot se desplace en cada dirección. Observamos que los motores DC consumen mucha corriente, por lo que requieren una fuente externa y un puente H para invertir su rotación sin dañar el microcontrolador. También comprobamos que los nombres de las funciones `der()` e `izq()` dependen del montaje físico de los motores, por lo que conviene verificarlos antes de usarlos.


### 3.2 Código utilizado


```c++
// Incluye la librería para controlar servomotores
#include <Servo.h>


// Crea un objeto servo llamado "eus"
Servo eus;


// Función para avanzar: ambos motores giran hacia adelante
void adelante() {
  digitalWrite(6, HIGH);
  digitalWrite(7, LOW);
  digitalWrite(3, HIGH);
  digitalWrite(4, LOW);
}


// Función para retroceder: ambos motores giran hacia atrás
void atras() {
  digitalWrite(7, HIGH);
  digitalWrite(6, LOW);
  digitalWrite(4, HIGH);
  digitalWrite(3, LOW);
}


// Función para girar a la derecha
void der() {
  digitalWrite(6, HIGH);
  digitalWrite(7, LOW);
  digitalWrite(4, HIGH);
  digitalWrite(3, LOW);
}


// Función para girar a la izquierda
void izq() {
  digitalWrite(7, HIGH);
  digitalWrite(6, LOW);
  digitalWrite(3, HIGH);
  digitalWrite(4, LOW);
}


void setup() {
  // El servo está conectado al pin 9
  eus.attach(9);


  // --- Motor 1 ---
  pinMode(6, OUTPUT);    // out1
  pinMode(7, OUTPUT);    // out2
  pinMode(5, OUTPUT);    // enable del motor 1
  digitalWrite(5, HIGH); // habilita el motor 1


  // --- Motor 2 ---
  pinMode(4, OUTPUT);    // out1
  pinMode(3, OUTPUT);    // out2
  pinMode(2, OUTPUT);    // enable del motor 2
  digitalWrite(2, HIGH); // habilita el motor 2
}


void loop() {
  // Mueve el servo a 0°, espera 1 segundo
  eus.write(0);
  delay(1000);


  // Mueve el servo a 90°, espera 1 segundo
  eus.write(90);
  delay(1000);


  // Mueve el servo a 180°, espera 1 segundo
  eus.write(180);
  delay(1000);
}
```


### 3.3 Evidencia fotográfica


![Montaje del motor DC en protoboard](../recursos/imgs/DC.jpg)
*Figura 3. Aquí probamos el puente L293D y como consumia corriente con el osciloscopio*


![Montaje completo](../recursos/imgs/circuito.png)
*Figura 4. Ensamble completo de los dos motores DC y un servo.*

Al final de probar los motores DC, también se añadió un servomotor. 


## 4. Sesión 5: Mecanismos


| Campo | Detalle |
| :--- | :--- |
| **Fecha** | 18 de septiembre del 2026 |
| **Autora** | Carmen Leyva López |
| **Asignatura** | Introducción a la mecatrónica |


### 4.1 Descripción


Los motores DC giran rápido y con poca fuerza. Por eso los mecanismos permiten adaptar ese movimiento: sacrifican velocidad para ganar fuerza, o cambian el tipo de movimiento.


### 4.2 Conceptos clave


- **Palancas:** la ventaja mecánica depende de los brazos, más longitud = más fuerza. 
- **Biela-manivela:** convierte rotación en traslación (y viceversa). La velocidad del extremo no es constante.


Fórmula:


![Formula](../recursos/imgs/fórmula.png)

La fórmula solo te dice que el tamaño, la velocidad y la fuerza están conectados.

- Si pones un engranaje grande para que las cosas giren lento, tendrás mucha fuerza.

- Si pones un engranaje pequeño para que giren rápido, tendrás poca fuerza.


### 4.3 Taller


Recorrimos estaciones con mecanismos impresos en 3D. En cada una movimos el mecanismo a mano, observamos y llenamos la ficha.


### Tabla:
| Estación | ¿Qué hace? | Relación *i* (más o menos) | ¿Se puede regresar o se traba? | ¿Dónde lo ves en la vida real? | ¿Para qué te sirve en el carro o en un proyecto? |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **A · Diferencial** | Hace que las ruedas de un mismo eje giren a velocidades distintas al dar vuelta. | No es fija (1:1 en línea recta). | Se puede regresar (si es abierto). Si tiene bloqueo, se traba. | En el eje trasero de los coches. | En el coche, para que no patinen las llantas al girar. |
| **B · Cicloidal** | Baja muchísimo las revoluciones y te mucha fuerza. Es muy compacto. | Alta (de 1:10 para arriba). | Se traba (no se puede regresar). | En brazos de robots industriales o máquinas CNC. | Para un brazo robótico o una base que gire con precisión. |
| **C · Cardán** | Pasa el giro entre dos ejes que no están alineados (hay un ángulo entre ellos). | Casi 1:1, pero no va parejo (da tirones). | Se puede regresar. | En el tubo de transmisión de camiones o autos con tracción trasera. | Para conectar el motor a las ruedas si están en ejes separados. |
| **D · Obturador** | Abre y cierra el paso de algo (como la luz) de forma intermitente. | No aplica (se mide por tiempo). | Se puede abrir y cerrar. | En las cámaras de fotos. | Para un sistema de frenos ABS o para luces intermitentes. |
| **E1 · Engranes rectos** | Pasa velocidad y fuerza entre ejes que van en paralelo. | Ej: 20 dientes a 40 1:2. | Se puede regresar. | En taladros, juguetes. | Para pasar el movimiento del motor a la rueda de forma simple. |
| **E2 · Tornillo sinfín** | Hace un giro de 90° y reduce  en un solo paso. | Ej: 1 entrada a 40 dientes 1:40. | Se traba (no se puede regresar). | En grúas o sillas de ruedas. | Para un sistema de elevación que no se caiga solo cuando apagues el motor. |


### 4.4 Evidencia 



## A · Diferencial

![diferencial](../recursos/imgs/funcion-diferencial-de-un-auto.jpg)
Nota. Adaptado [o Reutilizado] de Conoce tu vehículo: función del diferencial de un auto [Fotografía], por Derco Chile, s.f., Derco (https://www.derco.cl/noticias/conoce-tu-vehiculo-funcion-del-diferencial-de-un-auto). En el dominio público [o Derechos de autor [Año] por Derco].

## Estación B · Reductor cicloidal
![reductor](../recursos/imgs/reductor_c.jpeg)

## Estación C · Junta cardán (universal)
![Junta cardan](../recursos/imgs/j_cardan.jpeg)

## Estación D · Obturador de láminas (leaf shutter)
![obturador](../recursos/imgs/obturados_l.jpeg)

## Estación E1 · Transformar el movimiento
![t_mov](../recursos/imgs/trans.jpeg)
![t_mov](../recursos/imgs/spiral.jpeg)

## Estación E2 · Redirigir y reducir
![t2_mov](../recursos/imgs/bevel.jpeg)
![t2_mov](../recursos/imgs/worm_e.jpeg)


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
