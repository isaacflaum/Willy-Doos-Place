---
layout: post
title:  "Soil Moisture Level Sensor Pt. 2"
date:   2023-05-29 12:45:09 -0400
categories: update
---

I wasn't quite satisfied with the soil moisture sensor. What's the point of checking the moisture level automatically if we can't do anything about it? Let's add a small water pump to automatically water the plant when the soil is too dry! Here is my somewhat longer than expected journey.

I started with this snazzy little 3-Volt water pump (pictured below submerged in the water) 

![](/images/soil-moisture-2/okMotor.webp)

I immediately starting running into issues. Both the motor and the Pico's GPIO pins (the ones we can control with software) operate at 3-Volts so the Pico should be able to power the motor - right? Well power comes from both voltage and current, and I completely forgot about current! The 3-Volt water pump motor needs around 150mA (milli Amps) of current to operate. The Pico's digital pins however only typically supply 15mA or around 1/10th of the current needed to power the water pump! So while the Pico is capable of supplying the necessary voltage, it cannot supply the necessary current to drive the pump. Back to the drawing board. If we look at the pinout of the Raspberry Pi Pico, we only have software control over the GPIO pins (the green ones) which are all low current and operate at 3-Volts.

![](/images/soil-moisture-2/picoPinout.webp)

There are, however, three pins: 40, 39 and 36 that are all capable of supplying the necessary voltage and current to drive the motor. Unfortunately, we have no control over these pins turning on or off. They are always on if the Pico is powered and always off otherwise. Well that's not very useful - we want to use software to control when our water pump turns on and off. So we need to make sure:

 * Enough voltage and current can be supplied to the water pump
 * We can control powering the motor with our 3-Volt GPIO pins on the Raspberry Pi Pico


Introducing the MiniBoost from Adafruit to the rescue!
![](https://cdn-shop.adafruit.com/970x728/4654-03.jpg) This device is exactly what we are looking for - it will let us control a higher power signal with our Pico! We can use any of Pins 40, 39, or 36 to supply power to the MiniBoost through the `Vin` pin and then control the supply of power with the rightmost pin on the MiniBoost - labeled `En` for Enable. If we supply a 3V signal to the MiniBoost's enable pin, the MiniBoost passes the `Vin` signal through to the `5V` output pin. If we supply 0V through the enable pin, the MiniBoost does not pass `Vin` to `5V`. We can now wire the water pump to the `5V` output pin from the MiniBoost and control it with a few lines of code such as the following, in which the MiniBoost's enable pin is connected to the Pico's GP16 pin.

{% highlight python %}
water_pump_enable = digitalio.DigitalInOut(board.GP16)
water_pump_enable.direction = digitalio.Direction.OUTPUT

# Turn off water pump - sets GP16 to 0V
water_pump_enable.value = False

# Turn on water pump - sets GP16 to 3V
water_pump_enable.value = True
{% endhighlight %}

Eventually this did do the job. The soil moisture sensor communicated with the Pico to turn on or off the pump, but something was off. I'm not a huge fan of mixing water and electronics. The idea of the pump plus some wires having to be submerged in water didn't sit right. Sure it's only 3 Volts, but even the [seller doesn't recommend using it for long term projects - boo!](https://www.adafruit.com/product/4547)

So what can we do?!?!?! Peristaltic water pumps to the rescue! What the heck are those and why are they so useful? The word `Peristaltic` comes from `Peristalsis`
```
The wave-like muscle contractions that move food through the digestive tract. 
```
Similar to the human digestive tract, we can use a motor to expand and contract silicone tubing to push water (or other fluids)! Here is an example of the inside of a Peristaltic pump - note the 3 regions being `squeezed` by the rotating piece and how this pushes fluid through.
![](https://www.pumpsandsystems.com/sites/default/files/0718/flexflo_pumphead.jpg)

The advantage to Peristaltic pumps is durability since the motor and parts never come in contact with the liquid being pumped. a Peristaltic pump is exactly what we are looking for - no more mixing water and electronics! Here is the second take with the Peristaltic pump in the bottom right. Note that the pump is completely separate from the water this time!

![](/images/soil-moisture-2/betterMotor.webp)

I am quite pleased with the final result. Every 30 seconds we check the moisture level of the soil and turn on the pump for 10 seconds to water the plant :)

### Final Result
![](/images/soil-moisture-2/finalResult.webp)

### Aaaaand the Code

{% highlight python %}
import time
import digitalio
import board
import busio
from adafruit_seesaw.seesaw import Seesaw

SOIL_MOISTURE_THRESHOLD = 500

soil_sensor_i2c_bus = busio.I2C(scl = board.GP15, sda = board.GP14)
soil_sensor = Seesaw(soil_sensor_i2c_bus, addr=0x36)

water_pump_enable = digitalio.DigitalInOut(board.GP16)
water_pump_enable.direction = digitalio.Direction.OUTPUT
water_pump_enable.value = False

def water_plant():
    print("Watering plant...")
    water_pump_enable.value = True
    time.sleep(10)
    water_pump_enable.value = False

try:
    while True:
        print("Reading soil moisture level...")
        moisture_level = soil_sensor.moisture_read()
        print("Soil moisture level of " + str(moisture_level) + " detected")
        if (moisture_level < SOIL_MOISTURE_THRESHOLD):
            print("Soil moisture level low")
            water_plant()
        else:
            print("Soil moisture level ok!")
        time.sleep(30)
except KeyboardInterrupt:
    pass
{% endhighlight %}