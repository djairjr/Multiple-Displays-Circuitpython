# Multiple-Displays-Circuitpython
I am trying to emulate a Nixie Tube Clock with four GC9A01 Displays, a Raspberry Pi Pico W, and custom Circuitpython 8.10

First things first:

Follow the instructions in @todbot blog to compile a custom version of Circuitpython. The default version allow only one display, and you get a bunch of errors.
https://todbot.com/blog/2022/05/19/multiple-displays-in-circuitpython-compiling-custom-circuitpython/

I've shared my compiled firmware (for Raspberry Pi Pico W) and the GIF files (I started this project testing the new animated GIF implementation for Circuitpython). This youtube video shows what I've done.
https://www.youtube.com/watch?v=DQaZTZmtok0

Tod suggests two displays, but I will try with four... Lets see if it works.


My current code is that:

```py
import board, busio, time, digitalio
import displayio, bitmaptools
import adafruit_imageload
import gc9a01

miso = board.GP4 # AMARELO
sck = board.GP6 # AZUL
mosi = board.GP7 # VERDE

# For two GC9A01 round LCDs using SPI, you can still use a single SPI bus,
# but you do need separate CS, DC, RST lines

cs =[board.GP10, board.GP11, board.GP12, board.GP13]
dc= [board.GP8, board.GP14, board.GP15, board.GP16]
rst=[board.GP9, board.GP17, board.GP18, board.GP19]

# In my boards, I can not use BLK pin...
# blk=[board.GP20, board.GP21, board.GP22, board.GP26]

spi = busio.SPI(clock=sck, MOSI=mosi, MISO=miso)

# Creating empty arrays...
display_bus = [0,0,0,0]
display = [0,0,0,0]
valve = [0,0,0,0]
number = [0,0,0,0]

for i in range (4):
  display_bus[i] = displayio.FourWire(spi, command=dc[i], chip_select=cs[i], reset=rst[i], baudrate = 40000000)
  display[i] = gc9a01.GC9A01(display_bus[i], width=240, height=240)
  valve[i] = displayio.Group(scale = 4) # Frame with 60 x 60 px

sprite_sheet, palette = adafruit_imageload.load(
    "nixie.bmp",
    bitmap=displayio.Bitmap,
    palette=displayio.Palette,
)

# make the color at 0 index transparent.
palette.make_transparent(0) # a Green
palette.make_transparent(1) # a different green
palette.make_transparent(2) # another green

# Create the sprite TileGrid
numbers = displayio.TileGrid(
    sprite_sheet,
    pixel_shader=palette,
    width=10,
    height=1,
    tile_width=60,
    tile_height=60,
    default_tile=0,
)

# Testing only one display
valve[0].append (numbers)
display[0].show (valve[0])

for i in range (10):
    numbers[0,0] = i
    time.sleep(1)
```

And for the GIFIO example (only one display):

```py
import board, busio, time, digitalio
import gifio
import displayio, bitmaptools
import gc9a01

miso = board.GP4 # AMARELO
sck = board.GP6 # AZUL
mosi = board.GP7 # VERDE

dc= [board.GP8, board.GP14, board.GP15, board.GP16]
rst=[board.GP9, board.GP17, board.GP18, board.GP19]
cs = [board.GP10, board.GP11, board.GP12, board.GP13]

spi = busio.SPI(clock=sck, MOSI=mosi, MISO=miso)

display_bus = [0,0,0,0]
display = [0,0,0,0]
valve = [0,0,0,0]
number = [0,0,0,0]
odg = gifio.OnDiskGif('/nixie.gif')
next_delay = odg.next_frame()
start = time.monotonic()
odg.next_frame() # Load the first frame
end = time.monotonic()
overhead = end - start

i = 0

print ("Criando display ", i)
display_bus[i] = displayio.FourWire(spi, command=dc[i], chip_select=cs[i], reset=rst[i], baudrate = 40000000)
display[i] = gc9a01.GC9A01(display_bus[i], width=240, height=240)
valve[i] = displayio.Group(scale = 3)
number[i] = displayio.TileGrid(odg.bitmap, pixel_shader =displayio.ColorConverter(input_colorspace=displayio.Colorspace.RGB565_SWAPPED), )
display[i].root_group = valve[i]
valve[i].append(number[i])
display[i].refresh()
```
