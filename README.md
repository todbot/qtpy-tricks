# QT Py Tricks

Some things to do on a stock QT Py


### Print time on OLED display
```py
import time
import board 
import adafruit_ssd1306

i2c = board.I2C()
oled = adafruit_ssd1306.SSD1306_I2C(128, 64, i2c)

while True:
    oled.fill(0)
    oled.text( "hello world", 0,0,1)
    oled.text("time:"+str(time.monotonic()), 0,8,1)
    oled.show()
    time.sleep(1.0)
```
<img src="./imgs/qtpy-oled.jpg"/>

### Disco Party on built-in Neopixel
```py
import time
import board
import neopixel
from random import randint
pixel = neopixel.NeoPixel(board.NEOPIXEL, 1, brightness=0.2)
while True:
    pixel.fill( (randint(0,255), randint(0,255), randint(0,255) ))
    time.sleep(0.1)
```
<img src="./imgs/qtpy-neodisco.gif" />

### Output Farty Noises to DAC

```py
import time
import board 
import analogio
import random
dac = analogio.AnalogOut(board.A0)
i = 0
di = 40000
lasttime = 0
while True:
    dac.value = i
    i = (i+di) & 0xffff
    if time.monotonic() - lasttime > 1:
        lasttime = time.monotonic()
        di = random.randrange(40000,80000)
```
<img src="./imgs/qtpy-farty.jpg"/>


### Capsense Touch to colors on built-in LED

```py
import board
from touchio import TouchIn
import neopixel
pixel = neopixel.NeoPixel(board.NEOPIXEL, 1, brightness=0.2, auto_write=False)
touchA = TouchIn(board.A1)
touchB = TouchIn(board.A2)
while True:
  pixel[0] = (int(touchA.value*255), 0, int(touchB.value*255))
```

### Capsense Touch sensor to USB keyboard

```py
import time
import board 
import neopixel
import touchio
import usb_hid
from adafruit_hid.keyboard import Keyboard
from adafruit_hid.keycode import Keycode
touchA= touchio.TouchIn(board.A0)
pixel = neopixel.NeoPixel(board.NEOPIXEL, 1, brightness=0.2, auto_write=False)
pixel[0] = (0,0,0)
time.sleep(1)  # Sleep for a bit to avoid a race condition on some systems
kbd = Keyboard(usb_hid.devices)
while True:
    if touchA.value:
        print("A press")
        pixel[0] = (255,0,0)
        kbd.send(Keycode.RIGHT_ARROW)
    pixel.show()
    pixel[0] = (0,0,0)
    time.sleep(0.2)
```

