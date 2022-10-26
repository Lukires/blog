---
layout: post
title:  "Autonomous vehicles: Training a donkey car through reinforcement learning in a simulation"
date:   2022-10-25
categories: post 
authors: Lukas Schilling, Till Wenke 
---
# An autonomous vehicle project at Univeristy of Tartu
- [An autonomous vehicle project at Univeristy of Tartu](#an-autonomous-vehicle-project-at-univeristy-of-tartu)
- [**1. Motivation**](#1-motivation)
- [**2. Donkey Simulation**](#2-donkey-simulation)
  - [**2.1 Donkey sandbox**](#21-donkey-sandbox)
  - [**2.2 Reinforcement learning integration**](#22-reinforcement-learning-integration)
    - [**2.2.1 Existing solution**](#221-existing-solution)
      - [**2.2.1.1 reference blog post**](#2211-reference-blog-post)
    - [**2.2.2 Expanded and updated solution**](#222-expanded-and-updated-solution)
      - [**2.2.2.1 Motivation for expanding the solution**](#2221-motivation-for-expanding-the-solution)
      - [**2.2.2.2 Problems with expanding the solution so far**](#2222-problems-with-expanding-the-solution-so-far)
      - [**2.2.2.3 Future plans for simulation**](#2223-future-plans-for-simulation)
      - [**2.2.2.3.1 Adding obstacles**](#22231-adding-obstacles)
- [**3. Simulation to real life**](#3-simulation-to-real-life)
  - [**3.1 Building the competition course in our simulation**](#31-building-the-competition-course-in-our-simulation)
  - [**3.2 Procedurally generated courses and challenges**](#32-procedurally-generated-courses-and-challenges)
  - [**3.3 Image processing**](#33-image-processing)
- [**4. Energy consumption**](#4-energy-consumption)
  - [**4.1 Does training in a simulation save energy?**](#41-does-training-in-a-simulation-save-energy)

<hr>

# **1. Motivation**
# **2. Donkey Simulation**
## **2.1 Donkey sandbox**
## **2.2 Reinforcement learning integration**
### **2.2.1 Existing solution**
#### **2.2.1.1 reference blog post**
### **2.2.2 Expanded and updated solution**
#### **2.2.2.1 Motivation for expanding the solution**

#### **2.2.2.2 Problems with expanding the solution so far**
Websocket issues
#### **2.2.2.3 Future plans for simulation**
#### **2.2.2.3.1 Adding obstacles**

# **3. Simulation to real life**
## **3.1 Building the competition course in our simulation**
## **3.2 Procedurally generated courses and challenges**
## **3.3 Image processing**

# **4. Energy consumption**
Training AI models is an extremly energy intensive task for example training ... is equivalent to ...
Besides that indiviual vehicles are even more of a threat to the climate crisis. As a consequence - let's give the project a little green twist and try to think about some approaches to minimize energy consumption both for model training and while driving. The latter might also include adjustments to the hard and software on our Donkey Car. Maybe even the model we use can also account for later battery usage of the car and can therefore have a positive impact on it.
The key to comparing and later on reducing energy consumption will be to monitor it. 
Firstly we would have to monitor it during training which can be significant as stated before but it might also be neglectable comparing it to the overall driving and life time of a potential fleet of cars that would make use of it. Nevertheless we should pay attention to it as soon as we start training our models in a later stage of the project.

Secondly and more importantly there is the engery usage of the car while driving. In the case of our Donkey Car power is supplied by a about 8 V, 1700 mAh Li-Po battery. In order to get its energy level and change we need to know the current capacity (C) and voltage (U) over time as we can get the energy (E) by E = C * U . So far the Donkey car does not provide any means to poll the battery capacity but retrieving the voltage is or better should be possible which we will dive into in the next section.

## How to retrieve voltage on Donkey car?
So how do we get it? First - where do we have to put our attention among all those circuits, cables, sensors and motors of the Donkey Car? We can see that the battery is plugged into the blue circuit board at the top of the car - this is the Robohat MM1 (https://robohatmm1-docs.readthedocs.io/en/latest/) which is a microcontroller made for robotics. In this case its main use is to send the steering signals from the Raspberry Pi to the motor. It is said to have a INA219 current sensor so voltage should be available for us. It also provides a bunch of pins (https://robohatmm1-docs.readthedocs.io/en/latest/hardware/pinout/) over which we can retrieve information from it such as the SERVO pins for the steering commands. The pin that we are interested in is the PA02-pin or BATTERY-pin (https://robohatmm1-docs.readthedocs.io/en/latest/guide/Circuitpython%20API/Circuit_Python_API/) - it seems to give us information about the voltage of our battery. But how to talk to it?

As you might remeber from the car setup there is this Circuit Python code running on the robohat. We can connect to the robohat to access the code. We are mainly intersted in the code.py file as this is the main file that is executed once power is provided for the robohat. It includes a setup and then run in an infinite loop to poll steering information from the Raspberry Pi which is also done by some of the pins that we introduced before. In the same manner we can retrieve values from the analog BATTERY-pin in a quite easy way (https://learn.adafruit.com/circuitpython-essentials/circuitpython-analog-in).

from analogio import AnalogIn
analog_in = AnalogIn(board.BATTERY)
print(analog_in.value)

The only problem now is that the values are from a range of 0 to 2^16 - clearly not a value range we expected for voltage. We have to interprete it as proportion of the maximum voltage. But still the values are around 4.7 V which is a little off from the around 7.5 V that we get from the multimeter - so still a issue to work on.

Finally what are our options to store those values and monitor while the car is driving in laps. Basically we see two options both with their very own challenges. We could send the voltage values to the Raspberry over pins using the UART protocol and store them right next to the recorded image data which would be the most favourable option. But we would have to think about how to poll the pin values from the Raspberry which is not done currently. In comparison the naive option that is currently partially working (just one voltage value can be written) is to store the results in a file on the robohat. One has to pay attention to that the robohat file-system is read-only thus we have to remount it so that it can write to itself (https://learn.adafruit.com/circuitpython-essentials/circuitpython-storage)(just create a boot.py file with "storage.remount('/',False)"). Downside: you loose write access to the hat from your computer, so it is getting less convenient to make further changes to the code. As you see, there is still some work to do.

Our roadmap would be: store the values conveniently - get the right values - also retrieve battery capacity.

## **4.1 Does training in a simulation save energy?**