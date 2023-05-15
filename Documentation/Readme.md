**Blog**
====
# Project
This project is based on the *Donkey Car* platform and tackles the subject of **localization** and **autonomous**, **free-range**, driving in the Delta building in Tartu.

Our first intuition is to use RSSI to localize the moving car across the Delta building. As a matter of fact, WiFi routers are scattered around the entire building, and we would like to test how good WiFi localization is, and if we can implement it within the Donkey Car framework. Then, we would have a Donkey Car that is able to drive autonomously by controlling its own steering and speed, while avoiding obstacles, but a higher command would force the car to initiate turns to follow a route from a start position to the end goal. The car is embedded with an IMU sensor which used as a compass would ensure the car's heading towards the right direction.

Here is the full blog of this project where we display our research and experiments.

# The Donkey Car
<p align="center">
<img src="Pictures/donkeycar.jpg" width="500">
</p>

Donkey Car is an open source, DIY self-driving platform. It focuses on providing enthusiasts and students with all the tools to experiment with deep learning, object detection and autonomous driving. Thanks to libraries like TensorFlow, Keras, OpenCV, users can train their own self driving model, based on imitation learning for instance, and within hours they can obtain a working autonomous vehicle, capable of taking corners and avoiding obstacles.

The Donkey Car can be bought as a full kit, or assembled by the user with 3D printed parts and some electronics like the Raspberry Pi 4, Robot Hat MM1, servos...

The full project is accessible and very well documented online.

# Setting up the car

For this project, we started working with the Donkey Car number 124. The car was used previously by other students, but we decided to flash a brand new image on the SD Card just to be sure we can start from scratch and tailor our environment for our project.

**However, the steering angle of the car was terrible and we ended up not being able to access the car at all (corrupted OS, broken hardware...?). For that reason we were given the car number 260 instead. This new car worked flawlessly for the rest of the project.**

This setup is rather straightforward since Donkey Car provides a very step by step documentation on their website: http://docs.donkeycar.com

A small hiccup during the setup of the Raspberry Pi was that we misunderstood the network setting template and left some "<>" in the SSID and password definitions. It took us some time to figure out the mistake and without a working connection between the Pi and the router, we were not able to SSH into the car and perform further setup.

Our car being the number 260, we went with the hostname ```donkey-260```. So the car can be accessed through SSH like so:

```
ssh pi@donkey-260.local
```

Using a hostname makes things so much easier. On a small network, finding the IP associated with the car from the router's interface is doable, but on the university's network it's absolutely impossible. 

**Note:** our car is locked with a password so no one can access it while it's running via SSH. This doesn't prevent anyone from altering our SD card physically when the car is stored in its box of course...

# Calibrating the car

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

# Driving the car

When the car is calibrated, we can actually start driving. Again, the Donkey Car documentation makes it really straightforward. Here are some commands to start driving the car:

```
cd ~/mycar
python manage.py drive --js
```

Note that the ``` --js ``` argument specifies that we want to drive with the controller that's chosen in the myconfig.py file! In our case we use the Logitech F710.

Pressing the **START** button until the terminal says "user" we set the car so that it expects thumbsticks input for throttle and steering. The car should move if the speed multiplier is set high enough with the D-pad's up and down buttons!

# Indoor localization 

Indoor localization poses a significant challenge as conventional tools such as GPS are not reliable in buildings. As a result, our project turned to WIFI localization as a potential solution. We discovered a project developed by a bachelor's student from Tartu University that utilized WIFI signals for positioning. You can find the code on his GitHub repository: https://github.com/tonysln/delta-wifi-pos.

This code uses the RSSI of various WIFI routers in the surrounding area to approximate the location of the device. For the interface, it uses Qt framework to show the interactive map with the user location.

We decided to start from this code and make modifications. We adapted the code to suit our application.

This is what the interface of the original code looks like :

<p align="center">
<img src="Pictures/app_interface1.png" width="900">
</p>

The image represents the second floor of the Delta building of Tartu university. Our current location is represented by the green point, which has been determined by our code, and the circle surrounding it is proportional to the uncertainty of our location.
We estimate the inaccuracy to be from 2 to 5 meters from our real position.

