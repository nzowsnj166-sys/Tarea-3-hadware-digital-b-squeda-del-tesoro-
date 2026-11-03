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

este fue arguegado al arduino 1 "esclavo" 
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

ahora implemente las fms de juego porque no tenia limite estricto de ganar o perder entonces no se sabia cuando seguia o continueva lo que hice fue que agregue este codigo en el arduino 2  "maestro "
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

ahra lo que hice fue el calculo de la zona 2x2 con los dx y dy y el temporizador de pertenencia para saber si se ganaba con el roze de cordenadas y mas mi codigo agregado en el arduino 2"maestro fue este"

void calcularProximidadEjes() {
  int dx = (jugadorX < (tesoroX - 1)) ? (tesoroX - 1) - jugadorX : (jugadorX > tesoroX ? jugadorX - tesoroX : 0);
  int dy = (jugadorY < (tesoroY - 1)) ? (tesoroY - 1) - jugadorY : (jugadorY > tesoroY ? jugadorY - tesoroY : 0);
  
  nivelX = (dx > 10) ? 0 : ((dx >= 5) ? 1 : ((dx >= 1) ? 2 : 3));
  nivelY = (dy > 10) ? 0 : ((dy >= 5) ? 1 : ((dy >= 1) ? 2 : 3));
}

// Evaluación estricta de permanencia:
if (nivelX == 3 && nivelY == 3) {
  if (tiempoInicioPermanenciaMs == 0) {
    tiempoInicioPermanenciaMs = tiempoActualMs;
  } else if (tiempoActualMs - tiempoInicioPermanenciaMs >= TIEMPO_PERMANENCIA_MS) {
    estadoActual = VICTORIA;
    tiempoInicioPartidaMs = tiempoActualMs;
  }
} else {
  tiempoInicioPermanenciaMs = 0; // Se reinicia si te sales del área
}

ahor alo que hice fue que el potenciometro no actualizava el nivel de proximidad de cada eje entonces lo que hice fue lo siguiente en el pedazo de codigo que le agregue al arduino 1 "esclavo"

uint8_t leerEjeX() {
  analogRead(PIN_POT_X);
  int lecturaRaw = analogRead(PIN_POT_X);
  muestrasX[indiceFiltroX] = lecturaRaw;
  indiceFiltroX = (indiceFiltroX + 1) % TAMANO_FILTRO;
  long suma = 0;
  for (uint8_t i = 0; i < TAMANO_FILTRO; i++) suma += muestrasX[i];
  return (uint8_t)constrain(((suma / TAMANO_FILTRO) * 100L) / 1024L, 0, 100);
}

uint8_t leerEjeY() {
  analogRead(PIN_POT_Y);
  int lecturaRaw = analogRead(PIN_POT_Y);
  muestrasY[indiceFiltroY] = lecturaRaw;
  indiceFiltroY = (indiceFiltroY + 1) % TAMANO_FILTRO;
  long suma = 0;
  for (uint8_t i = 0; i < TAMANO_FILTRO; i++) suma += muestrasY[i];
  return (uint8_t)constrain(((suma / TAMANO_FILTRO) * 100L) / 1024L, 0, 100);
}



tuve que recablear en la protoboard los pines 6y 5  de los leds 1 y 2 mire 
<img width="527" height="428" alt="image" src="https://github.com/user-attachments/assets/fbf0214b-2b38-4a44-a047-649c6aa841f4" />

y con esto mostrare mi diagrama final ya que hice una mejora en el recableado y todo lo que iva haciendo y lo que iva trabajando me sirvio bastante para todo entonces entre tanta prueba ensayo y error pude concluir el trabajo 

<img width="1360" height="542" alt="Incredible Uusam-Blorr" src="https://github.com/user-attachments/assets/e3d71b7a-4e59-48fc-b29c-73dd90cba229" />
y el codigo aguegado al arduino 1 "esclavo":

// Remapeo de pines ajustado al montaje real probado
const uint8_t PIN_LED_X = 6; // LED Eje X (~PWM)
const uint8_t PIN_LED_Y = 5; // LED Eje Y (~PWM)

void setup() {
  pinMode(PIN_BTN_INICIO, INPUT_PULLUP);
  pinMode(PIN_LED_X, OUTPUT);
  pinMode(PIN_LED_Y, OUTPUT);
  
  // Test de encendido físico al energizar (ambos encienden por 1 segundo)
  digitalWrite(PIN_LED_X, HIGH);
  digitalWrite(PIN_LED_Y, HIGH);
  delay(1000);
  digitalWrite(PIN_LED_X, LOW);
  digitalWrite(PIN_LED_Y, LOW);
  
  Serial.begin(9600);
}

