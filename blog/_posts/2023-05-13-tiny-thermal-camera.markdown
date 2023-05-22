---
layout: post
title:  "Tiny Thermal Camera"
date:   2023-05-13 16:44:09 -0400
categories: update
---
`Radiometric thermal imaging sensors` are fascinating. Just like the more traditional image sensor in your phone, a `radiometric imaging sensor` takes in light and outputs a 2-dimensional matrix of values. In a traditional imaging sensor, these values are colors which a computer can stitch together to display an image. However, the values that a radiometric sensor returns are not colors but temperature readings! This means we can manufacture cameras which can see temperature. Being able see temperature is extremely useful for many tasks: from checking if the stove is hot, to surveying houses for insulation problems, and even in helping combat poachers! With all these uses one might logically question why radiometric imaging sensors are not more widespread; why don't our phones come with thermal imaging capabilities?


To answer this question we must understand a bit about how radiometric thermal imaging sensors function, and to do this we must first take a step back. When we mentioned before that these sensors take in `light` what we really mean are photon particles. An inconceivable number of photons of widely varying frequencies are vibrating and bouncing around all over our universe at all times. These photons are created by many different sources such as the sun, cosmic rays, microwaves, fireflies, wifi routers, radio stations, someone turning on a flashlight, etc.... When the frequency of the photons is between 400 THz to 800 THz - that is vibrating at 400 to 800 trillion times a second - we call this `visible light` because that is roughly the frequency range to which the rods and cones in our eyes can detect. But there are myriads of photons all around us that fall outside this frequency range that we cannot see with our naked eyes which begs the question - do we still call the photons we cannot see `light`?


Speaking of photons we cannot see, all objects are constantly emitting photons in the form of thermal energy. These photons vibrate at a much slower frequency than visible light. How much slower? About 20 to 40 times slower. Because the photons emitted in the form of thermal radiation are vibrating slower, they carry less energy than visible light. This means any sensor `looking` for thermal radiation has to be more sensitive and precise and hence more expensive. Even so, thermal cameras are a great tool for many purposes and are nothing short of amazing. To be able to point a camera at a stove and see the temperature without having to touch anything is nothing short of a miracle!


Fascinated by thermal imaging sensors, I wanted to take a crack at making my own tiny thermal camera with a basic user interface. Nothing too fancy just something to convert the temperature readings from the radiometric sensor to a heatmap image - something a human could easily read. For this project I used:
* Adafruit MLX90640 radiometric thermal imaging sensor.
* Raspbery Pi Pico (any RP2040 or arduino board should do)
* ST7789 SPI display
* Breadboard + wires


I used the left side of the Pico for an I2C connection to the MLX90640, while the right side of the board is used for SPI connection to the display. From there I wrote the following circuitpython code to read in the array of temperature readings from the MLX90640, convert that image into a heatmap and then display it along with the high and low temperature on the display. Because of the limited processing power and ram it will run ~2FPS on a Pico board but can do up to ~9HZ with a more powerful board.

Here is the code but if code is not your thing feel free to skip over this to see the final result :)
{% highlight python %}
import time
import board
import busio
import adafruit_mlx90640
import terminalio
import displayio
import bitmaptools
from adafruit_display_text import label
from adafruit_st7789 import ST7789

# Thermal Camera Software
# Isaac Flaum 2023
print("Board Pins:")
print(dir(board))

zoom_scale = 5.625

displayio.release_displays()
display_font = terminalio.FONT

display_spi_connection = busio.SPI(clock=board.SCK,
                MOSI=board.MOSI,
                MISO=board.MISO)
display_cs = board.A2
display_dc = board.A1

display_bus = displayio.FourWire(display_spi_connection, command=display_dc, chip_select=display_cs)
display = ST7789(
    display_bus, rotation=270, width=240, height=135, rowstart=40, colstart=53
)

mlx_90640_i2c_connection = busio.I2C(board.D1, board.D0, frequency=800000)
mlx_90640 = adafruit_mlx90640.MLX90640(mlx_90640_i2c_connection)
mlx_90640.refresh_rate = adafruit_mlx90640.RefreshRate.REFRESH_2_HZ

