+++
date = '2023-10-15'
draft = false
title = "Line-Following Robot"
[cover]
    image = "project-cover.png"
    relative = true
+++

### High-Level Summary
* **Description:** This project involved building a simple line-following robot. However, the project's complexity came from having to design circuits for the control of the motors and the use of the Arduino's PWM functionalities was not allowed. Hence, we had to use a DAC to convert our digital signal into an analog signal. This was then converted to a PWM using a 555 Timer and comparator. The final signal was then power amplified with a class AB amplifier.
* **Key Takeaways:** There was a point in this project where our entire circuit just didn't work, and we had to look at the voltage and current at every node to troubleshoot. When it finally worked, it felt amazing. Being able to build the circuit and then program the controller to get the robot to follow the line by reading the sensors and sending the appropriate PWM signals to the motor - it felt awesome to see it finally all come together at the end.
