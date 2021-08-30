---
layout: post
title: Creating a Steering Wheel for a Solar Race Car
subtitle: Brizo's Steering Wheel
author: Jonathan Mullen
banner:
  image: /img/wheel_front.jpg
  opacity: 0.38
categories: solar car
tags:
  - solar car
  - steering wheel
  - embedded systems
  - racing
---

## Argo's Steering Wheel

As a part of [Illini Solar Car](https://www.illinisolarcar.com/), I worked on a lot of different systems on the car, but one system would become a consistent focus - the steering wheel. My first big firmware project on the team was fixing the firmware controlling the original steering wheel for [Argo](https://www.illinisolarcar.com/argo). At the 2017 World Solar Challenge, the car had been plagued by issues due to this firmware. Lack of sufficient input safety and an abundance of untested edge conditions resulted in unreliable and sometimes unpredictable behavior. We learned an important lesson that everything is much harder once you put it together on a solar car.

![Argo's Steering Wheel](/img/argo_wheel.jpg)
<p align="center"><i>Argo's Steering Wheel<br></i></p>

As I re-wrote this firmware I also became a driver for the [2018 American Solar Challenge](https://www.illinisolarcar.com/asc-2018). Being a driver provided me with the opportunity to get to use the steering wheel in action. This, along with feedback from other drivers, allowed me to create a much more usable and reliable system for our 2018 competition. Unlike 2017, we didn't once stop because of a problem with the steering wheel.

## Protoype New Steering Wheel

Building a new car allowed us to re-think the steering wheel system in its entirety. Developing the prototype of this steering wheel was my senior design project along with one partner, another electrical team member. We studied steering wheels from competitors and Formula-E vehicles and took feedback from our drivers about the previous version. This led to a long list of potential changes, most of which would end up incorporated into our new steering wheel.

![Argo's Steering Wheel](/img/formulae_wheel.jpg)
<p align="center"><i>A 2019 Formula E Steering Wheel<br></i></p>

Foremost among these was the major system architecture change to make the steering wheel a standalone controller. The previous steering wheel was an I2C hardware slave of the dashboard. This was a problem because our steering wheels are removable (using a hub common in Formula-1 cars) to assist with driver egress. I2C is not a hot-swappable protocol, so this caused issues that weren't fixable with software. Instead, the steering wheel became its own node on the CAN bus. This necessitated an increase in hardware complexity, but luckily the MCU and supporting hardware used on many of our other controllers was up for the task of the steering wheel too.

Other new hardware features in this prototype included the following. We more than doubled the supported number of buttons, allowing us to move many oft-used features onto the wheel. The dash display moved onto the steering wheel to improve the viewing angle and eliminate obstructions. The display was also changed to an OLED display, instead of a backlit LCD, to improve contrast and eliminate backlight burn issues that occurred after an extended time in direct sunlight. The isolated push-to-talk system (controlled "hands-free" from the steering wheel) was tied into an isolated sensing circuit to allow for radio diagnostics. Some considerations were also taken for analog regenerative braking and/or acceleration control (although full implementation would await the finalization of the vehicle drive control system).

From a firmware standpoint, the steering wheel also changed significantly. For one, it would have its own firmware, not be controlled from the dash. The firmware was written mostly from scratch. Taking advantage of this on-board firmware and new hardware we were able to do more driver feedback, better diagnostics, etc. This made this steering wheel a lot nicer to work with. For this prototype, the main firmware goal was to create a demo of all the features and hardware. This required basic drivers for all the new hardware - including the OLED Display and the new LED driver used for the 7-Segment displays.

I wrote nearly all of the prototype firmware and designed the display supporting hardware, while my partner handled most of the other hardware. I also did all of the CAD modeling for the enclosure, based on the constraints from the previous vehicle (new vehicle constraints were not yet complete). Not all of the features were implemented with the demo enclosure (for example, it only provides mounting for a handful of buttons) however the rest of the features were proven on the bench without the demo enclosure.

![Protoype Steering Wheel](/img/prototype_wheel.jpg)
<p align="center"><i>Prototype Steering Wheel for the then-unnamed 2nd Vehicle<br></i></p>

All the new features were successfully demonstrated in the lab connected to a bench solar car electrical system. Of course, it was not without issues - on the inside you would notice some bodge wires, everything didn't quite fit perfectly, and some firmware bugs popped up. But, as a prototype, this met the purpose of proving the concept and demonstrating a lot of the features that would become critical on the next car. This project was selected to the [ECE Illinois Senior Design Hall of Fame](https://courses.engr.illinois.edu/ece445/hall-of-fame.asp) (Spring 2019).


## [Brizo's](https://www.illinisolarcar.com/brizo) Steering Wheel

In my last semester of school, I continued this project in the [Advanced Digital Systems Lab](https://wiki.illinois.edu/wiki/display/ece395/Illini+Solar+Car+Steering+Wheel) (ECE 395) to turn the prototype into the final version for [Brizo](https://www.illinisolarcar.com/brizo). With a much more finalized vehicle design my first goal was to improve the packaging of the system. While the prototype was very similar to the steering wheel from Argo, in terms of the enclosure, I now was able to depart from this with known constraints for Brizo. The new design is still very much inspired by the design of Argo's and uses the same basic structure and materials - but it has some major differences.

First, I was able to negotiate enough space to make the steering wheel bigger which allowed the screen to be moved to the top (previously prevented by its width). It is also slightly taller and thicker, together this allowed the complexity of the electrical hardware packaging to be reduced significantly - allowing all the electronics to fit on one main PCB and some breakout boards for peripherals. Additionally, I had space to place all of the buttons in reasonable locations - including embedding one in each handgrip.

![Brizo's Steering Wheel](/img/wheel_front.jpg)
<p align="center"><i>Brizo's Steering Wheel<br></i></p>

Another major change is easily visible, the accelerator was moved to the steering wheel in the form of a thumbwheel encoder. Due to space restrictions, the vehicle was not able to have an accelerator pedal that would meet regulations, so the accelerator was moved to the steering wheel which is not uncommon in solar cars. Other, under the hood, changes included adding an accelerometer, a buzzer for audible alerts, more on-board sensors for diagnostics, and an improved power system including super-capacitors to maintain power over short disconnections of the steering wheel.

Once the first version of this hardware existed (a few minor revisions would be necessary) the firmware really began. I created the base and overall structure, but from there the firmware was much more than a one-person project - many team members contributed to the steering wheel firmware from little fixes to whole features. I focused on writing the low-level drivers, defining CAN communication and interactions with other controllers, and implemented some of the features. Specifically, I implemented the button handling system, all of the power management, startup/shutdown procedures, setting configurations and EEPROM memory, and the 7-Segment Displays. I also supported all of the features from an architectural and integration standpoint.

![The inside of Brizo's Steering Wheel](/img/wheel_inside.jpg)
<p align="center"><i>The Interior of Brizo's Steering Wheel<br></i></p>

While not quite all of the desired features were implemented and tested in time for the [2021 competition](https://www.illinisolarcar.com/asc-2021), all of the required features were present and worked reliably. The drivers were able to control more of the car without removing their hands from the steering wheel and our telemetry team was able to know more about what the driver was doing while they were in the car. Importantly, this steering wheel was not a performance detriment, like the previous one had been at times, it enhanced the performance of the vehicle by being easier to use and providing the driver with more control and information right at the tip of their fingers. Hopefully, the team will be able to add the rest of the quality of life type features to make it even easier for drivers and the support team at the next race in 2022.

<blockquote class="twitter-tweet tw-align-center"><p lang="en" dir="ltr">Brizo just stopped for the night in Lakin, Kansas. We had a couple of roadblocks today including a fairing door coming loose, but we quickly recovered and completed a succesful day of driving! Next stop: Colorado! 🏔 <a href="https://t.co/ukfYmCdfYl">pic.twitter.com/ukfYmCdfYl</a></p>&mdash; Illini Solar Car (@IlliniSolarCar) <a href="https://twitter.com/IlliniSolarCar/status/1423094672793260032?ref_src=twsrc%5Etfw">August 5, 2021</a></blockquote> 

<p align="center"><i>Brizo Racing at the 2021 American Solar Challenge<br></i></p>

![Hits](https://hitcounter.pythonanywhere.com/count/tag.svg?url=https%3A%2F%2Fjtmullen.github.io%2Fsolar%2Fcar%2F2021%2F08%2F28%2FSolar-Car-Steering-Wheel.html)

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 