para finalizar 

---

## 🧠 Arquitectura del Sistema: ¿Por qué Maestro y Esclavo?

En sistemas embebidos y hardware distribuido, la denominación **Maestro / Esclavo** no depende de la posición física de las placas en la mesa ni de cuál envía datos primero, sino de **dónde reside la jerarquía de control y la toma de decisiones** del proyecto:

| Placa | Rol Arquitectónico | Responsabilidad Principal |
| :--- | :--- | :--- |
| **Arduino 2** | **Maestro** *(Cerebro / Game Engine)* | Gobierna la Máquina de Estados (FSM), la lógica matemática, temporizadores y reglas de victoria/derrota. |
| **Arduino 1** | **Esclavo** *(Interfaz I/O / Periférico)* | Lee sensores (potenciómetros), atiende el pulsador y ejecuta las órdenes de visualización (LEDs PWM). |

### 1. Arduino 2 — Maestro (Motor del Juego)
* **Control de la FSM:** Administra la transición estricta entre los estados `INACTIVO`, `JUGANDO`, `VICTORIA` y `DERROTA`.
* **Generación pseudoaleatoria:** Inicializa la semilla analógica con ruido térmico en `A0` (`randomSeed`) y genera la posición del tesoro en una cuadrícula virtual de $100 \times 100$.
* **Cálculo de distancias:** Determina el desplazamiento ortogonal ($dx$, $dy$) respecto al área meta de $2 \times 2$ casillas.
* **Temporizador global:** Ejecuta la atenuación PWM del LED verde (Pin `D3`) a lo largo de 60 segundos con aritmética no bloqueante mediante `millis()`.
* **Árbitro del juego:** Verifica el cumplimiento continuo de 2000 ms de permanencia en el objetivo antes de conceder la victoria.

### 2. Arduino 1 — Esclavo (Módulo de Entrada/Salida)
* **Adquisición analógica:** Muestrea los potenciómetros en `A0` y `A1`, aplicando descarte por multiplexor y filtro de promedio móvil (5 muestras) para eliminar ruido parásito.
* **Debouncing por software:** Detecta la pulsación en `D2` con resistencia `INPUT_PULLUP` mediante temporización con `millis()` sin detener el flujo del procesador.
* **Subordinación de control:** No procesa lógica de victoria ni cronómetros; recibe periódicamente la trama de estado del Maestro y modula el ciclo de trabajo de los pines `D6` y `D5` según los niveles PWM definidos:
  $$\text{PWM\_NIVELES}[4] = \{0,\; 20,\; 80,\; 255\}$$

---

## 🤖 Declaración de Uso de Asistencia por IA

> [!NOTE]
> Durante la realización de este trabajo se utilizó asistencia de Inteligencia Artificial (LLM) como **herramienta de soporte técnico, depuración (*debugging*) y optimización de código**, orientada a cumplir rigurosamente con los requisitos de la pauta académica de la asignatura CIN-212.

### Alcance y Aplicación Técnica de la IA:

1. **Protocolo UART Nativo:**  
   Asistencia en el diseño de tramas binarias compactas (`0xA0`, `0xA1`, `0xB0`) y ventanas de recepción no bloqueantes de 20 ms, resolviendo la saturación de búfer que presentaba `SoftwareSerial` en la simulación.
2. **Mitigación de Ruido en el ADC:**  
   Diagnóstico de la diafonía y capacitancia parásita interna del conversor analógico-digital del microcontrolador ATmega328P al conmutar entre canales, implementando el descarte de la primera lectura y desacoplando los buffers de promedio móvil de cada eje.
3. **Modelado Matemático de la FSM:**  
   Estructuración de las ecuaciones de proximidad por eje hacia el área de $2 \times 2$ casillas y el algoritmo de verificación temporal ininterrumpida de permanencia (2 segundos).
4. **Depuración del Circuito en Protoboard:**  
   Identificación visual y lógica de fallas de conexionado físico en Tinkercad (cortocircuito a tierra y falta de alimentación en el ánodo del LED del Eje Y), guiando la corrección en la protoboard y el remapeo de pines a nivel de firmware.

---

mis codigos de mis arduinos 1"esclavo"

/*
 * PROYECTO: Búsqueda del Tesoro - Módulo Esclavo (Pines Invertidos: X en D6, Y en D5)
 * ASIGNATURA: CIN-212 Hardware Digital - Universidad de Valparaíso
 */

const uint8_t PIN_POT_X = A0;
const uint8_t PIN_POT_Y = A1;
const uint8_t PIN_BTN_INICIO = 2;

