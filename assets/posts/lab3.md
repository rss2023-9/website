Lab 3
=====

<p align="center">
  <img src="assets/images/racecar.png" height="400"/>
</p>

You can find the PDF version of our lab report [here](https://drive.google.com/file/d/1FVIkmWdaSe2OyTu_l8AlUdAgZb04LqoW/view?usp=sharing).

## Introduction - [Abdurahman Sherif]

Throughout the past few labs, we have been laying the foundations for an autonomous race car that can effectively adapt and react to its environment. Through this lab, we tried to answer the questions: ”can the car stay on a desired path?” and ”can it stop before crashing into an object?” Both of these questions relate to our overall goal of translating math and simulations to real-world signals and controls. With these in mind, the wall follower car was designed into two separate components: wall following and safety control.

Our first goal was to test the wall follower we developed in simulation on a real-life race car. We achieved this using the Robot Operating System (ROS), the onboard 2D Hokuyo LiDAR, and feedback control, which allowed the car to detect obstacles through laser scans and react to its environment with the control parameters we provided. We used PD control to keep the car on course at a desired distance from the wall. Our second goal was to develop a safety controller to prevent the car from colliding with pedestrians, objects, or the wall, protecting its internal hardware in the process.

Both the wall follower and safety controller are crucial components of an autonomous system. Wall following is considered low-level control as it involves maintaining a specified distance from the wall. However, the wall follower can be utilized to accomplish more complex motion planning tasks such as reaching a destination from a starting point. The PD control we fine-tuned in this lab can help the car move smoothly in future autonomous navigation tasks. The safety controller acts as a backup in case the autonomous mode fails, particularly when the car is traveling at high speeds.

## Proposed Approach [Rumaisa Abdulhai, Brandon Lei]

This section will describe the initial setup, technical approach, and the ROS architecture we used to develop a robust and safe wall follower that can follow the desired side at various speeds and distances from the wall.

### Initial Setup - [Rumaisa Abdulhai]

To use the race car, our team had to set up the router, connect to the car via SSH, and charge the motor battery and car battery that powers the TX2. Once the batteries were charged, we mounted them on the car. We also made sure the joystick was connected to the car by attaching the receiver and running the teleop command in the terminal. This setup allowed us to teleoperate the race car with the joystick. Afterward, we copied our existing wall follower code to the car and began to visualize the laser scans from the LiDAR in RViz.

### Wall Follower Technical Approach - [Rumaisa Abdulhai]

Our wall follower implementation involves three components: filtering relevant laser scan data, using linear regression to fit a line to the filtered data, and implementing Proportional-Derivative (PD) control to steer the car and maintain a desired distance from the wall.

To filter the laser data, we use two criteria. Firstly, we only consider data on the side of the car that is relevant to the wall being followed. For example, if the car is following the right wall, we only pay attention to points on the right side of the car. Secondly, we discard laser scans that are far away from the car, as they are unlikely to be relevant. Filtering out irrelevant data enables us to obtain a more accurate estimate of the wall we want to follow. Specifically, we utilize points that are within 3 times the desired distance. To address the issue of the car turning around corners, we utilize the scans in front of the car when the distance between the car and the front wall is less than three times the desired distance. When the car is not turning a corner, we utilize the laser scans of the wall we are following. In our wall follower implementation in the simulator, we initially divided the laser scan into multiple parts using slicing. However, when we tested it on the car, we found that using angle ranges to extract the relevant side of the wall was more intuitive, so we switched from slicing to using angle ranges.

The second component involves using linear regression to obtain an equation for the line that estimates the wall using the filtered laser scan data. By fitting a line to the laser scan data, we can generalize the wall close to the car, which can be useful if the wall surface is rough or slanted. Using the $\texttt{np.polyfit}$ method, we estimate a line of best fit by performing a least-squares fit of a one-degree polynomial to x and y coordinates that correspond to the relative positions of the laser scans. From this, we obtain the slope and y-intercept (m and b), which we convert to the form $Ax + By + C = 0$. We then use these constants to determine the shortest distance between the car and the wall. The actual distance to the wall is determined by:
$$\frac{\left|C\right|}{\sqrt{A^2 + B^2}}$$
The third component involves using PD control to steer the car and maintain a desired distance from the wall. If the car is closer to the wall than the desired distance, it should turn away from the wall, while if it is farther from the wall, it should turn towards the wall. Feedback control is used because the car's hardware components affect the turns and the distance it travels. Therefore, PD control is employed to correct any bias and keep the car on the right path. By using the actual distance to the wall and the desired distance, the desired steering angle of the race car can be calculated:
$$\textit{steering angle} = -side \cdot (K_p \cdot \textit{current error} + K_d \cdot (error - \textit{previous error}))$$
where the current error is $(\textit{desired distance} - \textit{front distance})$ and the previous error is the error at the last time step. The side is -1 if the car is following the right side of the wall and +1 if the car is following the left side of the wall. Through PD tuning, we were able to determine the values of $K_p$ and $K_d$. Our goal was to minimize oscillations and enable the car to make sharp turns around corners. When turning inside corners, a larger $K_p$ value was used to allow the car to turn faster, while a smaller $K_p$ value was used when simply following the left or right side of the wall.

However, if a pedestrian or object suddenly comes into view of the car, or if the car is moving too fast and approaching a wall, the above approach may not suffice. To address this issue, a safety controller is needed to override our wall follower steering commands and stop the car in time to prevent any harm to the car. The following section will discuss how this problem is resolved.

### Safety Controller - [Brandon Lei]

Duis vel nunc sit amet risus consectetur dictum. Nulla mollis varius erat, vitae gravida est elementum a. Curabitur velit sapien, placerat ac scelerisque quis, ultricies at sem. Maecenas ut elit congue, condimentum lacus eu, scelerisque nunc. Curabitur mattis velit vitae sem placerat varius vel euismod leo. Cras quis elit quam. Proin scelerisque lobortis erat, eu euismod ex mattis ac. Curabitur non felis mauris. Integer mauris nisi, rutrum id finibus vel, ornare quis diam. Cras lectus nisi, pharetra ut elit at, porta auctor ex. Cras lobortis nisl leo, varius aliquet arcu sollicitudin vel. Aliquam quis nulla sapien. Donec porttitor, tortor vel iaculis vehicula, ipsum eros dictum eros, sit amet tempor orci felis vitae magna. Sed velit lacus, tincidunt sit amet quam sed, aliquam porttitor ex.

	Duis vel nunc sit amet risus consectetur dictum. Nulla mollis varius erat.

## Experimental Evaluation - [Erica Radler]

Nulla tempus tempor sollicitudin. Sed id tortor vestibulum, tincidunt lorem a, suscipit lacus. Mauris vitae pretium libero, at dapibus massa. Curabitur eleifend bibendum pharetra. Nullam gravida viverra lacus eu blandit. Praesent nec odio ut magna scelerisque vulputate. Sed in libero porta, imperdiet magna maximus, efficitur urna.

### Testing Procedure

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam sed consequat ligula. Aliquam erat volutpat. Cras iaculis diam vitae nunc ultricies, et egestas lorem eleifend. Ut sit amet leo vitae libero maximus molestie non ac nunc. Ut ac mi ante. Vivamus convallis convallis neque, sit amet sollicitudin arcu bibendum sit amet. Phasellus finibus dolor vitae leo cursus, eu lobortis nisl blandit. Quisque tincidunt et nisi a hendrerit. Sed et nunc quis neque egestas sollicitudin. Curabitur auctor bibendum odio. Proin aliquam cursus metus, at fermentum tellus luctus vel. Morbi ut mi id augue lacinia faucibus.

Duis vel nunc sit amet risus consectetur dictum. Nulla mollis varius erat, vitae gravida est elementum a. Curabitur velit sapien, placerat ac scelerisque quis, ultricies at sem. Maecenas ut elit congue, condimentum lacus eu, scelerisque nunc. Curabitur mattis velit vitae sem placerat varius vel euismod leo. Cras quis elit quam. Proin scelerisque lobortis erat, eu euismod ex mattis ac. Curabitur non felis mauris. Integer mauris nisi, rutrum id finibus vel, ornare quis diam. Cras lectus nisi, pharetra ut elit at, porta auctor ex. Cras lobortis nisl leo, varius aliquet arcu sollicitudin vel. Aliquam quis nulla sapien. Donec porttitor, tortor vel iaculis vehicula, ipsum eros dictum eros, sit amet tempor orci felis vitae magna. Sed velit lacus, tincidunt sit amet quam sed, aliquam porttitor ex.

### Results

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam sed consequat ligula. Aliquam erat volutpat. Cras iaculis diam vitae nunc ultricies, et egestas lorem eleifend. Ut sit amet leo vitae libero maximus molestie non ac nunc. Ut ac mi ante. Vivamus convallis convallis neque, sit amet sollicitudin arcu bibendum sit amet. Phasellus finibus dolor vitae leo cursus, eu lobortis nisl blandit. Quisque tincidunt et nisi a hendrerit. Sed et nunc quis neque egestas sollicitudin. Curabitur auctor bibendum odio. Proin aliquam cursus metus, at fermentum tellus luctus vel. Morbi ut mi id augue lacinia faucibus.

Duis vel nunc sit amet risus consectetur dictum. Nulla mollis varius erat, vitae gravida est elementum a. Curabitur velit sapien, placerat ac scelerisque quis, ultricies at sem. Maecenas ut elit congue, condimentum lacus eu, scelerisque nunc. Curabitur mattis velit vitae sem placerat varius vel euismod leo. Cras quis elit quam. Proin scelerisque lobortis erat, eu euismod ex mattis ac. Curabitur non felis mauris. Integer mauris nisi, rutrum id finibus vel, ornare quis diam. Cras lectus nisi, pharetra ut elit at, porta auctor ex. Cras lobortis nisl leo, varius aliquet arcu sollicitudin vel. Aliquam quis nulla sapien. Donec porttitor, tortor vel iaculis vehicula, ipsum eros dictum eros, sit amet tempor orci felis vitae magna. Sed velit lacus, tincidunt sit amet quam sed, aliquam porttitor ex.

## Conclusion - [Abdurahman Sherif]

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam sed consequat ligula. Aliquam erat volutpat. Cras iaculis diam vitae nunc ultricies, et egestas lorem eleifend. Ut sit amet leo vitae libero maximus molestie non ac nunc. Ut ac mi ante. Vivamus convallis convallis neque, sit amet sollicitudin arcu bibendum sit amet. Phasellus finibus dolor vitae leo cursus, eu lobortis nisl blandit. Quisque tincidunt et nisi a hendrerit. Sed et nunc quis neque egestas sollicitudin. Curabitur auctor bibendum odio. Proin aliquam cursus metus, at fermentum tellus luctus vel. Morbi ut mi id augue lacinia faucibus.

Duis vel nunc sit amet risus consectetur dictum. Nulla mollis varius erat, vitae gravida est elementum a. Curabitur velit sapien, placerat ac scelerisque quis, ultricies at sem. Maecenas ut elit congue, condimentum lacus eu, scelerisque nunc. Curabitur mattis velit vitae sem placerat varius vel euismod leo. Cras quis elit quam. Proin scelerisque lobortis erat, eu euismod ex mattis ac. Curabitur non felis mauris. Integer mauris nisi, rutrum id finibus vel, ornare quis diam. Cras lectus nisi, pharetra ut elit at, porta auctor ex. Cras lobortis nisl leo, varius aliquet arcu sollicitudin vel. Aliquam quis nulla sapien. Donec porttitor, tortor vel iaculis vehicula, ipsum eros dictum eros, sit amet tempor orci felis vitae magna. Sed velit lacus, tincidunt sit amet quam sed, aliquam porttitor ex.

## Lessons Learned - [Whole team]

1. Brandon Lei
2. Erica Radler
3. Abdurahman Sherif
4. Rumaisa Abdulhai
