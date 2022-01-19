---
toc: true
title: Develop Android app to park a camper level
description: How to develop an Android app using a MPU6050 and a Raspi to help parking a camper level
categories: [Raspi,  MPU6050, Android]
image: images/vsCodeSreenShot.png
comments: true
---
# Develop Android app to park a camper level


--- 

In the blog [Rotationssensor MPU-6050 mit WebGL am Raspberry Pi visualisieren](https://tutorials-raspberrypi.de/raspberry-pi-mpu-6050-rotationssensor-webgl-nodejs-server/)
>Der Raspberry Pi ist zu vielem in der Lage, so können einfach Rotations- und Beschleunigugnswerte mittels eines Sensors, wie dem MPU-6050, ausgelesen werden. Das Ergebnis sind jedoch einfache Zahlen, worunter man sich im Normalfall nicht all zu viel vorstellen wird. Jedoch ist es sehr einfach diese Zahlen zu visualisieren. Dies geht in modernen Browsern ganz einfach mittels WebGL, womit man 2D und 3D Objekte im Browser rendern kann.

 <figure>
  <img src="images/MpuWiringRapsi.png" width="20%" height="20%" class="center">
  <figcaption>Sensor MPU 6050</figcaption>
</figure> 


## VNC: Remote access a Raspberry Pi

Setting up a VNC connection from phone to the raspi follow the instructions given at

https://magpi.raspberrypi.org/articles/vnc-raspberry-pi



## Setting up a Raspberry Pi as a Wireless Access Point

Setting up the raspi as a wireless access point so that you can use VNC on a camping ground follow instructions given at https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md 
This documentation assumes that we are using the standard 192.168.x.x IP addresses for our wireless network, so we will assign the server the IP address 192.168.4.1. 

**Note:** Change default in `sudo nano /etc/hostapd/hostapd.conf` to something meaningful to you

```
ssid=NameOfNetwork # change this
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=AardvarkBadgerHedgehog # change this
```

## Measuring Rotation and acceleration with the Raspberry Pi

Follow instructions at **MPU6050 (Accelerometer+Gyroscope) Interfacing with Raspberry Pi** https://www.electronicwings.com/raspberry-pi/mpu6050-accelerometergyroscope-interfacing-with-raspberry-pi

## Calculate pitch and roll

from `MPU6050_9Axis_MotionApps41.h`


```uint8_t MPU6050::dmpGetYawPitchRoll(float *data, Quaternion *q, VectorFloat *gravity) {
    // yaw: (about Z axis)
    data[0] = atan2(2*q -> x*q -> y - 2*q -> w*q -> z, 2*q -> w*q -> w + 2*q -> x*q -> x - 1);
    // pitch: (nose up/down, about Y axis)
    data[1] = atan(gravity -> x / sqrt(gravity -> y*gravity -> y + gravity -> z*gravity -> z));
    // roll: (tilt left/right, about X axis)
    data[2] = atan(gravity -> y / sqrt(gravity -> x*gravity -> x + gravity -> z*gravity -> z));
    return 0;
```

The following output 

```
Gyroscope
--------
gyroscope_xout:   -260  scaled:  -2
gyroscope_yout:   -154  scaled:  -2
gyroscope_zout:     78  scaled:  0

Accelerometer
---------------------
acceleration_xout:   -1048  scaled:  -0.06396484375
acceleration_yout:    -676  scaled:  -0.041259765625
acceleration_zout:   16644  scaled:  1.01586914062
X Rotation:  -2.32121150537
Y Rotation:  3.59994842011
```



is created by   
[Measuring Rotation and acceleration with the Raspberry Pi](http://www.raspberrypirobotics.com/measuring-rotation-and-acceleration-with-the-raspberry-pi/)  

```python
#!/usr/bin/python
import smbus
import math
 
# Register
power_mgmt_1 = 0x6b
power_mgmt_2 = 0x6c
 
def read_byte(reg):
    return bus.read_byte_data(address, reg)
 
def read_word(reg):
    h = bus.read_byte_data(address, reg)
    l = bus.read_byte_data(address, reg+1)
    value = (h << 8) + l return value def read_word_2c(reg): val = read_word(reg) if (val >= 0x8000):
        return -((65535 - val) + 1)
    else:
        return val
 
def dist(a,b):
    return math.sqrt((a*a)+(b*b))
 
def get_y_rotation(x,y,z):
    radians = math.atan2(x, dist(y,z))
    return -math.degrees(radians)
 
def get_x_rotation(x,y,z):
    radians = math.atan2(y, dist(x,z))
    return math.degrees(radians)
 
bus = smbus.SMBus(1) # bus = smbus.SMBus(0) fuer Revision 1
address = 0x68       # via i2cdetect
 
# Aktivieren, um das Modul ansprechen zu koennen
bus.write_byte_data(address, power_mgmt_1, 0)
 
print "Gyroscope Sensor"
print "--------"
 
gyroscope_xout = read_word_2c(0x43)
gyroscope_yout = read_word_2c(0x45)
gyroscope_zout = read_word_2c(0x47)
 
print "gyroscope_xout: ", ("%5d" % gyroscope_xout), " scaled: ", (gyroscope_xout / 131)
print "gyroscope_yout: ", ("%5d" % gyroscope_yout), " scaled: ", (gyroscope_yout / 131)
print "gyroscope_zout: ", ("%5d" % gyroscope_zout), " scaled: ", (gyroscope_zout / 131)
 
print
print "Accelerometer Sensor"
print "---------------------"
 
acceleration_xout = read_word_2c(0x3b)
acceleration_yout = read_word_2c(0x3d)
acceleration_zout = read_word_2c(0x3f)
 
acceleration_xout_scaled = acceleration_xout / 16384.0
acceleration_yout_scaled = acceleration_yout / 16384.0
acceleration_zout_scaled = acceleration_zout / 16384.0
 
print "acceleration_xout: ", ("%6d" % acceleration_xout), " scaled: ", acceleration_xout_scaled
print "acceleration_yout: ", ("%6d" % acceleration_yout), " scaled: ", acceleration_yout_scaled
print "acceleration_zout: ", ("%6d" % acceleration_zout), " scaled: ", acceleration_zout_scaled
 
print "X Rotation: " , get_x_rotation(acceleration_xout_scaled, acceleration_yout_scaled, acceleration_zout_scaled)
print "Y Rotation: " , get_y_rotation(acceleration_xout_scaled, acceleration_yout_scaled, acceleration_zout_scaled)
```

## Sending data to an HTTP server - get and post methods

[Sending data to an HTTP server - get and post methods](http://raspberrywebserver.com/cgiscripting/sending-data-to-an-HTTP-server-get-and-post-methods.html#disqus_thread)

## MQTT
[Eclipse Mosquitto™](http://mosquitto.org/)
An open source MQTT broker
Mosquitto is highly portable and available for a wide range of platforms. For Raspberry Pi
Mosquitto is available through the main repository.

## Introduction to IoT: Build an MQTT Server Using Raspberry Pi

https://appcodelabs.com/introduction-to-iot-build-an-mqtt-server-using-raspberry-pi
