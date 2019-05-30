---
title: Wemos D1 mini - First use
layout: single
author_profile: true
read_time: true
comments: null
share: true
categories:
  - IoT
tags:
  - iot
  - ESP8266
  - Wemos D1 mini
---

My initial idea was to make a compact device with as much sensor as I can buy. But first I need to try out how the D1 mini works with the Tasmota rom. So I opend up the [wiki page](https://github.com/arendst/Sonoff-Tasmota/wiki/Wemos%20D1%20Mini) and started to add items to my list:
* [Wemos D1 mini](https://www.banggood.com/Wemos-D1-Mini-V3_0_0-WIFI-Internet-Of-Things-Development-Board-Based-ESP8266-4MB-p-1264245.html?rmmds=myorder&cur_warehouse=CN&p=4J240714553741201709&custlinkid=138851)
* [AM312 motion sensor](https://www.banggood.com/Mini-IR-Infrared-Pyroelectric-PIR-Body-Motion-Human-Sensor-Detector-Module-p-1015337.html?rmmds=myorder&cur_warehouse=CN&p=4J240714553741201709&custlinkid=138857)
* [HC-SR501 motion sensor](https://www.banggood.com/HC-SR501-Human-Infrared-Sensor-Module-Including-Lens-p-972697.html?rmmds=myorder&cur_warehouse=CN&p=4J240714553741201709&custlinkid=138852)
* [BME280 temp.-humidity-pressure sensor](https://www.banggood.com/BME280-Digital-Sensor-Temperature-Humidity-Atmospheric-Pressure-Sensor-Module-p-1354769.html?rmmds=myorder&cur_warehouse=CN&p=4J240714553741201709&custlinkid=138854)
* [TSL2561 light sensor](https://www.banggood.com/3Pcs-GY-2561-TSL2561-Luminosity-Sensor-Breakout-Infrared-Light-Sensor-Module-Integrating-Sensor-AL-p-1145233.html?rmmds=myorder&cur_warehouse=CN&p=4J240714553741201709&custlinkid=138856) (its a 3 pack 1 would cost nearly the same)
* [Breadboard pack](https://www.banggood.com/Power-Supply-Module-830-Hole-Breadboard-Resistor-Capacitor-LED-Kit-For-Arduino-Beginner-p-1133908.html?rmmds=myorder&cur_warehouse=CN&p=4J240714553741201709&custlinkid=138858)

You can also buy and try:
* Ultrasonic sensor (not really useful for me)
* Dust sensor (pricey, not really useful)
* CO2 sensor (pricey, maybe I will buy one later)
* UV sensor (I didn't find compatible, maybe later hack one into the rom)
* RFID reader (not really useful for me, already have 10 (you don't want to ask))
* Rgb shield 
* Led strip
* Motor control
* I thinking about some kind of accumulator + solar panel combo too but not yet

I borrowed a cheap soldering kit from my friend, and I bought a cheap multimater from a local store. I bought two kind of motion sensors, because the reviews were mixed, and I want to try both.

# Assembling
I didn't want to rush the building and testing, but because I borrowed the soldering kit (I really will need to buy one), I wanted to solder + test it in a small time window.

So soldering... I never did that alone. My first 4 pin took about half an hour. In contrast my remaining 30 pin took about the same. My conclusion is simple: **If you are a first-timer too; watch a youtube video.** I watched [this one](https://www.youtube.com/watch?v=IpkkfK937mU), and kudos for the creator, because finally a tutorial which starts with a "why you struggle" part.

I'm not a natural with soldering. I had some bad feelings about some probably not well connected pins. I soldered the D1 legs, a light sensor, and the BME280 (temp). The motion sensors came with legs, so I didn't used them yet.  A really useful idea to test things separately. Lets try them one-by-one.

# Blinking led
As a programmer I know that every language has its own "hello world" program which is mostly for configuring the tools. In the Wemos D1 case it is a simple code which makes the onboard led blinking.

I started with [this tutorial](https://www.instructables.com/id/Wemos-ESP8266-Getting-Started-Guide-Wemos-101/) to install the software, and load my first program. It was suprisingly easy. The steps are:
* (Installing drivers may be neccessary on windows, but for mac it went smoothly without them.)
* [Get & install Arduino IDE](https://www.arduino.cc/en/main/software)
* Arduino > Preferences > Additional Boards Manager URL
  * paste this: `http://arduino.esp8266.com/stable/package_esp8266com_index.json`
* Tools > Board > Board Manager
	* write to the filter: `esp8266`
	* install it
* Tools > Board > Wemos D1 R1
* Scetch > Include Libraries > Library manager
	* we can install a ton of library here, I already list the ones for the later chapters
	* ESP8266 Microgear
	* Adafruit BME280
	* Adafruit TSL2561
	* Adafruit unified sensor
* Connect the board with USB
* Tools > Port > Choose the good port
	* on unix based systems the name could help
	* on windows if you have multiple COM ports after connecting your device probably the biggest port
* File > Examples > ESP8266 > Blink
	* a new window will open up
	* click to the verify
	* click to the upload
	* (maybe you need to change com port)
	* (I got strange errors with "something something not writeable", typically an usb reconnect solved them)
	* the blue led should blink on your board
	* but probably it already blinked before you did anything...
	* there are two `delay()` calls in the scetch, try to modify the values (for example to 200 and 2000)
	* upload the modified program
	* it's blinking differently this time

# BME280
Starting with the temperature, humidity, and preassure sensor. Most of the sources didn't mention the proper wireing. About the 8th page told me how I really need to wire this thing.

| sensor | D1 | info |
| ----- | ----- | ----- |
| VCC | 3.3V | power |
| GND | GND | ground |
| SCL | D4 (gpio2) | I2C clock |
| SDA | D3 (gpio0) | I2C data |
| CSP | 3.3V | mode selection (h->I2C, l->SPI) (we want I2C) |
| SDO | 3.3V | address selection (h->0x077, l->0x076) |

After the cabeling lets flash up some testcode:
* In the Arduino studio File > New
* copy-paste this to the new window
 {% gist 747ed7aca638d667d07f7b5d62cbef79 %}
* verify and upload it
* Tools > Serial Monitor
* Choose 9600 baud from the dropdown
* There are two possible outcomes
	* Everything works, and you start to see readings from the sensor
	* `Could not find a valid BME280 sensor, check wiring!`
		* If you get this start with the bare minimum, unwire everything which is not the sensor
		* Check if you are using a good pins everywhere
		* Check if all the used pins are soldered correctly

# Light sensor
The light sensor actually worked for me for the first try! Start with unwireing everything!

| sensor | D1 | info |
| ----- | ----- | ----- |
| VIN / VCC | 3.3V | power |
| GND | GND | ground |
| SCL | D1 (gpio5) | I2C clock |
| SDA | D2 (gpio4) | I2C data |
| INT | - | interrupt pin, leave it alone |

After the cabeling lets flash up some testcode:
* In the Arduino studio File > New
* copy-paste this to the new window
 {% gist 66525472f69718d28224f4d0278112f0 light-sensor-test.ino  %}
* verify and upload it
* watch the serial as before

If you just mindlessly copypasted gists to the board there is the time to actually read them. We have an onboard led. We have light measures. What if we add some business logic? For example my light sensor measures 30+ lux normally and 10- if I form a shadow abowe it. So what if I switch up the led when the sensor measures less than 15 lux and switch down when it measures more? You can try to do it yourself. I think this is where the fun begins! But for reference here is my code:
{% gist 66525472f69718d28224f4d0278112f0 light-sensor-test-v2.ino %}


# Connecting both
For the first sensor we used D1, D2, for the second we used D3, D4, why we can't use them both at the same time? I don't know. But after some try I started to being courious about what I2C is. And when I finally started to read about the technology  I started to use, I found out that I need to use either D1-D2 or D3-D4 for **both** sensors. (I picked D3-D4, not a really smart idea, because the on-board led is seems to be on the D4 port, so my board is flashing... Maybe I will move them to D1-D2.)

I have no testcode, because I only reached this point after trying out the Tasmota rom.

# Final thoughts
I think it was fun. At the university we get boring, not really useable tasks with microcontrollers. But this is working, and I made it. Totally a good feeling.
The bad part was when I realized that some youtube video at the beginning would make this whole process more smoother. But at least I learned something new!
