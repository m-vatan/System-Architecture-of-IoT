# Lab 1 - Lab setup + MQTT with Arduino MKR WiFi1010

## Goals and Overview

The emerging and existing IoT frameworks share some central IoT architectural
features. A central pattern seems to be about discovering, connecting,
monitoring, controlling and/or interacting with a system of _smart things_ or
_Things_ (with capital T), where a Thing is an abstract notion of a digitally
augmented physical object. Digital augmentation is one or more combination of
sensors, actuators, computation element or communication interfaces attached to
the physical object. So, a Thing can be a circuit package with one sensor, or
multiple sensors and actuator, or even a group of devices mimicking a physical
object.

[Three
patterns](https://www.w3.org/Submission/wot-model/#web-things-integration-patterns)
emerge when it comes to binding the Things together. Direct Connectivity refers to
a pattern where each Thing is directly connected to another Thing. Each Thing
has its own application program interface (API) and they communicate with each
other using a standardised API. However, this is indeed the most difficult
approach due to diverse communication standards, lack of processing and
operational power on the devices to support full-fledged networking stack, and
lack of standardisation of the API.
[W3C](https://www.w3.org/Submission/wot-model) is coming up with one such
standard and is yet to be ratified. 

Gateway-based connectivity is the second pattern, where a Thing cannot offer an
API directly and sits behind an intermediary. The gateway mimics the Thing and
provides an API on behalf of the Thing. The last common pattern is the
cloud-based connectivity. This is similar to a gateway approach, the cloud
service provides the API for the Things. The difference lies in the fact that
the gateway can be physically closer to the Things. The interaction between
gateway/cloud and the Things themselves could be orchestrated through a tightly
coupled, proprietary communication protocols, that is lighter on power and
computational load when compared to full internet stack.

In the lab, we will focus on the cloud and gateway pattern. We will use MQTT
protocol to directly interact with the devices first and then subsequently
conduct the interactions through a gateway. In process, the goal is to
recognise some good practices when it comes to deploying a Thing network and
this is demonstrated using Amazon's IoT and Greengrass product. We will cover
common security practices that involve authorisation and authentication issues.
We will also touch upon elastic edge computing, if time permits.

Concretely, the lab exercises are designed to achieve the following learning
goals:

1. Lab 1: Get a working understanding of the MQTT protocol. Setup a rudimentary
   publisher and subscriber using a openly available broker.
2. Lab 2: Establish a secure MQTT connection and understand various parts of
   the setup
3. Lab 3: Establish a MQTT connection that is safe from (unintended) snooping
   and is private using a gateway.
4. Lab 4: Setup an edge - a local computing platform on the gateway for your
   devices to offload its computation. Connect a device to the edge and perform
   some computations. 

## Grading and Expectations

The lab work is graded based on a sumitted report team at the end of last Lab (one report for all the labs).
Each lab
exercise has a set of questions in the end that is to be answered, with the
goal of demonstrating working knowledge of concepts involved. You are
encouraged to provide any pictures, graphs, screenshots, diagrams etc. that
help understanding your solution. If you use content (picture, graphs, etc.)
from other sources, remember to properly cite and provide reference to the used
external sources. Add as appendix to your report the implemented source code (be sure your code is easily readable and understanble). 

At the end of the report you should also provide a reflection on
what you learned during this exercise. This section could provide answers to
the following questions:

1. Have you learned anything new?
2. Did anything surprise you?
3. Did you find anything challenging?
4. Did you find anything satisfying?

The report should contain full name and student number of each group member.

The expected size of this report is 8-12 pages of content.

The hard deadline for submission is ***31.03.2021 23:55***

## Setup your computer 
To perform lab exercises, you need to setup your computer. No extra computers will be provided and it is your responsibility to setup your computer.
It is assumed that you know some basic commands of linux.

For the Labs your will use a Virtualbox Ubuntu-based virtual machine (VM) as a development environment step. 
You should:

1. Download and install Virtualbox on your laptop: https://www.virtualbox.org/wiki/Downloads 
2. Download the VM from this link:
3. Import the downloaded VM into Virtualbox by selecting the downloaded .ova file. The current VM has 4GB of RAM - Feel free to adjust it based on your own laptop configuration
4. Check you can start the VM. 
- **The used username and password is iotlabs/iotlabs.**
- Feel free to change the screen resolution in ubuntu or to scale in/out the virtual screen (from the "screen logo" on the bottom left. 
- Feel free to create a shared folder between your physical machine and VM (Settings->Shared folder). The following mount point can be used: /home/iotlabs/SharedFolder
- The VM had Atom installed as a possible source code editor
   
(Navigate to `project_home/iot_labs` and run the following command. This will download ~1.6 GB virtual box consisting of the virtual environment. This will also create a file `id_rsa.pub` in your `project_home/iot_labs`. Finally, copy the content of the `id_rsa.pub` file from your `project_home/iot_labs` and upload it in moodle.)


## Lab 1 - Before you begin,

In this lab you will create a new IoT device using Arduino MKR WiFi1010 board.
The device will measure the room temperature and transmit to a remote server using
the MQTT protocol. You will establish an insecure link and measure the transmission
latency. You will also implement logic to perform actuation on the sensor based
on user commands.

For the lab we will use Arduino IDE: https://www.arduino.cc/en/Main/Software
The IDE is already avaialble from the provided VM and installed in the folder arduinoIDE, and should be start by executing the script _/.arduinoIDE/arduino_

Please do the following:


## Hardware and Software setup

### Arduino setup

Collect the board and the required components. You will need:

1. [Arduino MKR WiFi1010 board](https://store.arduino.cc/arduino-mkr-wifi-1010)
2. Breadboard
3. [Thermistor IC](http://ww1.microchip.com/downloads/en/DeviceDoc/20001942G.pdf)
4. Up to 5 breadboard connectors
5. A USB type A to USB micro cable
6. WiFi credentials for Arduino

Insert the Arduino to the breadboard. Make sure that Arduino pins are on either
sides of the central ridge of the breadboard and that all the pins are inserted
in the breadboard. (See the image below or ask me if you're unsure).

Now we will verify that Arduino IDE has read and write access to the Arduino
board.  On Arduino Create, create a **New Sketch** and paste the code from
the file `led.c` available from the folder Lab1. 

Verify, upload and save the sketch to the Arduino MKR WiFi 1010 board. If
everything goes well, you must see a yellow LED blinking on the board. You are done for the arduino setup as you verified you can compile and upload an exectable on the board.

## MQTT Client Setup

The overall architecture of the system you will implement is described in the figure bellow:
![](./figs/Lab1_Overview.png)


In order to subscribe to the messages that your Arduino board would send,
we need to setup a MQTT client on your desktop (the unbuntu VM in your case). The mosquitto MQTT client was already install on the provided VM (see https://mosquitto.org/download/)


## Arduino + Temperature Sensor

Now we are set to measure room temperature and transmit to a remote MQTT server.
First setup the hardware as shown in the circuit diagram below.

![](./figs/schematic_mkr_1000.png)

**IMPORTANT: Do not flip Vcc and GND connections. Also, do not short Vcc and GND.
Power the board via the USB cable AFTER you verify that the circuit is correct**

The setup itself should similar to the figure below. The thermistor IC's pin
numbers are determined by holding the flat end towards you, with pins facing
downwards and counting from left. From the schematic, you connect first pin of
the thermistor to third pin on Arduino marked as **Vcc**. Connect the second
pin of the thermistor to second pin on the Arduino situated on the other side
of the Vcc and is marked as **DAC0/A0**. Connect the third pin of the Arduino
to the fourth pin of the Arduino board, marked as **GND**. GND pin is on the
same side as Vcc. 


![](./figs/breadboard_mkr_1000.png)

Once the hardware is setup, create a new sketch and paste the code from the file `mqtt_unsecure.c` available from the Lab1 folder.
Have a look at the code and understand what it does before uploading the code.
Use the provided WiFi username and password. Remember to installe the ArduinoMqttClient library (Tools->manage Library)
Compile the code and upload it to the board. 

Verify the state of the Arduino board by connecting to the *Monitor* in the
IDE.  You can see the transmitted MQTT messages by subscribing to the topic
(read the code to get the relevant topic). **In your ubuntu VM machine**, run
the following code. Substitute `responseTopic` within the quotes (retain the quotes
later) with the topic your device is sending the messages into. You may have to
change the topic, so read the source code.

```bash
vagrant> mosquitto_sub -h broker.hivemq.com -t "responseTopic"
```

### To do

1. Read and understand the code. Specifically understand the `getTemp`
   function. Explain its working in your report. Consult the
   [datasheet](http://ww1.microchip.com/downloads/en/DeviceDoc/20001942G.pdf)
   if necessary. Get a working understanding of the
   [WiFiNINA](https://www.arduino.cc/en/Reference/WiFiNINA) and
   [ArduinoMqttClient](https://github.com/arduino-libraries/ArduinoMqttClient)
   library, although you don't have to explain the functionality in the report.
2. Measure the latency of transmission. Use the function `micros()` to return the current
state of the timer in microseconds. 
3. Does the length of the message have an impact on latency of transmission?
4. What is the value returned by mqttClient.messageQoS() ? What does it means ?
5. What is the value returned by mqttClient.messageRetain() ? What does it means ?

## Command and Reponse

The code also has a subscriber component built in. Have a look at the function
`onMqttMessage`. This is called when the device receives a message. To see it
working, open another terminal and do the following. Replace "commandTopic" with
the appropriate topic name and "command" with appropriate message.

```bash
vagrant ssh
vagrant> mosquitto_pub -h broker.hivemq.com -t "commandTopic" -m "command"
```

### To do

1. Modify the publisher to transmit messages only when an appropriate command
   is received.
2. Modify your subscriber to implement these two commands
    1. The **ON** command will turn on the onboard LED. 
    2. Similarly, **OFF** command will turn off the onboard LED.
    3. The **TEMP** command will send the temperature on the appropriate
       response topic, once.
    4. Any other command will not generate a response.

Hint: Check
[String](https://www.arduino.cc/reference/en/language/variables/data-types/stringobject/)
to create strings. Read character by character and append to the string. Don't
forget to terminate with a Null character (`'\0'`) in the end. 

## Wrap Up

Well, that's it for today. You can return the components. You can safely
disconnect the MQTT broker and power-off your VM (from the top right corner of the window, like if you would shout down a real machine)



