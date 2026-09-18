<!-- ## 2. Motor Control Methods

| Motor Type | Microcontroller Signal | Unit / Parameter | Interface | Resulting Motion |
|---|---|---|---|---|
| **Brushed DC Motor** | PWM Signal | Duty Cycle | Motor Driver | Continuous rotation; RPM generally depends on duty cycle |
| **BLDC Motor** | PWM Signal | Throttle / Speed Command | Electronic Speed Controller (ESC) | Continuous rotation; RPM depends on the commanded speed and motor/ESC characteristics |
| **Stepper Motor** | Step Pulses | Steps / Microsteps | Stepper Motor Driver | Discrete angular movement with precise position and speed control |
| **Servo Motor** | PWM-style Position Signal | Pulse Width / Angular Position | Servo Control Pin | Controlled and accurate angular positioning | -->
# Problem Statement 1 — Motors, Drivers and Electrical Noise

## 1. Motor Comparison

| Parameter | Brushed DC Motor | BLDC Motor | Stepper Motor | Servo Motor |
|---|---|---|---|---|
| Basic construction | Permanent-magnet rotor with brushes and commutator | Permanent-magnet rotor with electronically commutated stator | Multi-phase stator with toothed/magnetized rotor | Motor combined with feedback sensor and controller |
| Typical controller input | PWM + direction | PWM, speed command, analog, CAN/UART depending on controller | STEP + DIR pulses | Pulse-width angle command, CAN/UART/RS485 depending on servo |
| Command unit | Duty cycle (%) | Speed command / duty cycle / voltage | Pulses or steps | Angle / position |
| Driver required | H-bridge motor driver | 3-phase BLDC driver / ESC | Stepper driver | Servo drive/controller |
| Feedback | Optional encoder/Hall sensor | Hall sensor / encoder commonly available | Usually none in open-loop operation | Required for closed-loop operation |
| Motion | Continuous rotation | Continuous rotation | Discrete angular increments | Closed-loop position/velocity/torque control |
| Holding behaviour | Poor without active control | Poor without active control | Strong static holding torque when energized | Actively holds commanded position |
| Main advantage | Simple, inexpensive, high starting torque | Efficient, low maintenance and good speed control | High low-speed torque and simple position control | High accuracy and closed-loop positioning |
| Main disadvantage | Brush wear and electrical noise | More complex electronic commutation | Can lose steps in open-loop operation | Higher cost and greater system complexity |
| Typical failure mode | Brush/commutator wear, driver failure, overheating during stall | Driver/phase/Hall failure, thermal overload | Missed steps, resonance, overheating | Encoder/driver failure, overheating, gearbox backlash |
| Typical robotics application | Simple geared drive systems | Mobile robot drivetrain | Lead screws, indexing and Z axes | Robot joints and precision actuators |

The problem statement specifically requires comparison of DC, BLDC, stepper and servo motors, including their input signals, units, required drivers, feedback, holding behaviour and failure modes.  
**Source:** Electrical Problem Statements, Problem Statement 01, pages 4–6.

---

## 2. The Unit Question

### Brushed DC Motor

> To make this move, the microcontroller sends **PWM signals**, in units of **duty cycle (%)**, through an **H-bridge motor driver**, and the resulting motion is **continuous rotation whose speed depends on applied average voltage and mechanical load**.

For a 24 V supply and 60% duty cycle:

\[
V_{avg}=D V_{supply}
\]

\[
V_{avg}=0.60\times24
\]

\[
\boxed{V_{avg}=14.4V}
\]

The motor speed is not necessarily 60% of its no-load speed because the actual operating point depends on load, back-EMF, winding resistance and losses.

---

### BLDC Motor

> To make this move, the microcontroller sends a **speed/drive command**, in units of **speed command, PWM duty or control voltage**, through an **electronic commutation controller/ESC**, and the resulting motion is **continuous rotation**.

A Hall-sensored BLDC controller uses rotor-position information to determine the appropriate stator phase excitation.

---

### Stepper Motor

> To make this move, the microcontroller sends **STEP and DIR pulses**, in units of **steps/pulses**, through a **stepper driver**, and the resulting motion is **discrete angular movement determined by the commanded step rate and step size**.

