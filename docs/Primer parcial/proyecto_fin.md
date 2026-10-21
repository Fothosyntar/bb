# Proyecto primer parcial

**Autores:** Carmen Leyva López, (escribe tu nombre)  
**Asignatura:** Introducción a la Mecatrónica  
**Periodo:** Septiembre – Octubre 2026

En la sesión 

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
Primer paso: soldar los pines del puente H y probarlo
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

// Definición de pines 
#define AIN1 26      // Dirección 1
#define AIN2 27      // Dirección 2
#define PWMA 25      // PWM (velocidad)
#define STBY 33      // Habilitación del driver

#define PWM_FREQ       1000   
#define PWM_RESOLUTION 8      

void setup() {
  // Configurar pines de control como salida
  pinMode(AIN1, OUTPUT);
  pinMode(AIN2, OUTPUT);
  pinMode(STBY, OUTPUT);
  ledcAttach(PWMA, PWM_FREQ, PWM_RESOLUTION);

  // Activar el driver (sacar del modo standby)
  digitalWrite(STBY, HIGH);
}

void loop() {
  // Giro hacia adelante 
  digitalWrite(AIN1, HIGH);
  digitalWrite(AIN2, LOW);
  ledcWrite(PWMA, 200);   
  delay(3000);

  // Detener 
  ledcWrite(PWMA, 0);
  delay(2000);

  // Giro en reversa 
  digitalWrite(AIN1, LOW);
  digitalWrite(AIN2, HIGH);
  ledcWrite(PWMA, 200);
  delay(3000);

  // Detener 
  ledcWrite(PWMA, 0);
  delay(2000);
}
```

Pines utilizados
------------------------------------------------------------
- AIN1  (GPIO 26): dirección 1
- AIN2  (GPIO 27): dirección 2
- PWMA  (GPIO 25): velocidad (señal PWM)
- STBY  (GPIO 33): encendido del driver (HIGH = activo)

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


### Evidencia visual: puente H

[![Ver video en YouTube](https://img.youtube.com/vi/h693ElG9Kuk/0.jpg)](https://www.youtube.com/watch?v=h693ElG9Kuk)

------------------------------------------------------------
Segundo paso: probar conexión vía bluetooth y buscar un joystick adecuado
------------------------------------------------------------
Código bluetooth:

```c++
#include <BTAddress.h>
#include <BTAdvertisedDevice.h>
#include <BTScan.h>
#include <BluetoothSerial.h>

BluetoothSerial SerialBT;
String buffer = "";

// Pines del TB6612FNG 
#define AIN1 26
#define AIN2 27
#define PWMA 25
#define STBY 33

// PWM 1
#define PWM_FREQ       1000
#define PWM_RESOLUTION 8

// Variables de control 
int velocidad = 200;
const int VEL_MIN = 80;
const int VEL_MAX = 255;
const int PASO_VEL = 25;

void setup() {
  Serial.begin(115200);

  // Configurar pines
  pinMode(AIN1, OUTPUT);
  pinMode(AIN2, OUTPUT);
  pinMode(STBY, OUTPUT);

  // PWM 2
  ledcAttach(PWMA, PWM_FREQ, PWM_RESOLUTION);

  // Activar driver
  digitalWrite(STBY, HIGH);

  // Motor detenido al inicio
  detenerMotor();

  // Iniciar Bluetooth
  SerialBT.begin("ESP32test");
  Serial.println("\nThe device started, now you can pair it with bluetooth!");
  Serial.println("Comandos: F=adelante, B=atras, S=parar, +=subir vel, -=bajar vel, 0-9=velocidad");
}

void loop() {
  // Leer todo lo que llega por Bluetooth y acumularlo en el buffer
  while (SerialBT.available()) {
    char c = SerialBT.read();
    Serial.print(c);          

    if (c == '\n' || c == '\r') {
      if (buffer.length() > 0) {
        procesarComando(buffer);
        buffer = "";
      }
    } else {
      buffer += c;
      if (buffer.length() > 32) buffer = "";
    }
  }

  delay(20);
}

// PROCESAR COMANDO
void procesarComando(String cmd) {
  cmd.trim();
  cmd.toUpperCase();

  if (cmd.length() == 0) return;

  // Tomamos el primer carácter como comando principal
  char c = cmd.charAt(0);

  Serial.print("\nComando recibido: ");
  Serial.println(cmd);

  switch (c) {
    case 'F':
      motorAdelante();
      break;

    case 'B':
      motorAtras();
      break;

    case 'S':
      detenerMotor();
      break;

    case '+':
      velocidad = min(velocidad + PASO_VEL, VEL_MAX);
      Serial.print("Velocidad: "); Serial.println(velocidad);
      actualizarVelocidad();
      break;

    case '-':
      velocidad = max(velocidad - PASO_VEL, VEL_MIN);
      Serial.print("Velocidad: "); Serial.println(velocidad);
      actualizarVelocidad();
      break;

    default:
      // Si el comando es un número no aplica
      if (c >= '0' && c <= '9') {
        int nivel = c - '0';
        if (nivel == 0) {
          detenerMotor();
        } else {
          velocidad = map(nivel, 1, 9, VEL_MIN, VEL_MAX);
          Serial.print("Velocidad fijada: "); Serial.println(velocidad);
          actualizarVelocidad();
        }
      } else {
        Serial.println("Comando no reconocido");
      }
      break;
  }
}

//FUNCIONES DEL MOTOR
void motorAdelante() {
  digitalWrite(AIN1, HIGH);
  digitalWrite(AIN2, LOW);
  ledcWrite(PWMA, velocidad);
  Serial.println("Motor: ADELANTE");
}

void motorAtras() {
  digitalWrite(AIN1, LOW);
  digitalWrite(AIN2, HIGH);
  ledcWrite(PWMA, velocidad);
  Serial.println("Motor: ATRAS");
}

void detenerMotor() {
  digitalWrite(AIN1, LOW);
  digitalWrite(AIN2, LOW);
  ledcWrite(PWMA, 0);
  Serial.println("Motor: DETENIDO");
}

void actualizarVelocidad() {
  if (digitalRead(AIN1) != digitalRead(AIN2)) {
    ledcWrite(PWMA, velocidad);
  }
}
```
¿Qué hace el programa?
------------------------------------------------------------
El ESP32 se conecta por Bluetooth con el celular y recibe
comandos para controlar el motor. Los comandos se guardan
en un buffer hasta que llega un salto de línea; entonces
se procesan completos. Así se evitan errores por los
saltos de línea que manda la app.

Comandos
------------------------------------------------------------
- F  -> adelante
- B  -> atrás
- S  -> detener
- (+)  -> subir velocidad
- (-)  -> bajar velocidad
- 1 a 9 -> velocidad directa
- 0  -> detener

Partes importantes
------------------------------------------------------------
```bash
String buffer = ""
```
  Es donde se guardan las letras que llegan.
```bash
if (c == '\n' || c == '\r')
```
  Detecta el fin del comando y lo procesa.
```bash
cmd.trim() y cmd.toUpperCase()
```
  Limpian el texto y lo pasan a mayúsculas.
```bash
cmd.charAt(0)
```
  Toma solo la primera letra del comando.
```bash
map(nivel, 1, 9, VEL_MIN, VEL_MAX)
```
  Convierte un número del 1 al 9 en velocidad real
  entre 80 y 255.

------------------------------------------------------------