flir_ironbow_colors = [0x00000a, 0x00001e, 0x00002a, 0x000032, 0x00003a, 0x000042, 0x00004a, 0x000052, 0x010057, 0x02005c, 0x040061, 0x050065, 0x070069, 0x09006e, 0x0b0073,
                       0x0d0075, 0x0e0077, 0x120079, 0x15007c, 0x19007e, 0x1c0081, 0x200084, 0x240086, 0x280089, 0x2c008a, 0x30008c, 0x34008e, 0x38008f, 0x3b0091, 0x3e0093,
                       0x410094, 0x440095, 0x470096, 0x4a0096, 0x4e0097, 0x510097, 0x540098, 0x580099, 0x5c0099, 0x5f009a, 0x63009b, 0x66009b, 0x6a009b, 0x6d009c, 0x70009c,
                       0x73009d, 0x77009d, 0x7a009d, 0x7e009d, 0x81009d, 0x84009d, 0x87009d, 0x8a009d, 0x8d009d, 0x91009c, 0x95009c, 0x98009b, 0x9b009b, 0x9d009b, 0xa0009b,
                       0xa3009b, 0xa6009a, 0xa8009a, 0xaa0099, 0xad0099, 0xaf0198, 0xb00198, 0xb20197, 0xb40296, 0xb60295, 0xb80395, 0xba0495, 0xbb0593, 0xbd0593, 0xbf0692,
                       0xc00791, 0xc10890, 0xc20a8f, 0xc30b8e, 0xc50c8c, 0xc60e8a, 0xc81088, 0xca1286, 0xcb1385, 0xcc1582, 0xce1780, 0xcf187c, 0xd01a79, 0xd11c76, 0xd21d74,
                       0xd32071, 0xd4226e, 0xd52469, 0xd72665, 0xd82862, 0xda2b5e, 0xdb2e5a, 0xdc2f54, 0xdd314e, 0xde3347, 0xdf3541, 0xe0373a, 0xe03933, 0xe23b2d, 0xe33d26,
                       0xe43f20, 0xe4421c, 0xe54419, 0xe64616, 0xe74814, 0xe84a12, 0xe84c0f, 0xe94d0d, 0xea4f0c, 0xeb510a, 0xeb5309, 0xec5608, 0xec5808, 0xed5a07, 0xee5c06,
                       0xee5d05, 0xef5f04, 0xef6104, 0xf06303, 0xf06503, 0xf16603, 0xf16803, 0xf16a02, 0xf16b02, 0xf26d01, 0xf36f01, 0xf37101, 0xf47300, 0xf47500, 0xf47700,
                       0xf47a00, 0xf57c00, 0xf57f00, 0xf68100, 0xf78300, 0xf78500, 0xf88700, 0xf88800, 0xf88a00, 0xf88c00, 0xf98d00, 0xf98f00, 0xf99100, 0xf99300, 0xfa9500,
                       0xfb9800, 0xfb9a00, 0xfc9d00, 0xfca000, 0xfda200, 0xfda400, 0xfda700, 0xfdaa00, 0xfdac00, 0xfdae00, 0xfeb000, 0xfeb200, 0xfeb400, 0xfeb600, 0xfeb900,
                       0xfeba00, 0xfebc00, 0xfebe00, 0xfec100, 0xfec300, 0xfec500, 0xfec700, 0xfec901, 0xfeca01, 0xfecc02, 0xfece03, 0xfecf04, 0xfed106, 0xfed409, 0xfed60a,
                       0xfed80c, 0xffda0e, 0xffdb10, 0xffdc14, 0xffde19, 0xffdf1e, 0xffe122, 0xffe226, 0xffe42b, 0xffe531, 0xffe638, 0xffe83f, 0xffea46, 0xffeb4d, 0xffed54,
                       0xffee5b, 0xffef63, 0xfff06a, 0xfff172, 0xfff17b, 0xfff285, 0xfff38e, 0xfff496, 0xfff59e, 0xfff5a6, 0xfff6af, 0xfff7b6, 0xfff8bd, 0xfff8c4, 0xfff9ca,
                       0xfffad1, 0xfffbd8, 0xfffcdf, 0xfffde5, 0xfffeeb, 0xfffef1, 0xfffff6]
