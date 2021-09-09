# Social Pepper Detection

A Social integration for Pepper that makes it look in three directions (left, center, right) and say what it saw.  
You can see a demonstration on [youtube](https://www.youtube.com/watch?v=GA1LUYwDQnA).

### Authors: Group 14
| Name | Student ID |
|--------------|--------|
|Davide Della Monica | 0622701345|
|Vincenzo di Somma | 0622701283|
|Salvatore Gravina | 0622701063|
|Ferdinando Guarino | 0622701321|

# Running

### Requirements
- Ros melodic
- NaoQI SDK
- To run speech_node you need to install the [inflect package](https://pypi.org/project/inflect/)
~~~ sh
pip3 install inflect
~~~

git clone for repository and submodules
~~~ sh
git clone --recurse-submodules -j8 https://github.com/FerdyGuar97/SocialPepperDetectionG14.git
~~~
build the workspace
~~~ sh
catkin build
~~~

add NaoQI SDK to path

1. Copy the realpath of the package
~~~ sh
D=$(realpath pynaoqi-python2.7-2.5.7.1-linux64)
~~~

2. Add the package to the "setup.bash" file
~~~ sh
echo "export PYTHONPATH=\${PYTHONPATH}:$D/lib/python2.7/site-packages" >> devel/setup.bash
~~~

~~~ sh
echo "export DYLD_LIBRARY_PATH=\${DYLD_LIBRARY_PATH}:$D/lib" >> devel/setup.bash
~~~


## Execution

For a successful execution follows these steps:

1. connect to pepper
~~~ sh
roslaunch pepper_bringup pepper_full_py.launch nao_ip:=PEPPER_IP
~~~

2. run the detector, wait for the "detector ready" feedback to launch the number 6
~~~ sh
rosrun detector detector_node
~~~

3. run the image publisher
~~~ sh
rosrun detector imagepublisher_node
~~~

4. run the service AnimatedSay
~~~ sh
rosrun pepper_talk pepper_talk_node --pip PEPPER_IP
~~~

5. run the speech node
~~~ sh
rosrun pepper_talk speech_node
~~~

6. run the head movement node
~~~ sh
rosrun detector headmovement_node
~~~

In order to re-execute the demostration you just need to run the headmovement_node again. 

# Architecture
![SocialPepperDetector](https://user-images.githubusercontent.com/52747015/100008851-43dcf880-2dce-11eb-88bd-319c04db0e40.jpg)

### imagepublisher_node
This node acquires the image stream from Pepper's front camera by subscribing to the */pepper_robot/camera/front/camera/image_raw* topic. It also subscribes 
to the *positionACK* topic so that it can publish one image from the stream on the *frame* topic only when Pepper's head is properly oriented.

### headmovement_node
This node uses NaoQi to move Pepper's head. It publishes the positions on the */pepper_robot/pose/joint_angles* topic, it notifies when the head reaches the designated position by publishing an ACK on the *positionACK* topic and publishes the orientation of the head (left, center, right) on the *pov* topic.
At the end of the movement routine publishes an ACK on the *end* topic.

### detector_node
The task of this node is to load the detector and to perform a detection on the image received by subscribing to the *frame* topic. It also subscribes to the *pov* topic to add information about the point of view of the scene to the detection results, before publishing everything on the *detection* topic.

### peppertalk_node
This node is tasked with the initialization of the AnimatedSay service, which allows us to make Pepper talk while executing some animations.

### speech_node
This node receives information about objects detected using the three predefined point of views by subscribing to the *detection* topic. It uses this information to build a sentence to be spoken by Pepper. 
The speech is activated by calling the AnimatedSay service after having received an ACK from the *end* topic.

# Design choices

### Detector
We chose EfficientDet D1 640x640 for its trade-off between performance and speed. We chose a threshold of 50% for confidence value to filter valid detections.

### Rotation angle
We chose a rotation angle of 57.2° (1 rad) in order to move the camera by an entire field of view of the Pepper front camera.

### Robustness
We chose to implement a robust architecture in order to grant independence from the hardware running the system. 
In particular:
- imagepublisher_node publishes an image only when the head arrives at the designated position
- detector_node uses a buffer to store positions related to the images to be detected in the correct order

### Sociality
To achieve a better flow of speech we designed the system so that Pepper only talks after the head movement routine ends and the results of all detections are available.
