# Blog
Here is the full blog of this project where we display our research and experiments.

## The Donkey Car
<p align="center">
<img src="Pictures/donkeycar.jpg" width="500">
</p>

Donkey Car is an open source, DIY self-driving platform. It focuses on providing enthusiasts and students with all the tools to experiment with deep learning, object detection and autonomous driving. Thanks to libraries like TensorFlow, Keras, OpenCV, users can train their own self driving model, based on imitation learning for instance, and within hours they can obtain a working autonomous vehicle, capable of taking corners and avoiding obstacles.

The Donkey Car can be bought as a full kit, or assembled by the user with 3D printed parts and some electronics like the Raspberry Pi 4, Robot Hat MM1, servos...

The full project is accessible and very well documented online.

## Setting up the car

For this project, we are working with the Donkey Car number 124. The car was used previously by other students, but we decided to flash a brand new image on the SD Card just to be sure we can start from scratch and tailor our environment for our project.

This setup is rather straightforward since Donkey Car provides a very step by step documentation on their website: http://docs.donkeycar.com

A small hiccup during the setup of the Raspberry Pi was that we misunderstood the network setting template and left some "<>" in the SSID and password definitions. It took us some time to figure out the mistake and without a working connection between the Pi and the router, we were not able to SSH into the car and perform further setup.

Our car being the number 124, we went with the hostname ```donkey-124```. So the car can be accessed through SSH like so:

```
ssh pi@donkey-124.local
```

Using a hostname makes things so much easier. On a small network, finding the IP associated with the car from the router's interface is doable, but on the university's network it's absolutely impossible. 

**Note:** our car is locked with a password so no one can access it while it's running via SSH. This doesn't prevent anyone from altering our SD card physically when the car is stored in its box of course...

## Calibrating the car

Once the car is setup and can be accessed via SSH, it can technically drive, but its direction must be calibrated to make sure it drives straight. To do so, there are two ways:

1. Using the command line, playing with different PWM values until we found one that's satisfying.

This method did not work for us, as it is complicated to understand which GPIO pin is used by the board to steer, and how to call it in the command.

2. Using the mobile app and installing all the necessary server software on the car.

This method worked so easily for us we almost regretted wasting time on the first method. The Donkey Car documentation breaks down the steps on how to setup the mobile app (client) and software (server) on the car.

http://docs.donkeycar.com/guide/deep_learning/mobile_app/

For the mobile app to detect the car (obviously connected on the same WiFi network), the server must be running on the car. To do so, here are some commands to be executed in the car via SSH:

```
cd /opt/donkeycar-console
python manage.py runserver 0.0.0.0:8000
```

From the app it's important to calibrate the steering but **ALSO** the throttle. We had a bad surprise and the car zoomed across the room breaking the camera mount. The car should lifted off the ground and the throttle PWM should be calibrated (reduced/increased) so that the car's wheels spin at a reasonable speed when the speed multiplier set by the controller is high.

## Driving the car

When the car is calibrated, we can actually start driving. Again, the Donkey Car documentation makes it really straight forward. Here are some commands to start driving the car:

```
cd ~/mycar
python manage.py drive --js
```

Note that the ``` --js ``` argument specifies that we want to drive with the controller that's chosen in the myconfig.py file! In our case we use the Logitech F710.

Pressing the **START** button until the terminal says "user" we set the car so that it expects thumbsticks input for throttle and steering. The car should move if the speed multiplier is set high enough with the D-pad's up and down buttons!

## Indoor localization 

Indoor localization poses a significant challenge as conventional tools such as GPS are not reliable in buildings. As a result, our project turned to WIFI localization as a potential solution. We discovered a project developed by a bachelor's student from Tartu University that utilized WIFI signals for positioning. You can find the code on his GitHub repository: https://github.com/tonysln/delta-wifi-pos.

This code uses the RSSI of various WIFI routers in the surrounding area to approximate the location of the device. For the interface, it uses Qt framework to show the interactive map with the user location.

We decided to start from this code and make modifications. We adapted the code to suit our application.

This is what the interface of the original code looks like :

<p align="center">
<img src="Pictures/app_interface1.png" width="900">
</p>

The image represents the second floor of the Delta building of Tartu university. Our current location is represented by the green point, which has been determined by our code, and the circle surrounding it is proportional to the uncertainty of our location.
We estimates the inaccuracy to be from 2 to 5 meters from our real position.

This first test is localizing our computer hosting the app. However, we want everything to run on the Donkey Car. So we went ahead and installed the full repository on the Raspberry Pi, but unfortunately, the package *Pyside6* which is responsible for the GUI and map of the Delta building cannot be installed on the single board computer (We thought we'd access the GUI running on the car via SSH with an -X argument). A workaround we came up with is installing an MQTT broker on the Pi, and sending the car's position as MQTT messages which a distant computer can grasp and place on the map. This way, the car can determine its position with respect to the routers, use it to navigate, and on our computer we're able to track the said position on the map. While we were at it, we also installed Node-RED on the Pi to take advantage of the MQTT receiver tool and debugging console.

The schematic below breaks down this process in case this explanation was not clear enough:

<p align="center">
<img src="Pictures/mqttserver.png" width="500">
</p>

Here is a video of us trying to locate the car on the computer, while we drive it manually:

https://www.youtube.com/watch?v=ZmvoQWlBWLI


With this method we can retrieve the car's location from the server and display it on the map, but there is a significant delay between the actual position of the car and the position shown on the map.The delay is likely caused by the time required for the code to scan all available routers and calculate the location approximation. This delay, combined with the imprecision, creates significant challenges in maintaining accurate real-time tracking of the car when driving autonomously.