This first test is localizing our computer hosting the app. However, we want everything to run on the Donkey Car. So we went ahead and installed the full repository on the Raspberry Pi, but unfortunately, the package *Pyside6* which is responsible for the GUI and map of the Delta building cannot be installed on the single board computer (We thought we'd access the GUI running on the car via SSH with an -X argument). A workaround we came up with is installing an MQTT broker on the Pi, and sending the car's position as MQTT messages which a distant computer can grasp and place on the map. This way, the car can determine its position with respect to the routers, use it to navigate, and on our computer we're able to track the said position on the map. While we were at it, we also installed Node-RED on the Pi to take advantage of the MQTT receiver tool and debugging console.

The schematic below breaks down this process:

<p align="center">
<img src="Pictures/mqttserver.png" width="500">
</p>

Here is a video of us trying to locate the car on the computer, while we drive it manually:

https://www.youtube.com/watch?v=ZmvoQWlBWLI


With this method we can retrieve the car's location from the server and display it on the map, but there is a significant delay between the actual position of the car and the position shown on the map.The delay is likely caused by the time required for the code to scan all available routers and calculate the location approximation. This delay, combined with the imprecision, creates significant challenges in maintaining accurate real-time tracking of the car when driving autonomously.

Ultimately, in order to achieve the desired results, we had to improve the scanning time of the network as the original method was too inconsistent. Our initial attempts involved scanning only wireless access points with a specific frequency, such as all those operating at 2.4GHz. By implementing this method, we were able to reduce the scanning time from arround 3.7 sec to 0.07 sec. Despite this improvement, we still faced issues with uncertainty.

Here is a video demo of our sped up localization:

https://youtu.be/D-p2JqKJJ3Y

# IMU

Having a way to localize the car at hand, we then needed to measure the car's heading so that we could implement some path planning from the vehicle's current location to a goal point (in other words we wanted the car to have its own compass). The MM1 Hat that is installed on the car has a MPU9250, an IMU (Inertial Measurement Unit) composed of a gyroscope, accelerometer and a complementary magnetometer. We tried to take advantage of this sensor using the very famous i2cdevlib module for Python developed by J. Rowberg (https://github.com/jrowberg/i2cdevlib) but the sensor is wired onto the board in a way that makes it difficult to use. Instead, we used our own MPU6050 IMU which is a downgraded version of the MPU9250 without the magnetometer.

The issue with the lack of magnetometer is that we needed to implement a compass for the car, and using the accelerometer/gyroscope combo alone can only do the trick to some extent. Z-drift is very noticeable overtime but thankfully J.Rowberg thought about it and an interrupt pin on the MPU6050 coupled with a DMP (Digital Motion Processor) embedded on the sensor allows some digital filtering and the drift is significantly attenuated.

<p align="center">
<img src="Pictures/imu.jpg" width="500">
</p>

We mounted the MPU6050 at the front of the Donkey Car, and the sensor itself is connected to a Wemos D1 mini (ESP8266) micro-controller (MCU). The MCU reads the IMU and sends the Z rotation as Euler angle through Serial (USB). Then we have a python script that reads the Serial port on the Raspberry Pi and extracts the Z orientation. From there we can process the value and use it for navigation.

<p align="center">
<img src="Pictures/wemos.jpg" width="250">
</p>

**Note:** Since we do not use a magnetometer, which in essence can tell the North orientation. We added some calibration step in the IMU start up code to calibrate the orientation with an external compass (on a mobile app for instance). The car needs to be placed on a flat surface, oriented with its front facing the North and the Reset button on the D1 mini should be pressed once. The calibration takes roughly 30 seconds.

## Serial communication difficulties
Another difficulty we encountered was related to the serial communication between the ESP8266 and the Python script. The MCU streams the angles continuously and we would struggle "syncing" the serial read from the Python script with the serial write from the board. Thankfully, Raspberry Pi's official website contains a guide on how to read serial through USB and gives some arguments to pass when declaring a ```serialPort``` object.

```python
serialPort = serial.Serial('/dev/ttyUSB0',
                            baudrate=115200,
                            parity=serial.PARITY_NONE,
                            stopbits=serial.STOPBITS_ONE)
```
The file ```readcompass.py``` reads from the USB serial and sends the orientation as MQTT messages on the *orientation* topic. With a MQTT subscriber it's very easy to retrieve the values and we actually used MOSQUITTO on Linux to monitor the values. MOSQUITTO is installed this way:

```
sudo apt-get update
sudo apt-get install mosquitto
sudo apt-get install mosquitto-clients
```

Then to read the orientation messages:

```
mosquitto_sub -h donkey-260 -t orientation
```

Once we could reliably read the orientation, the next step was to handle those angles and generate high level commands.

# High Level Command (HLC)

Our objective was to use the high-level command to gather information about whether the car should turn RIGHT, LEFT, or continue CENTER. For instance, when approaching an intersection, the car would rely on the HLC to determine whether to turn or continue straight while also avoiding obstacles.

To run the drive with the high level command we use :
```
python3 manage.py drive --js --type behavior
```
We added the ```--type behavior``` parameter to ensure that we wouldn't encounter any issues.


## Manual HLC
To set up the model that takes high-level commands as inputs, we typically just need to set **TRAIN_BEHAVIOR** to **True** in the Donkey Car configuration file ```myconfig.py```. However, this approach did not work for us since the high-level commands were not being included in the model inputs when we attempted to train it. In order to resolve this issue, we needed to modify the code. Specifically, we modified the controller code to include the high-level state when the drive was launched. This allowed us to add the high-level command as an input for the model during training. Additionally, while making these changes, we also modified the controller code to enable us to manually change the high-level command using the dpad on the controller.

Here is a demo video of manual HLC:

https://youtu.be/aO8aMNdHT0k


## IMU HLC
The aim was to create a high-level command based on IMU data and basic logic. For instance, we manually set a target angle of 90 degrees to the right, which generated an HLC of RIGHT until the IMU detected that we had completed the 90-degree turn, at which point it generated a CENTER HLC. To test this, we added a button to the controller that would prioritize either the IMU-generated HLC or the manual HLC from the dpad. Furthermore, we were able to replace the dpad's manual command sending function with a function that sends the target angle to the IMU code, allowing the IMU to generate the appropriate HLC. This simulation was necessary to prepare for the scenario where another code (like localization) would provide target angles to the IMU code.

## Localization and HLC
The goal is to generate High-Level Commands (HLC) based on the car's position, orientation, and a given goal point. To achieve this, we need to calculate the angle difference between the car and the goal point and use the same process as in the IMU HLC section. However, instead of setting the angle manually, we will derive it from the car's relative position to the goal point. Although this approach is theoretically feasible, the current lack of precision in localization makes it unreliable in practice.

# Training a model

To train the model we need to run the following command:
```
donkey train --tub ./data/<tub name> --model ./models/<model name>
```
But this is only if we want to train a model without the high level command. If we want to add the high level command, we need to specify again ```--type behavior:```
```
donkey train --tub ./data/<tub name> --model ./models/<model name> --type behavior
 ```

```<tub name>```  is not necessary is you don't have the data directly in the data folder.

Initially, we aimed to train a linear model to enable autonomous driving of the car along the circuit. We gathered data from multiple laps and attempted to train the model. However, the training process took much longer than anticipated due to several issues, which we will discuss below.

Here is a video demo of our Donkey Car doing laps autonomously:

https://youtu.be/tWAjIFH8FDw


To train our model, we attempted to use the computer's GPU by following the instructions in the Donkey Car documentation. After numerous hours of installation, deletion, and reinstallation of different versions, we finally succeeded in getting the GPU to work. However, the training process was still problematic - it took a long time to start and would stop or crash after a few epochs. At first, we thought that it might be due to corrupted data. So, we tried different configurations, such as changing the resolution of the images, but we still couldn't get it to work. We also tried reducing the number of parameters of the model to lessen the GPU's load. Finally, what worked for us was setting IMAGE_H which is in the ```myconfig.py``` file to 128 and training the model on the CPU instead of the GPU. Initially, we had kept the default IMAGE_H value of 120, but a warning message indicated that it would be automatically adjusted to 128 because 120 was not feasible. To ensure that we had control over this variable, we manually defined IMAGE_H as 128.


Once we were able to train a linear model, the next challenge was to train behavior model and to make it perform well. To achieve the desired results, we had to overfit the model to the specific use case. For example, if we wanted to train the model to drive the car straight until it reached an intersection, and then turn left or continue straight based on high-level commands, we had to use the exact same intersection for both training and testing. If we tested the same model on a different intersection, it would not work. This limitation was primarily due to the amount of training data we used. By increasing the quantity of data, we could potentially generalize the model to multiple intersections. However, our immediate goal was to test whether the high-level command would work with the model so that we could move forward and try the same approach with localization.



# Hardware issues

The Donkey Cars are cool little cars to learn Deep Learning... When they don't fail miserably.

During the project we unfortunately had issues with our car, the number 124. At first it had a really bad steering angle but we managed to tune it a little bit by setting up the steering mechanism: the axles can be extended or shortened by screwing them in or out. This pushes the wheel inward or outward and can correct a bad parallelism. Also, the software calibration presented at the beginning of the blog helped compensate some drift but the car was actually never able to make sharp turns.

At some point we tried to SSH into the car and it just stopped working overnight. We thought the OS was corrupted at first. If the battery is too low and the car shuts down while it's processing, it could ruin the file system and make the car unbootable. That's why the command below should always be used before unplugging the battery or PSU. 
```
sudo poweroff
``` 

Unfortunately, a fresh install on the SD card didn't solve the problem. We tried different SD cards, several Raspberry Pis, we also tried to plug the HDMI port to a monitor to see if we could play with the terminal but there was no signal. This was very problematic since we couldn't do anything with the car and make any progress. 

We simply got a brand new car, the number 260. This one had a really good steering angle and we could make much sharper and more usable turns. After wasting precious hours we could finally go back to work.

# Next steps

At the stage were we have to submit this blog (10/05/2023) the project is technically not complete. The car is not able to navigate autonomously yet, but we still managed to develop the building blocks of what could be the final solution:

- **Localization:** improving the work of Anton Slavin on WiFi router localization, we have a way to localize more or less precisely the car in a straight line (ideally Delta's 2 floor for now). The refresh rate is okay for a moving vehicle but the accuracy of localization could be improved further (better calculations, filtering routers...).

- **IMU:** we have our "compass" working. The MPU6050 works enough for now although we need to manually set the North everytime. We could improve this part using the MPU9250 that's on the car. The Donkey Car community could help, or the sensor would be rather easy to source and install the way we did with our own IMU.

- **High level commands:** the high level commands work with a very simple model trained on very few data. We could start training a more serious model driving around the entire building. That way we could hopefully generalize on more intersections that the one next to the IoT Lab. The commands are easy to generate both manually and with our compass.

# Conclusion

We showed with this project that it is feasible to localize the Donkey Car more or less precisely to allow free range driving in the entire Delta building without using traditional solutions like LiDars. We also managed to implement high level commands to dictate some action based on a compass, which the car obeys only if the environment allows it.

Unfortunately, we wasted quite a lot of precious time on hardware debugging. Thus, we do not have a working autonomous car at the end date of this project. It is quite frustrating but we still managed to come up with all the "parts" we would need to push this work forward and eventually complete it.

About the learning outcomes of this project, it was a great opportunity for us to dive deep into some complex code architecture and inject our bits here and there, with sometimes some very limited documentation from the Donkey Car community. We also learned that even if the scope of this project was mostly software based, when the hardware doesn't want to cooperate, the entire project is stalled.

# Acknowledgement

We would like to thank Anton Slavin for his work on WiFi localization and his tips on how to make his app work on our machine.

We would also like to thank Naveed Muhammad and Ardi Tampuu for their trust and guidance trhoughout this whole project.
