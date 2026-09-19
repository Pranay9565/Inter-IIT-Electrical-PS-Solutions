## 2. Motor Control Methods

| Motor Type | Microcontroller Signal | Unit / Parameter | Interface | Resulting Motion |
|---|---|---|---|---|
| **Brushed DC Motor** | PWM Signal | Duty Cycle | Motor Driver | Continuous rotation; RPM generally depends on duty cycle |
| **BLDC Motor** | PWM Signal | Throttle / Speed Command | Electronic Speed Controller (ESC) | Continuous rotation; RPM depends on the commanded speed and motor/ESC characteristics |
| **Stepper Motor** | Step Pulses | Steps / Microsteps | Stepper Motor Driver | Discrete angular movement with precise position and speed control |
| **Servo Motor** | PWM-style Position Signal | Pulse Width / Angular Position | Servo Control Pin | Controlled and accurate angular positioning |


## Part 2

# Brushed DC Motor
To make this move, the microcontroller sends pwm signals, in units of duty cycle, through a 
Motor driver, and the resulting motion is continuous rotation whose rpm depends on duty cycle.

# BLDC Motor
To make this move, the microcontroller sends pwm signals, in units of electrical cycles, through electronic speed controller, and the resulting motion is continuous rotation at constant rpm which is dictated by the pwm signal.

# Stepper Motor
To make this move, the microcontroller sends step pulses, in unit of steps, through a stepper motor driver, and the resulting motion is discrete angular movement.

# Servo Motor
To make this move, the microcontroller sends pwm signals, in units of angular position, through its pins, and the resulting motion is accurate angular positioning.



## Section 5
# Drive wheels for rover
For rover wheels it is better to use dc motors as they offer high energy efficiency, simplicity to use, lightweight design, no need of very complex controller board the way needed for bldc motors.
# An arm joint
For arm joints, bldc motors with high gear reductions are used so as to maximize load bearing capacity and precision is offered by various sensors in the arm.
# Vertical lead screw
For vertical lead screw to return the same height repeatably, it is better to use stepper motors as they move in precise and fixed increments.