// ASIGNACIÓN AJUSTADA A TU CIRCUITO:
const uint8_t PIN_LED_X = 6; // Tu primer LED está en el pin 6 (~PWM)
const uint8_t PIN_LED_Y = 5; // Tu segundo LED está en el pin 5 (~PWM)

const uint8_t HEADER_COORD   = 0xA0;
const uint8_t HEADER_START   = 0xA1;
const uint8_t HEADER_MAESTRO = 0xB0;

const uint8_t PWM_NIVELES[4] = {0, 20, 80, 255};
const uint8_t TAMANO_FILTRO = 5;

int muestrasX[TAMANO_FILTRO] = {0};
int muestrasY[TAMANO_FILTRO] = {0};
uint8_t indiceFiltroX = 0;
uint8_t indiceFiltroY = 0;

unsigned long ultimoEnvioSerialMs = 0;
const unsigned long INTERVALO_ENVIO_MS = 60;

int ultimoEstadoBoton = HIGH;
unsigned long ultimoReboteMs = 0;
const unsigned long TIEMPO_DEBOUNCE_MS = 30;

uint8_t estadoJuegoRecibido = 0;
uint8_t nivelProximidadX = 0;
uint8_t nivelProximidadY = 0;

uint8_t leerEjeX();
uint8_t leerEjeY();
void procesarBoton();
void recibirEstadoMaestro();
void actualizarLedsProximidad();

void setup() {
  pinMode(PIN_BTN_INICIO, INPUT_PULLUP);
  pinMode(PIN_LED_X, OUTPUT);
  pinMode(PIN_LED_Y, OUTPUT);
  
  // Test de encendido inicial (1 segundo ambos encendidos al 100%)
  digitalWrite(PIN_LED_X, HIGH);
  digitalWrite(PIN_LED_Y, HIGH);
  delay(1000);
  digitalWrite(PIN_LED_X, LOW);
  digitalWrite(PIN_LED_Y, LOW);
  
  Serial.begin(9600);
}

void loop() {
  unsigned long tiempoActualMs = millis();
  
  procesarBoton();
  
  if (tiempoActualMs - ultimoEnvioSerialMs >= INTERVALO_ENVIO_MS) {
    ultimoEnvioSerialMs = tiempoActualMs;
    
    uint8_t coordX = leerEjeX();
    uint8_t coordY = leerEjeY();
    
    Serial.write(HEADER_COORD);
    Serial.write(coordX);
    Serial.write(coordY);
  }
  
  recibirEstadoMaestro();
  actualizarLedsProximidad();
}

void procesarBoton() {
  int lectura = digitalRead(PIN_BTN_INICIO);
  
  if (lectura != ultimoEstadoBoton) {
    if ((millis() - ultimoReboteMs) > TIEMPO_DEBOUNCE_MS) {
      if (lectura == LOW) {
        Serial.write(HEADER_START);
      }
      ultimoReboteMs = millis();
    }
    ultimoEstadoBoton = lectura;
  }
}

uint8_t leerEjeX() {
  analogRead(PIN_POT_X);
  int lecturaRaw = analogRead(PIN_POT_X);
  
  muestrasX[indiceFiltroX] = lecturaRaw;
  indiceFiltroX = (indiceFiltroX + 1) % TAMANO_FILTRO;
  
  long suma = 0;
  for (uint8_t i = 0; i < TAMANO_FILTRO; i++) {
    suma += muestrasX[i];
  }
  
  int promedioRaw = suma / TAMANO_FILTRO;
  long escalado = (promedioRaw * 100L) / 1024L;
  return (uint8_t)constrain(escalado, 0, 100);
}

uint8_t leerEjeY() {
  analogRead(PIN_POT_Y);
  int lecturaRaw = analogRead(PIN_POT_Y);
  
  muestrasY[indiceFiltroY] = lecturaRaw;
  indiceFiltroY = (indiceFiltroY + 1) % TAMANO_FILTRO;
  
  long suma = 0;
  for (uint8_t i = 0; i < TAMANO_FILTRO; i++) {
    suma += muestrasY[i];
  }
  
  int promedioRaw = suma / TAMANO_FILTRO;
  long escalado = (promedioRaw * 100L) / 1024L;
  return (uint8_t)constrain(escalado, 0, 100);
}

void recibirEstadoMaestro() {
  while (Serial.available() > 0) {
    if (Serial.read() == HEADER_MAESTRO) {
      unsigned long t0 = millis();
      while (Serial.available() < 3 && (millis() - t0 < 20));
      
      if (Serial.available() >= 3) {
        estadoJuegoRecibido = Serial.read();
        nivelProximidadX = Serial.read();
        nivelProximidadY = Serial.read();
      }
    }
  }
}

