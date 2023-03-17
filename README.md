# LED Dot matrix (MAX7219)

El LED Dot Matrix (Matriz de puntos LED) es una pantalla formada por una matriz de puntos LED, que pueden ser controlados individualmente para crear diferentes patrones y textos. El MAX7219 es un controlador de LED dot matrix que se utiliza para simplificar el proceso de control de una matriz de puntos LED.

El MAX7219 es un circuito integrado que se encarga de la gestión de la matriz de puntos LED. Puede controlar hasta 64 LEDs, organizados en una matriz de 8x8, o dos matrices de 8x8, lo que permite la creación de una matriz de 16x8. El MAX7219 también puede controlar la intensidad de los LEDs, lo que permite ajustar el brillo de la pantalla.

El MAX7219 se comunica con un microcontrolador mediante un protocolo de comunicación serie de tres hilos (CLK, DIN y LOAD). El microcontrolador envía los datos a la matriz de puntos LED a través de la línea DIN, y utiliza el CLK para sincronizar la transmisión de datos. El LOAD se utiliza para latches los datos enviados a la matriz de puntos LED.

![Image alt text](sis1.png?raw=true)
