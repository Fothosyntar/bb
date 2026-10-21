# Proyecto primer parcial

**Autora:** Carmen Leyva López  
**Asignatura:** Introducción a la Mecatrónica  
**Periodo:** Septiembre – Octubre 2026

Lista de componentes
------------------------------------------------------------
Para comenzar a trabajar en el proyecto partimos de la
siguiente lista de componentes:

- Un microcontrolador ESP32
- Dos motores DC con caja reductora
- Un puente H TB6612FNG
- Cables calibre 22
- Una protoboard mediana
- Un interruptor chico

------------------------------------------------------------
Primer paso: soldar los pines del puente H
------------------------------------------------------------
Primero soldamos los pines del puente H, porque el módulo
TB6612FNG viene sin pines y no se puede conectar a la
protoboard hasta ponerlos.

Cómo lo hicimos:
  1. Colocamos los pines en el módulo y los sujetamos para
     que no se movieran al soldar.
  2. Soldamos con cautín y estaño delgado, cuidando no unir
     dos pines con estaño de más.
  3. Revisamos que cada soldadura quedara firme.

------------------------------------------------------------
Prueba del puente H
------------------------------------------------------------
Después de soldar, probamos el puente H con un código
sencillo para confirmar que funcionaba. El código hace
girar el motor adelante, lo detiene, lo pone en reversa
y lo detiene otra vez, repitiendo el ciclo.

Código de prueba:


```c++
#include <Arduino.h>

// pines 
#define AIN1 26      // Dirección 1
#define AIN2 27      // Dirección 2
#define PWMA 25      // PWM (velocidad)
#define STBY 33      // Habilitación del driver

// PWM
#define PWM_CHANNEL   0

#define PWM_FREQ      1000   
#define PWM_RESOLUTION 8     

void setup() {
  pinMode(AIN1, OUTPUT);
  pinMode(AIN2, OUTPUT);
  pinMode(STBY, OUTPUT);

  // Configurar el canal PWM del ESP32
  ledcSetup(PWM_CHANNEL, PWM_FREQ, PWM_RESOLUTION);
  ledcAttachPin(PWMA, PWM_CHANNEL);

  // Activar el driver
  digitalWrite(STBY, HIGH);
}

void loop() {
  // adelante 
  digitalWrite(AIN1, HIGH);
  digitalWrite(AIN2, LOW);
  ledcWrite(PWM_CHANNEL, 200);   // Velocidad (0 a 255)
  delay(3000);

  // STOP
  ledcWrite(PWM_CHANNEL, 0);
  delay(2000);

  // reversa
  digitalWrite(AIN1, LOW);
  digitalWrite(AIN2, HIGH);
  ledcWrite(PWM_CHANNEL, 200);
  delay(3000);

  // STOP
  ledcWrite(PWM_CHANNEL, 0);
  delay(2000);
}
```
Hace girar un motor DC en dos sentidos (adelante y reversa),
con velocidad regulada por PWM y pausas entre movimientos.

Pines utilizados
------------------------------------------------------------
- AIN1  (GPIO 26): dirección 1
- AIN2  (GPIO 27): dirección 2
- PWMA  (GPIO 25): velocidad (señal PWM)
- STBY  (GPIO 33): encendido del driver (HIGH = activo)


Configuración del PWM
------------------------------------------------------------
El ESP32 no usa analogWrite como el Arduino UNO.
Usa la vía LEDC con canales.

- Canal:       0
- Frecuencia:  1000 Hz
- Resolución:  8 bits  (valores de 0 a 255)

ledcSetup(canal, frecuencia, resolución) configura el canal.

ledcAttachPin(pin, canal) conecta el pin físico al canal.

ledcWrite(canal, valor) escribe la velocidad (0 a 255).

setup()
------------------------------------------------------------
- Declara AIN1, AIN2 y STBY como salidas.
- Configura y conecta el canal PWM al pin 25.
- Pone STBY en HIGH para activar el driver.
  Si STBY está en LOW, el motor no gira aunque se le mande PWM.

------------------------------------------------------------
loop()
------------------------------------------------------------
Adelante:
  AIN1=HIGH, AIN2=LOW, PWM=200, espera 3000 ms.
  (200 de 255 ≈ 78 % de la velocidad máxima)

Stop:
  PWM=0, espera 2000 ms. El motor se detiene por inercia.

Reversa:
  AIN1=LOW, AIN2=HIGH, PWM=200, espera 3000 ms.
  Se invierten las direcciones y el puente H gira al revés.

Stop final:
  PWM=0, espera 2000 ms y vuelve a empezar.

------------------------------------------------------------
Observaciones
------------------------------------------------------------
- La velocidad 200 se puede cambiar por cualquier valor entre 0 y 255.
- delay() bloquea el programa; para hacer varias cosas a la vez
  hay que usar millis().
- Con PWM=0 el motor se detiene por inercia, no se frena activamente.


### Video de la prueba del puente H

[![Ver video en YouTube](https://img.youtube.com/vi/h693ElG9Kuk/0.jpg)](https://www.youtube.com/watch?v=h693ElG9Kuk)





