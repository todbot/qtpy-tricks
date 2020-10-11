# QT Py Tricks

Some things to do on a stock [Adafruit QT Py](https://adafruit.com/qtpy)
running [CircuitPython](https://circuitpython.org) 6.
This list is mostly to help me remmeber how to do things in CircuitPython.

These code snippets will also work on a Trinket M0 and really just about
CircuitPython-compatible board, but you may need to adjust some of the `board` pins.

Notes:
- You will need to copy needed libraries from the [CircuitPython libraries bundle](https://circuitpython.org/libraries) to your CIRCUITPY drive
- When copying files to QT Py or Trinket M0, and you're on a Mac,
be sure to [use the `xattr` trick described here](https://todbot.com/blog/2020/10/03/prevent-annoying-mac-_-files-being-created-on-circuitpy/) to save flash space

## Table of Contents

* [Print "time" on OLED display](#print-time-on-oled-display)
* [Disco Party on built-in Neopixel](#disco-party-on-built-in-neopixel)
* [Output Farty Noises to DAC](#output-farty-noises-to-dac)
* [Capsense Touch to Colors on Built-in LED](#capsense-touch-to-colors-on-built-in-led)
* [Rotary Encoder to Built-in LED](#rotary-encoder-to-built-in-led)
* [Fire Simulation on External Neopixel Strip](#fire-simulation-on-external-neopixel-strip)
* [Two servos with Python Class for Easing / Sequencing](#two-servos-with-python-class-for-easing--sequencing)
* [Get Size of Device's Flash Disk](#get-size-of-devices-flash-disk)
* [Capsense Touch Sensor to USB keyboard](#capsense-touch-sensor-to-usb-keyboard)
         

## Print "time" on OLED display
```py
import time
import board 
import adafruit_ssd1306 # requires: adafruit_bus_device and adafruit_framebuf
i2c = board.I2C()
oled = adafruit_ssd1306.SSD1306_I2C(width=128, height=32, i2c=i2c)
while True:
    oled.fill(0)
    oled.text( "hello world", 0,0,1) # requires 'font5x8.bin'
    oled.text("time:"+str(time.monotonic()), 0,8,1)
    oled.show()
    time.sleep(1.0)
```
<img width=400 src="./imgs/qtpy-oled.jpg"/>

## Disco Party on built-in Neopixel
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
<img width=400 src="./imgs/qtpy-neodisco.gif" />

## Output Farty Noises to DAC

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
<img width=400 src="./imgs/qtpy-farty.jpg"/>

([hear it in action](https://twitter.com/todbot/status/1313223090181042177))



## Capsense Touch to Colors on Built-in LED

```py
import board
from touchio import TouchIn
import neopixel
pixel = neopixel.NeoPixel(board.NEOPIXEL, 1, brightness=0.2)
touchA = TouchIn(board.A1)
touchB = TouchIn(board.A2)
print("hello")
while True:
  pixel[0] = (int(touchA.value*255), 0, int(touchB.value*255))
```
<img width=400 src="./imgs/qtpy-capsense.gif"/>

## Rotary Encoder to Built-in LED

```py
import board
import time
import neopixel
import rotaryio
pixel = neopixel.NeoPixel(board.NEOPIXEL, 1, brightness=1.0)
encoder = rotaryio.IncrementalEncoder( board.MOSI, board.MISO ) # any two pins
while True:
    b = (encoder.position % 32) * 8
    print(encoder.position,b)
    pixel.fill((0,0,b))
    time.sleep(0.1)
```
<img width=400 src="./imgs/qtpy-encoder.gif"/>


## Fire Simulation on External Neopixel Strip
Uses Python array operations and list comprehensions for conciseness.

```py
import board
import time
import neopixel
from random import randint
# External Neopixel strip, can be on any pin
leds = neopixel.NeoPixel(board.RX,8,brightness=0.2,auto_write=False)
while True:
    # reduce brightness of all pixels by (30,30,30)
    leds[0:] = [[max(i-30,0) for i in l] for l in leds]
    # shift LED values down by one
    leds[1:] = leds[0:] 
    # pick new random fire color for LED 0
    leds[0] = (randint(150,255),randint(50,100),0) 
    leds.show()
    time.sleep(0.1)
```
<img width=400 src="./imgs/qtpy-fire.gif"/>

##  Two servos with Python Class for Easing / Sequencing

```py
import board
import time
from pulseio import PWMOut
from adafruit_motor import servo
from random import random

servoA = servo.Servo(PWMOut(board.RX, duty_cycle=2**15, frequency=50))
servoB = servo.Servo(PWMOut(board.TX, duty_cycle=2**15, frequency=50))

class AngleSequence:
    def __init__(self,angles,speed):
        self.angles = angles
        self.aindex = 0
        self.angle = angles[0]
        self.speed = speed
    def next_angle(self):
        self.aindex = (self.aindex + 1) % len(self.angles)
    def update(self):
        new_angle = self.angles[self.aindex]
        self.angle += (new_angle - self.angle) * self.speed  # simple easing
        return self.angle

seqA = AngleSequence( [90,70,90,90,90,90,60,80,100], 0.1) # set speed to taste
seqB = AngleSequence( [90,90,90,80,70,90,120], 0.1) # set angles for desired animation

lasttime = time.monotonic()
while True:
    if time.monotonic() - lasttime > (0.2 + random()*4.0  ):  # creepy random timing
        lasttime = time.monotonic()
        seqA.next_angle() # go to next angle in list
        seqB.next_angle()
    servoA.angle = seqA.update() # get updated (eased) servo angle
    servoB.angle = seqB.update()
    time.sleep(0.02) # wait a bit, note this affects 'speed'
```
<img width=400 src="./imgs/qtpy-servoeye.gif"/>


## Get Size of Device's Flash Disk
```py
import os
print("\nHello World!")
fs_stat = os.statvfs('/')
print("Disk size in MB", fs_stat[0] * fs_stat[3] / 1024 / 1024)
while True: pass
```
<img width=400 src="./imgs/qtpy-flashsize-2MB.png"/>

## Capsense Touch Sensor to USB keyboard

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

