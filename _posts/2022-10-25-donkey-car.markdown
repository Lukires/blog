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
  - [**2.1 Self driving car sandbox**](#21-self-driving-car-sandbox)
  - [**2.2 Reinforcement learning integration**](#22-reinforcement-learning-integration)
    - [**2.2.1 OpenAI Gym Environment for Donkey Car**](#221-openai-gym-environment-for-donkey-car)
    - [**2.2.2 Expanding upon the solution**](#222-expanding-upon-the-solution)
      - [**2.2.2.3 Changing the throttle dynamically**](#2223-changing-the-throttle-dynamically)
      - [**2.2.2.4 Custom course and custom obstacles**](#2224-custom-course-and-custom-obstacles)
- [**3. Energy consumption**](#3-energy-consumption)
  - [**3.1 How to retrieve voltage on Donkey car?**](#31-how-to-retrieve-voltage-on-donkey-car)
- [4. Conclusion](#4-conclusion)
- [5. Bibliography](#5-bibliography)

# **1. Motivation**
We to train a model that can compete in the [ADL Minicar Challenge 2023](https://courses.cs.ut.ee/t/DeltaXSelfDriving/Main/HomePage) at the University of Tartu.
We thought it would be interesting to train a model in a simulation, using reinforcement learning, which is what this post will be focusing on.
We will walk through our discoveries and progress, as well as our future plan, in a way that should make it possible for readers to follow along and
reproduce our results.


# **2. Donkey Simulation**
To start, we need to find simulator, where we can train our car. For this we use [Self driving car sandbox](https://github.com/tawnkramer/sdsandbox). 
## **2.1 Self driving car sandbox**
[The self driving car sandbox](https://github.com/tawnkramer/sdsandbox) is great, because it is made for the donkey car. It is made in Unity, which means we can easily make additions and changes to the simulation, which we will get into a bit more further down. We have made a fork of the project, which is where all our changes to the simulator will be: [Self driving car sandbox fork](https://github.com/Lukires/gym-donkeycar)

## **2.2 Reinforcement learning integration**
Luckily we're not the first that wants to use reinforcement learning in the self driving car sandbox. Using the OpenAI [gym-donkeycar](https://github.com/tawnkramer/gym-donkeycar) environment for donkey car makes this a trivial task.

### **2.2.1 OpenAI Gym Environment for Donkey Car**
The OpenAI Gym repository has a reinforcement learning implementation based on [this blog post](https://flyyufelix.github.io/2018/09/11/donkey-rl-simulation.html) about implementing Double Deep Q Learning in the donkey simulator. Their implementation is a little old, and has a few issues with the latest versions of its libraries, which we have fixed in [our fork of the project](https://github.com/Lukires/gym-donkeycar). Starting the training is quite simple, once you have set up the project and the simulator, simply use ``python gym-donkeycar/examples/reinforcement_learning/ddqn.py --sim <path to simulator>``. Which starts the training as seen in the image below.
![DDQN training](../assets/donkey_ddqn_test.png)

### **2.2.2 Expanding upon the solution**
The DDQN implementation uses a constant throttle, meaning we are only training the steering. Some of the challenges in the [ADL Minicar Challenge 2023](https://courses.cs.ut.ee/t/DeltaXSelfDriving/Main/HomePage) require the ability to slow down and stop completely. Thus, it is important that our model is able to control its throttle. It also means that we need to create a course,
that can recreate the problems that require slowing down and stopping completely, which we could then use for training.

#### **2.2.2.3 Changing the throttle dynamically**
Adding adaptive throttling to our model turns out to be very easy, it is essentially just expanding the output space by 1 variable and sending this variable to our simulation as the car's throttle. However, while implementing it is easy, training the model becomes a lot harder. We also have to consider how throttle translates to speed, because the throttle's affect on speed can vary,
espcially for a real donkey car. We are still working on an optimal solution for this.

#### **2.2.2.4 Custom course and custom obstacles**
We are working on creating a course which will look like the [ADL Minicar Challenge 2023](https://courses.cs.ut.ee/t/DeltaXSelfDriving/Main/HomePage) competition course. Luckily, since the simulator is made in Unity, this becomes quite trivial, and we will push our finished course to [our fork of the simulator](https://github.com/Lukires/gym-donkeycar).
A big part of the custom course will also be custom obstacles. The current simulation implementation calculates a cross track error, as well as when was the last collision, and sends it to our model, which then uses it for evaluating its decisions. This is enough for following a road and avoiding obstacles, such as walls or pedrestians.
The ADL Minincar challenge, however, poses a few more challenges, such as stopping at a pedestrian crossing, waiting for a bit and then continuing. Luckily things like these are rather easy
to detect in Unity, and we can simply implement a pedestrian crossing component, and a service that watches whether the car stops before pedestrian crossings, and then send it to our model along with the cross track error and last collision information.

# **3. Energy consumption**
Training AI models alone is an extremly [energy intensive task](https://numenta.com/blog/2022/05/24/ai-is-harming-our-planet).
Besides that indiviual vehicles are even more of a [threat to the climate crisis](https://ourworldindata.org/co2-emissions-from-transport). As a consequence - let's give the project a little green twist and try to think about some approaches to minimize energy consumption both for model training and while driving. The latter might also include adjustments to the hard and software on our Donkey Car. Maybe even the model we use can also account for later battery usage of the car and can therefore have a positive impact on it.

The key to comparing and later on reducing energy consumption will be to monitor it. 

Firstly we would have to monitor it during training which can be significant as stated before but it might also be neglectable comparing it to the overall driving and life time of a potential fleet of cars that would make use of it. Nevertheless we should pay attention to it as soon as we start training our models in a later stage of the project.

Secondly and more importantly there is the engery usage of the car while driving. In the case of our Donkey Car power is supplied by a about 8 V, 1700 mAh Li-Po battery. In order to get its energy level and change we need to know the current capacity (C) and voltage (U) over time as we can get the energy (E) by E = C * U . So far the Donkey car does not provide any means to poll the current capacity but retrieving the voltage is or better should be possible which we will dive into in the next section.

## **3.1 How to retrieve voltage on Donkey car?**
So how do we get it? First - where do we have to put our attention among all those circuits, cables, sensors and motors of the Donkey Car? We can see that the battery is plugged into the blue circuit board at the top of the car - this is the [Robohat MM1](https://robohatmm1-docs.readthedocs.io/en/latest/) which is a microcontroller made for robotics. In this case its main use is to send the steering commands from the Raspberry Pi to the motor. It is said to have a [INA219 current sensor](https://robohatmm1-docs.readthedocs.io/en/latest/) so voltage should be available for us. It also provides a bunch of [pins](https://robohatmm1-docs.readthedocs.io/en/latest/hardware/pinout/) over which we can retrieve information from it such as the SERVO pins for the steering commands. The pins that we are interested in is the [PA02-pin or BATTERY-pin](https://robohatmm1-docs.readthedocs.io/en/latest/guide/Circuitpython%20API/Circuit_Python_API/)  - it seems to give us information about the voltage of our battery. But how to talk to it?

As you might remember from the [car setup](https://docs.donkeycar.com/guide/create_application/) there is this Circuit Python code running on the robohat. We can connect to the robohat via USB to access the code. We are mainly intersted in the *code.py* file as this is the main file that is executed once power is provided for the robohat. It includes a setup and then runs in an infinite loop to poll steering information from the Raspberry Pi which is also done by some of those pins. In the same manner we can [retrieve values from the¸ analog BATTERY-pin](https://learn.adafruit.com/circuitpython-essentials/circuitpython-analog-in) in a quite easy way .

```
from analogio import AnalogIn
analog_in = AnalogIn(board.BATTERY)
print(analog_in.value)
```

The only problem now is that the values are from a range of 0 to 2^16 - clearly not a value range we expected for voltage. We have to [interprete it](https://learn.adafruit.com/circuitpython-essentials/circuitpython-analog-in) as proportion of the maximum voltage. But still the values are around 4.7 V which is a little off from the around 7.5 V that we get from the multimeter - so still a issue to work on.

Finally, what are our options to store those values and monitor while the car is driving in laps? Basically we see two options both with their very own challenges. We could send the voltage values to the Raspberry over pins using the UART protocol and store them right next to the recorded image data which would be the most favourable option. But we would have to think about how to poll the pin values from the Raspberry which is not done currently. In comparison the naive option that is currently partially working (just one voltage value can be written) is to store the results in a file on the robohat. One has to pay attention to that the robohat file-system is read-only thus we have to [remount](https://learn.adafruit.com/circuitpython-essentials/circuitpython-storage) it so that it can write to itself (just create a *boot.py* file with *storage.remount('/',False)*). Downside: you loose write access to the hat from your computer, so it is getting less convenient to make further changes to the code. As you see, there is still some work to do.

Our roadmap would be: store the values conveniently - get the right values/ the values right - also retrieve current capacity.

# 4. Conclusion
While we have found a way to do reinforcement learning in a simulator, as we set out to do, we want to optimize it for the [ADL Minicar Challenge 2023](https://courses.cs.ut.ee/t/DeltaXSelfDriving/Main/HomePage). This means we will have to do implement a lot of customizations in the Unity simulator, which is still a work in progress.
We have also been able to somewhat track energy consumption, which would like to play around with a lot to figure out how to minimize consumption.

# 5. Bibliography
[ADL Minicar Challenge 2023](https://courses.cs.ut.ee/t/DeltaXSelfDriving/Main/HomePage)\
[Self driving car sandbox](https://github.com/tawnkramer/sdsandbox)\
[gym-donkeycar](https://github.com/tawnkramer/gym-donkeycar)