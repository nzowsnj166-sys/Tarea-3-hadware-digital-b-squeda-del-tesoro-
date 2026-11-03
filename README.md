# Tarea-3-hadware-digital-b-squeda-del-tesoro-
Es un juego en tiempo real donde un Arduino lee tus movimientos físicos (controles y botón) y se comunica constantemente con un segundo Arduino que procesa las reglas del juego, calcula qué tan cerca estás del objetivo y controla el tiempo límite.

\# 🏴‍☠️ Proyecto: Búsqueda del Tesoro en Dos Arduino UNO R3 \*\*Asignatura:\*\* CIN-212 Hardware Digital \*\*Institución:\*\* Universidad de Valparaíso \*\*Plataforma de Simulación:\*\* Tinkercad Circuits ---

## 📌 1\. Descripción del Proyecto 
El objetivo de este proyecto es implementar un sistema embebido interactivo de \*\*Búsqueda del Tesoro\*\* en tiempo real sobre una grilla virtual de $100 \\times 100$ posiciones.

El sistema utiliza una arquitectura distribuida compuesta por dos microcontroladores \*\*Arduino UNO R3\*\* interconectados mediante una línea de comunicación serial no bloqueante (\`SoftwareSerial.h\` a 9600 baudios): 

1\. \*\*Arduino 1 (Esclavo / Joystick):\*\* Lee dos potenciómetros para las coordenadas $(X, Y)$, procesa el botón de inicio con antirrebote (\*debouncing\*) y muestra la proximidad al tesoro mediante dos LEDs indicadores PWM.

2\. \*\*Arduino 2 (Maestro / Motor del Juego):\*\* Ejecuta la máquina de estados principales, genera la ubicación aleatoria del tesoro, calcula las distancias de proximidad y gestiona un temporizador decreciente de 60 segundos. --- ## 


🛠️ 2\. Componentes y Conexiones del Sistema 
### Lista de Materiales:
\* 2 Microcontroladores \*\*Arduino UNO R3\*\* (Maestro y Esclavo).

\* 2 Potenciómetros lineales de $10\\text{ k}\\Omega$ (Ejes X e Y). 

\* 1 Pulsador de inicio (Configurado con \`INPUT\_PULLUP\` interno).

\* 3 LEDs (2 para Proximidad X/Y en el Esclavo, 1 para Temporizador en el Maestro).

\* Resistencias limitadoras de $220\\,\\Omega$ para los LEDs. 

\* Cables de conexión sobre protoboard en Tinkercad.

<img width="765" height="1024" alt="image" src="https://github.com/user-attachments/assets/1f18262b-3759-4cc3-b025-152f65aaefa8" />

A continuación se presenta el esquema visual de arquitectura de interconexión física entre ambos microcontroladores y la asignación de pines por módulo:
<img width="1360" height="542" alt="tarea 3 hadware" src="https://github.com/user-attachments/assets/01ffc827-26c9-4444-8857-38691ffa77b4" />

 en el codigo del arduino 1 

 5. Resumen de lo que definimos
Elemento	Configuración
Placa	Arduino UNO R3
Función	Esclavo
Biblioteca	SoftwareSerial
RX	D10
TX	D11
Velocidad	9600 baudios
Mensaje	HOLA MAESTRO
Frecuencia	Cada 1 segundo

codigo en c++
#include <SoftwareSerial.h>

SoftwareSerial comunicacion(10, 11);

void setup() {
  Serial.begin(9600);
  comunicacion.begin(9600);

  Serial.println("ESCLAVO LISTO");
}

void loop() {
  comunicacion.println("HOLA MAESTRO");
  Serial.println("MENSAJE ENVIADO");
  delay(1000);
}

en el codigo del arduino 2 maestro 

Resumen del código Arduino 2 — Maestro

Definimos un programa para que el Arduino Maestro pueda recibir mensajes enviados por el Arduino Esclavo mediante comunicación serial.
- Incorporamos la biblioteca SoftwareSerial.
- Configuramos los pines 10 (RX) y 11 (TX).
- Establecimos la velocidad de comunicación en 9600 baudios.
- Mostramos el mensaje MAESTRO LISTO al iniciar.
- Revisamos continuamente si llegan datos desde el Esclavo.
- Si recibe información, contamos los bytes y mostramos el mensaje en el Monitor Serial.
Pendiente
 La comunicación entre ambos Arduino todavía no está funcionando correctamente.
Estado actual del proyecto
- Arduino Esclavo: código de prueba de envío preparado.
- Arduino Maestro: código de prueba de recepción preparado.
- Lectura de potenciómetros en el Esclavo: probada.
- Comunicación entre ambos: pendiente de resolver.
Próximo paso
En nuestra siguiente conversación nos concentraremos exclusivamente en solucionar la conexión entre el Arduino Esclavo y el Maestro, revisando los pines RX/TX, el cableado y la configuración hasta conseguir que el Maestro reciba correctamente el mensaje HOLA MAESTRO.

#include <SoftwareSerial.h>

SoftwareSerial comunicacion(10, 11);

void setup() {
  Serial.begin(9600);
  comunicacion.begin(9600);

  Serial.println("MAESTRO LISTO");
}

void loop() {
  int cantidad = comunicacion.available();

  if (cantidad > 0) {
    Serial.print("BYTES RECIBIDOS: ");
    Serial.println(cantidad);

    while (comunicacion.available()) {
      Serial.write(comunicacion.read());
    }
  }
}
restructure el codigo entonces lo que hice fue lo siguiente modificar y adaptar este codigo 
const uint8_t HEADER_COORD = 0xA0;
const uint8_t HEADER_START = 0xA1;

// En loop: envío directo sin delay
Serial.write(HEADER_COORD);
Serial.write(coordX);
Serial.write(coordY);

ahora lo que hice fue estabilizar las lectruaas Adc de los petenciometros 
el codigo que hice 
uint8_t leerYEscalarPotenciometro(uint8_t pin) {
  analogRead(pin); // Lectura descartada para limpiar el multiplexor ADC
  int lecturaRaw = analogRead(pin);
  
  if (pin == PIN_POT_X) {
    muestrasX[indiceFiltro] = lecturaRaw;
  } else {
    muestrasY[indiceFiltro] = lecturaRaw;
    indiceFiltro = (indiceFiltro + 1) % TAMANO_FILTRO;
  }
  
  long suma = 0;
  for (uint8_t i = 0; i < TAMANO_FILTRO; i++) {
    suma += (pin == PIN_POT_X) ? muestrasX[i] : muestrasY[i];
  }
  
  int promedioRaw = suma / TAMANO_FILTRO;
  long escalado = (promedioRaw * 100L) / 1024L;
  return (uint8_t)constrain(escalado, 0, 100);
}