For a 200-step/revolution motor:

\[
N_{rev}=\frac{N_{steps}}{200}
\]

For 1000 step pulses:

\[
N_{rev}=\frac{1000}{200}
\]

\[
\boxed{N_{rev}=5\text{ rev}}
\]

In open-loop operation, the controller assumes the commanded steps were successfully executed.

---

### Servo Motor

> To make this move, the microcontroller sends a **position command**, in units of **angle/position**, through a **servo controller**, and the resulting motion is **closed-loop movement toward and then holding the commanded position**.

For a conventional hobby servo, the pulse width represents position rather than motor-power duty cycle.

---

# 3. Microstepping

For a two-phase stepper, the phase currents can be varied approximately according to:

\[
I_A=I_{max}\sin\theta
\]

\[
I_B=I_{max}\cos\theta
\]

This allows the rotor equilibrium position to be placed between full-step positions.

### Advantages

- Smoother motion
- Reduced vibration
- Lower audible noise
- Finer command resolution

### Limitations

Microstepping does not imply a proportional increase in practical positional accuracy. Mechanical backlash, friction, motor nonlinearity, load disturbance and torque per microstep limit the usable accuracy.

The problem statement specifically notes that 1/256 microstepping does not provide 256 times the useful positional accuracy.

---

# 5. Drivetrain Sizing Calculation

## 5.1 Given Values

From the problem statement:

| Parameter | Value |
|---|---:|
| Robot mass | 20 kg |
| Number of driven wheels | 4 |
| Wheel radius | 60 mm = 0.06 m |
| Ramp angle | 15° |
| Required flat-ground speed | 1.2 m/s |

The problem statement asks for:

- Required torque per wheel
- Required RPM
- Current at the operating point
- Whether the selected motor and gearbox can survive continuously

Because rolling resistance and acceleration are not provided, assumptions must be stated explicitly.

---

## 5.2 Assumptions

| Parameter | Assumed value |
|---|---:|
| Rolling-resistance coefficient, \(C_{rr}\) | 0.02 |
| Design acceleration | 0.5 m/s² |
| Gravity, \(g\) | 9.81 m/s² |
| Load distribution | Equal across four wheels |

The value of \(C_{rr}\) should ideally be measured for the actual wheel and floor combination.

---

## 5.3 Force Due to Gravity

The component of gravitational force acting down the 15° slope is:

\[
F_g=mg\sin\theta
\]

Substituting:

\[
F_g=(20)(9.81)\sin15^\circ
\]

\[
\boxed{F_g=50.78N}
\]

---

## 5.4 Rolling Resistance

Rolling resistance is approximated as:

\[
F_{rr}=C_{rr}mg\cos\theta
\]

\[
F_{rr}=0.02(20)(9.81)\cos15^\circ
\]

\[
\boxed{F_{rr}=3.79N}
\]

---

## 5.5 Acceleration Force

For the assumed acceleration:

\[
F_a=ma
\]

\[
F_a=(20)(0.5)
\]

\[
\boxed{F_a=10N}
\]

---

## 5.6 Total Required Traction Force

\[
F_{total}=F_g+F_{rr}+F_a
\]

\[
F_{total}=50.78+3.79+10
\]

\[
\boxed{F_{total}=64.57N}
\]

---

## 5.7 Force Per Wheel

Assuming equal load sharing between all four driven wheels:

\[
F_{wheel}=\frac{F_{total}}{4}
\]

\[
F_{wheel}=\frac{64.57}{4}
\]

\[
\boxed{F_{wheel}=16.14N}
\]

---

## 5.8 Required Torque Per Wheel

Wheel torque is:

\[
T=Fr
\]

Therefore:

\[
T_{wheel}=16.14(0.06)
\]

\[
\boxed{T_{wheel}=0.969Nm}
\]

Therefore, under the stated assumptions:

\[
\boxed{T_{wheel}\approx0.97Nm}
\]

---

## 5.9 Steady-State Torque Without Acceleration

If only climbing force and rolling resistance are considered:

\[
F_{steady}=F_g+F_{rr}
\]

\[
F_{steady}=50.78+3.79
\]

\[
F_{steady}=54.57N
\]