flir_ironbow_number_of_colors = len(flir_ironbow_colors)
flir_ironbow_palette = displayio.Palette(flir_ironbow_number_of_colors)

for i in range(flir_ironbow_number_of_colors):
    flir_ironbow_palette[i] = flir_ironbow_colors[i]

thermal_group = displayio.Group()

def render_thermal_legend():
    thermal_slider = displayio.Bitmap(int(5 * zoom_scale), int(24 * zoom_scale), flir_ironbow_number_of_colors)
    slider_tilegrid = displayio.TileGrid(thermal_slider, pixel_shader=flir_ironbow_palette, x = int(32 * zoom_scale) + 5, y = 0)
    for h in range(int(24 * zoom_scale)):
        for w in range(int(5 * zoom_scale)):
            palette_val = flir_ironbow_number_of_colors - int((h / (24 * zoom_scale)) * flir_ironbow_number_of_colors)
            thermal_slider[w, h] = palette_val
    return slider_tilegrid

def render_thermal_bitmap(frame, min_temp, max_temp):
    thermal_data_frame = [0] * 768
    thermal_bitmap = displayio.Bitmap(32, 24, flir_ironbow_number_of_colors)
    for h in range(24):
        for w in range(32):
            thermal_palette_val = int((frame[h*32 + w] - min_temp) / (max_temp - min_temp) * flir_ironbow_number_of_colors)
            if (thermal_palette_val >= flir_ironbow_number_of_colors):
                thermal_palette_val = thermal_palette_val - 1
            thermal_bitmap[w, h] = thermal_palette_val

    thermal_bitmap_scaled = displayio.Bitmap(int(32 * zoom_scale), int(24 * zoom_scale), flir_ironbow_number_of_colors)
    bitmaptools.rotozoom(thermal_bitmap_scaled, thermal_bitmap, scale = zoom_scale)
    tile_grid = displayio.TileGrid(thermal_bitmap_scaled, pixel_shader=flir_ironbow_palette)  
    
    return tile_grid

def render_min_temp(min_temp):
    min_temp_text = label.Label(display_font, text=str(int(min_temp)), color=0x00FF00)
    min_temp_text.x = int(32 * zoom_scale) + int(5 * zoom_scale) + 15
    min_temp_text.y = 130
    return min_temp_text

def render_max_temp(max_temp):
    max_temp_text = label.Label(display_font, text=str(int(max_temp)), color=0x00FF00)
    max_temp_text.x = int(32 * zoom_scale) + int(5 * zoom_scale) + 15
    max_temp_text.y = 10
    return max_temp_text


thermal_slider = render_thermal_legend()
thermal_group.append(thermal_slider)
display.show(thermal_group)

data_frame = [0] * 768
try:
    while True:
        try:
            mlx_90640.getFrame(data_frame)
            max_temp = max(data_frame)
            min_temp = min(data_frame)
            thermal_data_frame = render_thermal_bitmap(data_frame, min_temp, max_temp)
            min_temp_text_area = render_min_temp(min_temp)
            max_temp_text_area = render_max_temp(max_temp)
            if (len(thermal_group) > 1):
                thermal_group.pop()
                thermal_group.pop()
                thermal_group.pop()
            thermal_group.append(thermal_data_frame)
            thermal_group.append(min_temp_text_area)
            thermal_group.append(max_temp_text_area)
            
        except ValueError:
            # these happen, no biggie - retry
            continue
except KeyboardInterrupt:
    pass
{% endhighlight %}

# And here's the final result:

### Camera all wired up
![Here's the camera all wired up](/images/tiny-thermal/tiny-thermal-full.webp)

### Close up of the display
![Close-up of the display](/images/tiny-thermal/tiny-thermal-close.webp)