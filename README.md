# README.md

This is my project in which I wanted to be able to know when someone rings my mechanical door-bell, even when I'm out of earshot of the bell. 

I like the bell as it is, so I didn't want to replace it with something like a [Ring doorbell](https://nl-nl.ring.com/pages/doorbells).

So I needed something that detects motion at the bell-end, and is able to send this wirelessly to another device, which in turn can somehow let me know things are happening. I settled on a Pi Pico W with a tilt-switch as motion detection as the sender, attached to the doorbell, A Pi Zero acts as the receiver, attached to a regular light in my office on the other side of the house. The latter can also be a Pi Pico W or similar. If so, change the `mqttclient.py` code where needed. 

In my case the Pi Zero W (receiver) is also the one running the MQTT broker, and that the receiver and the Pi Pico W (sender) are on the same WiFi network. You could run the MQTT from another device, as long as this one is always on.  

----
# Hardware
## Sender
- [Raspberry Pi Pico W](https://www.raspberrypi.com/products/raspberry-pi-pico/)
- Some kind power supply (I used 4xAA [rechargeable batteries](https://stfn.pl/blog/06-pico-aa-batteries/)) 
[SW-520D tilt switch](https://www.otronic.nl/nl/sw-520d-helling-tilt-sensor.html). Depending on the power supply you need some kind of battery case. 
- Two jumper wires to attach the tilt-switch with. 
- (Optional) An on-off switch [I like this type](https://www.amazon.nl/CESFONJER-mini-tuimelschakelaar-marinevoertuig-tuimelschakelaar-instrumententafel/dp/B07J4KB38W?pd_rd_w=9Hm25&content-id=amzn1.sym.e72a5e35-8016-4887-8196-dbb0ef37d504&pf_rd_p=e72a5e35-8016-4887-8196-dbb0ef37d504&pf_rd_r=P4G9P1S46DFGANS9FQDW&pd_rd_wg=JdrYL&pd_rd_r=aa7b88f0-2827-42a5-a1ba-15d1b3f6562c&pd_rd_i=B07J4KB38W&ref_=pd_bap_d_grid_rp_0_1_ec_i&th=1)

Attach the tilt-switch unto the end of the wires. In my case they needed to be a few centimeters long, to provide the tilt-switch with enough range of motion to trigger whenever the bell rings. Attach the wired-up tilt-switch to any of the regular pins and a ground pin. Make sure to set that same pin in the `motionsensor.py` on line 12. In the code provided it's attached to GP11 (physical pin 15). Make sure the tilt-switch points upwards, otherwise it won't work. 

Wire the power supply to the VBUS and a ground pin. I wired the on-off switch on the ground wire in between the battery and the Pico: I only turn it on when I am expecting someone or a delivery. 

## Receiver
- Raspberry Pi, such as the [Zero W](https://www.raspberrypi.com/products/raspberry-pi-zero-w/), but any of the Pi's with Wifi could do in this case. The receiver code is written in Python, but can easily be modified to run in MicroPython. Just make sure that in that case, the MQTT broker is running from some other device. In this project, the broker runs on the same device as the receiver. 
- A [relays breakout board](https://www.kiwi-electronics.com/nl/twee-kanaals-5v-relais-module-911?search=relais). Alternatively, follow along in the excellent Udemy course by [Dr. Peter Dalmaris](https://www.udemy.com/course/raspberrypibc/) on how to make your own relays circuit. 
- Some power supply. I opted to use an old iPhone charger which from which the USB cable has it's 5V and ground wires connected to jumper wires. These connect to 5V and ground pins on the Zero. This was done because in my case ther was no space to connect the micro-USB connection. 
- A case to hold everything in. I used a box from a wrist-watch.
- Male and female power-sockets with some cabling.

Attach the relays to pin 21 on the receiver (or at least to the same pin as is in the `mqttclient.py` script).
Attach the ground cable of the power sockets to the relays (I opted for the NC channel, so that if my relays somehow failed, the light would still be on). 

Attach the power supply to the receiver.

Attach a light or something useful to you to the female power socket of the receiver. It will toggle off-on a few times in response to the MQTT signal. 

----
# Battery life

Through testing I found that I could get the best battery life of the sender using a setup with 4 rechargeable AA batteries. This provides between 50 and 60 hours of battery life.

The Wifi only connects once motion is detected, and then only stays on for a couple of minutes. This does delay the response between sender and receiver (it is between 5-10 seconds before the receiver starts toggling).

This is also why I added the external on-off switch to the sender. I want to be able to turn it off when not needed. The downside is that now I have to remember how many cummulative hours it has been on since last charge.

----
# Todo

In the wifi_connect.py you need to fill out your WiFi SSID and password.

Setup the Mosquitto broker. A good explanation on how to set this up on the Pi can be found [here](http://www.steves-internet-guide.com/install-mosquitto-linux/). In both the mqtthandler.py (on the sender) and the mqttclient.py (on the receiver) you need to fill out the IP-address of your MQTT broker.

## Running as the mqttclient.py as a service on the PiZero

On the receiver, put the `mqttclient.py` in `/usr/bin/`. 

Copy the systemd file mqttclient.service to: `/etc/systemd/system/mqttclient.service`

Reload, start, and enable
`sudo systemctl daemon-reload`
`sudo systemctl enable mqttclient.service`
`sudo systemctl start mqttclient.service`

Your sender will probably crash if the broker is not yet on. Make sure the broker is operational before turning on the sender. 


------
# Usage

Hit the switch on the sender. Once the sender detects motion, it will establish a WiFi connection (slow LED blinks during connection, rapid blinks once connected). Then it will blink quickly again to indicate motion has been detected, and it will let the receiver know via the MQTT protocol. The WiFi connection stays active for a couple of minutes, then shuts down to conserve battery. 

The receiver receives the signal from the MQTT, and will toggle the relays on and off a few times. If a light is attached this will flicker. 


----
# Project plans

For now the project is at a good stage: I got it where I wanted it to be, and it is quite useful to me. 

Some future ideas might include:
- Improving the battery life of the sender from 50-60 hours to weeks or months
- Reducing the footprint of the sender
- Improving the response time of the sender/receiver connection from 5-10 sec to 1-2 sec
- Swapping the receiver Pi Zero for another Pi Pico W

Any ideas and improvements are welcome!
You are free to use and modify this project to your heart's content. 
Wherever applicable I noted in the scripts where I got the original code from.