Per wheel:

\[
F_{steady,wheel}=\frac{54.57}{4}
\]

\[
F_{steady,wheel}=13.64N
\]

Required steady-state wheel torque:

\[
T_{steady}=13.64(0.06)
\]

\[
\boxed{T_{steady}\approx0.82Nm}
\]

---

# 6. Required Wheel RPM

The required wheel angular velocity is:

\[
\omega=\frac{v}{r}
\]

\[
\omega=\frac{1.2}{0.06}
\]

\[
\boxed{\omega=20rad/s}
\]

Convert rad/s to RPM:

\[
RPM=\omega\frac{60}{2\pi}
\]

\[
RPM=20\frac{60}{2\pi}
\]

\[
\boxed{RPM\approx191RPM}
\]

Therefore:

\[
\boxed{RPM_{wheel}\approx191}
\]

---

# 7. Mechanical Power Requirement

Total mechanical power at the assumed ramp operating point is:

\[
P=Fv
\]

\[
P=64.57(1.2)
\]

\[
\boxed{P\approx77.5W}
\]

Per wheel:

\[
P_{wheel}=\frac{77.5}{4}
\]

\[
\boxed{P_{wheel}\approx19.4W}
\]

Therefore each wheel needs approximately:

\[
\boxed{0.97Nm\text{ at }191RPM}
\]

under the stated assumptions.

---

# 8. Example Motor Check

A suitable example for checking the calculated requirement is a:

**42BLS61, 24 V, 50 W BLDC geared motor with 20:1 gearbox**

Published specifications include approximately:

- 24 V supply
- 50 W rated power
- 200 RPM rated speed
- 2.5 Nm rated torque
- 20:1 gearbox
- Hall sensors
- 4 A rated current

### Torque margin

Required:

\[
T_{required}=0.969Nm
\]

Published rated torque:

\[
T_{rated}=2.5Nm
\]

Torque margin ratio:

\[
M_T=\frac{2.5}{0.969}
\]

\[
\boxed{M_T\approx2.58}
\]

Thus the published rated torque is approximately 2.58 times the calculated requirement.

### Speed comparison

Required:

\[
RPM_{required}=191RPM
\]

Rated speed:

\[
RPM_{rated}=200RPM
\]

Therefore:

\[
\frac{191}{200}=0.955
\]

The required operating speed is approximately:

\[
\boxed{95.5\%}
\]

of the stated rated speed.

The manufacturer's speed-torque curve should still be checked before a real hardware purchase because the rated torque and rated speed should not automatically be treated as an arbitrary simultaneous operating point.

---

# 9. Current Estimate

Using the example motor's published rated torque and current:

\[
T_{rated}=2.5Nm
\]

\[
I_{rated}=4A
\]

A first-order estimate of the effective torque constant is:

\[
k_T\approx\frac{T}{I}
\]

\[
k_T\approx\frac{2.5}{4}
\]

\[
\boxed{k_T\approx0.625Nm/A}
\]

Estimated current at the calculated wheel torque:

\[
I\approx\frac{T_{required}}{k_T}
\]

\[
I\approx\frac{0.969}{0.625}
\]

\[
\boxed{I\approx1.55A}
\]

This is an engineering estimate, not a datasheet measurement at exactly 191 RPM and 0.969 Nm.

For conservative electrical design, the motor's published 4 A rated current should be used:

\[
I_{4motors}=4(4)
\]

\[
\boxed{I_{4motors}=16A}
\]

---

# 10. Motor Driver Considerations

For a BLDC motor, the driver must provide:

- Correct motor voltage range
- Sufficient continuous current
- Sufficient peak current
- Hall/encoder compatibility
- Thermal protection
- Over-current protection
- Appropriate control interface

A BLD-305S-class BLDC controller provides Hall-sensor support, closed-loop control and protection functions such as over-current, over-voltage, under-voltage and over-temperature protection.

The driver should not be selected only from the motor's average operating current. Stall/peak current and thermal limits must also be considered.

---

# 11. Why a Stepper Can Lose Steps

In an open-loop stepper system:

```text
MCU
 |
 | STEP / DIR
 v
Stepper Driver
 |
 v
Stepper Motor