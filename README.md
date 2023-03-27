# AVP-Localization
University of Tartu - Autonomous Vehicle Project

This repository is dedicated to the **Autonomous Vehicle Project** conducted at the University of Tartu by Robotics & Computer Engineering MA students Gautier Reynes and Malcom Radigon.

This project is based on the *Donkey Car* platform and tackles the subject of **localization** and **autonomous**, **free-range**, driving in the Delta building in Tartu.

Our first intuition is to use RSSI to localize the moving car across the Delta building. As a matter of fact, WiFi routers are scattered around the entire building, and we would like to test how good WiFi localization is, and if we can implement it within the Donkey Car framework. Then, we would have a Donkey Car that is able to drive autonomously by controlling its own steering and speed, while avoiding obstacles, but a higher command would force the car to initiate turns to follow a route from a start position to the end goal. The car is embedded with an IMU sensor which used as a compass would ensure the car's heading towards the right direction.

A priori this approach seems challenging and we will document our research and experiments in the blog linked below.

[Blog](/Documentation/Readme.md)
