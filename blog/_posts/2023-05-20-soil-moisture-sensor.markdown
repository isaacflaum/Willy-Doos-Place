---
layout: post
title:  "Soil Moisture Level Sensor"
date:   2023-05-20 13:34:09 -0400
categories: update
---

I keep forgetting to water my plants until they start to show signs of wilting! Wondering what I could do, I eventually stumbled upon this [soil moisture sensor by Adafruit](https://www.adafruit.com/product/4026) which looks perfect for me! I ordered ~10 of the sensors which arrived this week. Eager to get a working prototype going, I headed over to Factory 3.

The sensors are under 10$ a pop and take advantage of the fact that the electrical conductivity and capacitance of soil changes as the moisture level changes. This gives us a basis for converting electrical capacitance (something we can measure with electrical components) into a soil moisture level reading that is easily understandable to humans. The sensor returns values from roughly 200 (very dry soil) to 2000 (very wet soil) and also returns a temperature reading of the soil.

Another advantage of the sensor is it uses the simple I2C interface only requiring 4 wires to use! For this project I don't need speed or fancy color displays so I went with a simple design with a Raspberry Pi Pico to read the sensor data and display it on a small screen.


### The Code
{% highlight python %}
import time
import busio
import board
import adafruit_ssd1306

from adafruit_seesaw.seesaw import Seesaw

soil_sensor_i2c_bus = busio.I2C(scl = board.GP1, sda = board.GP0)
soil_sensor = Seesaw(soil_sensor_i2c_bus, addr=0x36)

ssd1306_i2c = busio.I2C(scl = board.GP27, sda = board.GP26)
display = adafruit_ssd1306.SSD1306_I2C(128, 64, ssd1306_i2c)

display.fill(0)

while True:
    # read moisture level through capacitive touch pad
    moisture_level = soil_sensor.moisture_read()

    # read temperature from the temperature sensor
    temp = soil_sensor.get_temp()
    
    display.fill(0)
    display.text("Soil Sensor", 40, 0, 2)
    display.text("Soil Temp: " + str(temp), 0, 30, 2)
    display.text("Moisture Level: " + str(moisture_level), 0, 50, 2)
    display.show()

    time.sleep(1)
{% endhighlight %}

And after a bit of tinkering with pins and some minor coding errors I got a working prototype! Here it is below, reading the temperature and moisture level of a small potted plant. After watering the plant, I observe the moisture reading go up to ~1000 so the sensor is working!

### The Result
![](/images/soil-moisture/finalresult.png)