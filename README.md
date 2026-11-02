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