void actualizarLedsProximidad() {
  switch (estadoJuegoRecibido) {
    case 0: // INACTIVO
      analogWrite(PIN_LED_X, 0);
      analogWrite(PIN_LED_Y, 0);
      break;
      
    case 1: // JUGANDO: modulación PWM independiente
      analogWrite(PIN_LED_X, PWM_NIVELES[constrain(nivelProximidadX, 0, 3)]);
      analogWrite(PIN_LED_Y, PWM_NIVELES[constrain(nivelProximidadY, 0, 3)]);
      break;
      
    case 2: { // VICTORIA: parpadeo a 2 Hz
      bool parpadeo = (millis() / 250) % 2;
      digitalWrite(PIN_LED_X, parpadeo ? HIGH : LOW);
      digitalWrite(PIN_LED_Y, parpadeo ? HIGH : LOW);
      break;
    }
    
    case 3: // DERROTA
      analogWrite(PIN_LED_X, 0);
      analogWrite(PIN_LED_Y, 0);
      break;
  }
}

y el codigo de mi arduino 2"maestro"

/*
 * PROYECTO: Búsqueda del Tesoro - Módulo Maestro
 * ASIGNATURA: CIN-212 Hardware Digital - Universidad de Valparaíso
 * Pauta oficial: Máquina de estados, temporizador lineal 60s, zona 2x2, permanencia 2s
 */

const uint8_t PIN_LED_VERDE = 3; 
const uint8_t PIN_RANDOM_SEED = A0;

const uint8_t HEADER_COORD   = 0xA0;
const uint8_t HEADER_START   = 0xA1;
const uint8_t HEADER_MAESTRO = 0xB0;

enum EstadoJuego { INACTIVO, JUGANDO, VICTORIA, DERROTA };
EstadoJuego estadoActual = INACTIVO;

const unsigned long DURACION_PARTIDA_MS    = 60000;
const unsigned long TIEMPO_PERMANENCIA_MS  = 2000;
const unsigned long INTERVALO_ENVIO_MS     = 60;

unsigned long tiempoInicioPartidaMs = 0;
unsigned long tiempoInicioPermanenciaMs = 0;
unsigned long ultimoEnvioSerialMs = 0;
unsigned long ultimoPrintDebugMs = 0;

uint8_t tesoroX = 0;
uint8_t tesoroY = 0;
uint8_t jugadorX = 0;
uint8_t jugadorY = 0;
uint8_t nivelX = 0;
uint8_t nivelY = 0;

void generarUbicacionTesoro();
void recibirDatosEsclavo();
void calcularProximidadEjes();
void actualizarMaquinaDeEstados();
void enviarEstadoEsclavo();

void setup() {
  pinMode(PIN_LED_VERDE, OUTPUT);
  analogWrite(PIN_LED_VERDE, 0);
  
  Serial.begin(9600);
  randomSeed(analogRead(PIN_RANDOM_SEED));
}

void loop() {
  unsigned long tiempoActualMs = millis();
  
  recibirDatosEsclavo();
  actualizarMaquinaDeEstados();
  
  if (tiempoActualMs - ultimoEnvioSerialMs >= INTERVALO_ENVIO_MS) {
    ultimoEnvioSerialMs = tiempoActualMs;
    enviarEstadoEsclavo();
  }
}

void generarUbicacionTesoro() {
  tesoroX = random(1, 101);
  tesoroY = random(1, 101);
  Serial.println(F("\n=============================================="));
  Serial.print(F(">>> TESORO EN: X="));
  Serial.print(tesoroX);
  Serial.print(F(" | Y="));
  Serial.print(tesoroY);
  Serial.println(F(" (Zona 2x2) <<<"));
  Serial.println(F("=============================================="));
}

void recibirDatosEsclavo() {
  while (Serial.available() > 0) {
    uint8_t header = Serial.read();
    
    if (header == HEADER_START) {
      if (estadoActual == INACTIVO) {
        generarUbicacionTesoro();
        estadoActual = JUGANDO;
        tiempoInicioPartidaMs = millis();
        tiempoInicioPermanenciaMs = 0;
      }
    }
    else if (header == HEADER_COORD) {
      unsigned long t0 = millis();
      while (Serial.available() < 2 && (millis() - t0 < 20));
      
      if (Serial.available() >= 2) {
        jugadorX = Serial.read();
        jugadorY = Serial.read();
      }
    }
  }
}

