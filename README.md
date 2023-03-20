# LED Dot matrix (MAX7219)

El LED Dot Matrix (Matriz de puntos LED) es una pantalla formada por una matriz de puntos LED, que pueden ser controlados individualmente para crear diferentes patrones y textos. El MAX7219 es un controlador de LED dot matrix que se utiliza para simplificar el proceso de control de una matriz de puntos LED.

El MAX7219 es un circuito integrado que se encarga de la gestión de la matriz de puntos LED. Puede controlar hasta 64 LEDs, organizados en una matriz de 8x8, o dos matrices de 8x8, lo que permite la creación de una matriz de 16x8. El MAX7219 también puede controlar la intensidad de los LEDs, lo que permite ajustar el brillo de la pantalla.

El MAX7219 se comunica con un microcontrolador mediante un protocolo de comunicación serie de tres hilos (CLK, DIN y LOAD). El microcontrolador envía los datos a la matriz de puntos LED a través de la línea DIN, y utiliza el CLK para sincronizar la transmisión de datos. El LOAD se utiliza para latches los datos enviados a la matriz de puntos LED.

![Image alt text](sis1.png?raw=true)

# diagram.json
```json
{
  "version": 1,
  "author": "Anonymous maker",
  "editor": "wokwi",
  "parts": [
    {
      "type": "wokwi-pi-pico",
      "id": "pico",
      "top": 7.33,
      "left": -208.67,
      "attrs": { "env": "micropython-20220618-v1.19.1" }
    },
    {
      "type": "wokwi-max7219-matrix",
      "id": "matrix1",
      "top": 28.17,
      "left": -66.97,
      "attrs": { "chain": "4" }
    }
  ],
  "connections": [
    [ "matrix1:V+.2", "pico:VBUS", "green", [ "v0" ] ],
    [ "pico:GP2", "matrix1:CLK.2", "green", [ "h0" ] ],
    [ "pico:GP3", "matrix1:DOUT", "green", [ "h0" ] ],
    [ "pico:GP5", "matrix1:CS.2", "green", [ "h0" ] ],
    [ "pico:GND.2", "matrix1:GND.2", "black", [ "h0" ] ]
  ],
  "dependencies": {}
}
```


# max7219.py

```python

"""
MicroPython max7219 cascadable 8x8 LED matrix driver
https://github.com/mcauser/micropython-max7219

MIT License
Copyright (c) 2017 Mike Causer

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
"""

from micropython import const
import framebuf

_NOOP = const(0)
_DIGIT0 = const(1)
_DECODEMODE = const(9)
_INTENSITY = const(10)
_SCANLIMIT = const(11)
_SHUTDOWN = const(12)
_DISPLAYTEST = const(15)

class Matrix8x8:
    def __init__(self, spi, cs, num):
        """
        Driver for cascading MAX7219 8x8 LED matrices.

        >>> import max7219
        >>> from machine import Pin, SPI
        >>> spi = SPI(1)
        >>> display = max7219.Matrix8x8(spi, Pin('X5'), 4)
        >>> display.text('1234',0,0,1)
        >>> display.show()

        """
        self.spi = spi
        self.cs = cs
        self.cs.init(cs.OUT, True)
        self.buffer = bytearray(8 * num)
        self.num = num
        fb = framebuf.FrameBuffer(self.buffer, 8 * num, 8, framebuf.MONO_HLSB)
        self.framebuf = fb
        # Provide methods for accessing FrameBuffer graphics primitives. This is a workround
        # because inheritance from a native class is currently unsupported.
        # http://docs.micropython.org/en/latest/pyboard/library/framebuf.html
        self.fill = fb.fill  # (col)
        self.pixel = fb.pixel # (x, y[, c])
        self.hline = fb.hline  # (x, y, w, col)
        self.vline = fb.vline  # (x, y, h, col)
        self.line = fb.line  # (x1, y1, x2, y2, col)
        self.rect = fb.rect  # (x, y, w, h, col)
        self.fill_rect = fb.fill_rect  # (x, y, w, h, col)
        self.text = fb.text  # (string, x, y, col=1)
        self.scroll = fb.scroll  # (dx, dy)
        self.blit = fb.blit  # (fbuf, x, y[, key])
        self.init()

    def _write(self, command, data):
        self.cs(0)
        for m in range(self.num):
            self.spi.write(bytearray([command, data]))
        self.cs(1)

    def init(self):
        for command, data in (
            (_SHUTDOWN, 0),
            (_DISPLAYTEST, 0),
            (_SCANLIMIT, 7),
            (_DECODEMODE, 0),
            (_SHUTDOWN, 1),
        ):
            self._write(command, data)

    def brightness(self, value):
        if not 0 <= value <= 15:
            raise ValueError("Brightness out of range")
        self._write(_INTENSITY, value)

    def show(self):
        for y in range(8):
            self.cs(0)
            for m in range(self.num):
                self.spi.write(bytearray([_DIGIT0 + y, self.buffer[(y * self.num) + m]]))
            self.cs(1)
```

# main.py

```python

from machine import Pin, SPI
import max7219
from time import sleep

spi = SPI(0,sck=Pin(2),mosi=Pin(3))
cs = Pin(5, Pin.OUT)

display = max7219.Matrix8x8(spi, cs, 4)

display.brightness(10)

while True:

    display.fill(1)
    display.text('PICO',1,1,1)
    display.show()
    sleep(1)

    display.fill(0)
    display.text('1234',0,0,0)
    display.show()
    sleep(1)

    display.fill(0)
    display.text('done',0,0,1)
    display.show()
    sleep(1)
```