void calcularProximidadEjes() {
  int dx = 0;
  if (jugadorX < (tesoroX - 1)) {
    dx = (tesoroX - 1) - jugadorX;
  } else if (jugadorX > tesoroX) {
    dx = jugadorX - tesoroX;
  }
  
  int dy = 0;
  if (jugadorY < (tesoroY - 1)) {
    dy = (tesoroY - 1) - jugadorY;
  } else if (jugadorY > tesoroY) {
    dy = jugadorY - tesoroY;
  }
  
  // Regla matemática de proximidad solicitada en el enunciado:
  if (dx > 10)       nivelX = 0; // Lejos
  else if (dx >= 5)  nivelX = 1; // Tibio
  else if (dx >= 1)  nivelX = 2; // Cerca
  else               nivelX = 3; // Sobre el tesoro
  
  if (dy > 10)       nivelY = 0; // Lejos
  else if (dy >= 5)  nivelY = 1; // Tibio
  else if (dy >= 1)  nivelY = 2; // Cerca
  else               nivelY = 3; // Sobre el tesoro
}

void actualizarMaquinaDeEstados() {
  unsigned long tiempoActualMs = millis();
  
  switch (estadoActual) {
    case INACTIVO:
      analogWrite(PIN_LED_VERDE, 0);
      nivelX = 0;
      nivelY = 0;
      break;
      
    case JUGANDO: {
      unsigned long tiempoTranscurridoMs = tiempoActualMs - tiempoInicioPartidaMs;
      
      if (tiempoTranscurridoMs > DURACION_PARTIDA_MS) {
        estadoActual = DERROTA;
        tiempoInicioPartidaMs = tiempoActualMs;
        Serial.println(F("\n[MAESTRO] >> TIEMPO AGOTADO. DERROTA."));
        break;
      }
      
      // Atenuación lineal de 255 a 0 proporcional a los 60 segundos
      long pwmRestante = 255L - ((long)tiempoTranscurridoMs * 255L / (long)DURACION_PARTIDA_MS);
      analogWrite(PIN_LED_VERDE, constrain(pwmRestante, 0, 255));
      
      calcularProximidadEjes();
      
      if (tiempoActualMs - ultimoPrintDebugMs >= 500) {
        ultimoPrintDebugMs = tiempoActualMs;
        unsigned long segRestantes = (DURACION_PARTIDA_MS - tiempoTranscurridoMs) / 1000;
        Serial.print(F("[JUEGO] "));
        Serial.print(segRestantes);
        Serial.print(F("s | Pos: ("));
        Serial.print(jugadorX);
        Serial.print(F(","));
        Serial.print(jugadorY);
        Serial.print(F(") | Prox X:"));
        Serial.print(nivelX);
        Serial.print(F(" Y:"));
        Serial.print(nivelY);
        Serial.print(F(" | Tesoro: ("));
        Serial.print(tesoroX);
        Serial.print(F(","));
        Serial.print(tesoroY);
        Serial.println(F(")"));
      }
      
      // Permanencia obligatoria de 2000 ms en área 2x2
      if (nivelX == 3 && nivelY == 3) {
        if (tiempoInicioPermanenciaMs == 0) {
          tiempoInicioPermanenciaMs = tiempoActualMs;
        } else if (tiempoActualMs - tiempoInicioPermanenciaMs >= TIEMPO_PERMANENCIA_MS) {
          estadoActual = VICTORIA;
          tiempoInicioPartidaMs = tiempoActualMs;
          Serial.println(F("\n=============================================="));
          Serial.println(F(">>> VICTORIA CUMPLIDA! (2s de permanencia) <<<"));
          Serial.println(F("=============================================="));
        }
      } else {
        tiempoInicioPermanenciaMs = 0;
      }
      break;
    }
    
    case VICTORIA:
      analogWrite(PIN_LED_VERDE, 255); // Brillo máximo continuo
      if (tiempoActualMs - tiempoInicioPartidaMs > 5000) {
        estadoActual = INACTIVO;
      }
      break;
      
    case DERROTA: {
      bool parpadeo = (millis() / 250) % 2; // Parpadeo a 2 Hz
      analogWrite(PIN_LED_VERDE, parpadeo ? 255 : 0);
      if (tiempoActualMs - tiempoInicioPartidaMs > 5000) {
        estadoActual = INACTIVO;
      }
      break;
    }
  }
}

void enviarEstadoEsclavo() {
  Serial.write(HEADER_MAESTRO);
  Serial.write((uint8_t)estadoActual);
  Serial.write(nivelX);
  Serial.write(nivelY);
}
