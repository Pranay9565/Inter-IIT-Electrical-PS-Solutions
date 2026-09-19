# Problem Statement 02 — Encoders, Odometry and Sensor Characterization

> **Theme:** Sensing & Odometry  
> **Type:** Study, derivation and calibration  
> **Core skill:** Convert encoder counts into linear/angular velocity and characterize the robot's sensor intrinsics.

---

## 1. Objective

An encoder directly gives us **counts**. A differential-drive robot needs:

- wheel displacement
- wheel velocity
- robot linear velocity \(v\)
- robot angular velocity \(\omega\)
- calibrated odometry
- an error budget

The key idea is:

\[
\boxed{
\text{Encoder counts}
\rightarrow
\text{wheel revolutions}
\rightarrow
\text{wheel velocity}
\rightarrow
v,\omega
}
\]

The problem requires the encoder and robot intrinsics to be **measured rather than simply assumed**.

---

# 2. Encoder Types

## 2.1 Incremental Encoder

An incremental encoder generates pulses as the shaft rotates.

A quadrature encoder normally provides two signals:

- Channel A
- Channel B

The phase difference between A and B provides direction information.

### Advantages

- Simple interface
- High resolution possible
- Good for velocity measurement
- Relatively inexpensive

### Limitation

An incremental encoder does not normally know its absolute shaft position immediately after power-on. A startup reference such as homing or an index pulse may therefore be required.

---

## 2.2 Absolute Encoder

An absolute encoder directly provides the shaft position.

At power-on:

\[
\theta = \theta_{\text{absolute}}
\]

so the controller can know the shaft position without first rotating the mechanism to find a reference.

### Typical use

Absolute encoders are particularly useful for robot joints where the actual angular position is important immediately after startup.

---

## 2.3 Optical vs Magnetic Encoders

| Feature | Optical | Magnetic |
|---|---|---|
| Sensing principle | Light/interruption or optical code | Magnetic field |
| Resolution | Can be very high | Moderate to high |
| Environmental robustness | Can be affected by contamination | Generally tolerant of dirt |
| Alignment | Mechanical/optical alignment required | Magnet-to-sensor alignment required |
| Typical use | Precision motion | Robotics and harsh environments |

---

## 2.4 Hall Sensors

Hall sensors detect magnetic-field transitions.

They are commonly used for:

- BLDC commutation
- speed sensing
- simple position feedback

They generally provide much lower position resolution than a dedicated high-resolution encoder.

---

# 3. PPR, CPR and Decoding

Encoder terminology must be checked carefully because manufacturers do not always use PPR and CPR in exactly the same way.

For a quadrature encoder:

| Decoding | Counts per encoder revolution |
|---|---:|
| X1 | \(PPR\) |
| X2 | \(2PPR\) |
| X4 | \(4PPR\) |

For X4 decoding, every rising and falling edge of both channels can be counted.

Therefore:

\[
N_{\text{counts/motor}}
=
4PPR
\]

---

# 4. Gearbox Effect

Let:

- \(PPR\) = encoder pulses per motor revolution
- \(D\) = decoding factor
- \(G\) = motor-to-wheel reduction ratio

Then:

\[
\boxed{
N_C=PPR\times D\times G
}
\]

where \(N_C\) is the number of counts per wheel revolution.

## Example

Given:

\[
PPR=500
\]

\[
D=4
\]

\[
G=30:1
\]

Then:

\[
N_C=500\times4\times30
\]

\[
\boxed{
N_C=60\,000\ \text{counts/wheel revolution}
}
\]

### Important

This calculation is valid only if the encoder's **500 PPR** specification means 500 cycles per channel per motor revolution. Always verify the manufacturer's definition.

---

# 5. From Counts to Wheel Velocity

Let:

- \(C[k]\) = encoder count at sample \(k\)
- \(C[k-1]\) = previous encoder count
- \(\Delta C=C[k]-C[k-1]\)
- \(N_C\) = counts per wheel revolution
- \(r\) = effective wheel radius
- \(\Delta t\) = sampling interval

---

## 5.1 Counts → Wheel Revolutions

\[
N_{\text{wheel rev}}
=
\frac{\Delta C}{N_C}
\]

---

## 5.2 Wheel Revolutions → Angular Displacement

One revolution equals:

\[
2\pi\ \text{rad}
\]

Therefore:

\[
\Delta\theta_w
=
\frac{2\pi\Delta C}{N_C}
\]

---

## 5.3 Angular Displacement → Angular Velocity

\[
\omega_w
=
\frac{\Delta\theta_w}{\Delta t}
\]

Therefore:

\[
\boxed{
\omega_w
=
\frac{2\pi\Delta C}
{N_C\Delta t}
}
\]

---

## 5.4 Wheel Angular Velocity → Linear Velocity

For a wheel:

\[
v_w=r\omega_w
\]

Therefore:

\[
\boxed{
v_w
=
\frac{2\pi r\Delta C}
{N_C\Delta t}
}
\]

---

# 6. Differential-Drive Robot

For a differential-drive robot:

- \(v_L\) = left wheel velocity
- \(v_R\) = right wheel velocity
- \(b\) = effective track width

The robot linear velocity is:

\[
\boxed{
v=\frac{v_R+v_L}{2}
}
\]

The robot angular velocity is:

\[
\boxed{
\omega=\frac{v_R-v_L}{b}
}
\]

Substituting the encoder equations:

\[
\boxed{
v=
\frac{\pi r}
{N_C\Delta t}
(\Delta C_R+\Delta C_L)
}
\]

and:

\[
\boxed{
\omega=
\frac{2\pi r}
{bN_C\Delta t}
(\Delta C_R-\Delta C_L)
}
\]

These equations form the core encoder-to-odometry conversion.

---

# 7. Odometry Position Update

Wheel displacement during a sampling interval is:

\[
\Delta s_L=v_L\Delta t
\]

\[
\Delta s_R=v_R\Delta t
\]

Robot displacement:

\[
\Delta s=
\frac{\Delta s_R+\Delta s_L}{2}
\]

Heading change:

\[
\Delta\theta=
\frac{\Delta s_R-\Delta s_L}{b}
\]

A midpoint update can be written as:

\[
x_{k+1}
=
x_k+
\Delta s
\cos\left(
\theta_k+\frac{\Delta\theta}{2}
\right)
\]

\[
y_{k+1}
=
y_k+
\Delta s
\sin\left(
\theta_k+\frac{\Delta\theta}{2}
\right)
\]

\[
\theta_{k+1}
=
\theta_k+\Delta\theta
\]

---

# 8. Distance Represented by One Count

One wheel revolution travels:

\[
L=2\pi r
\]

Therefore:

\[
\boxed{
d_{\text{count}}
=
\frac{2\pi r}{N_C}
}
\]

where \(d_{\text{count}}\) is the distance represented by one encoder count.

## Example

For:

\[
r=60\text{ mm}
\]

and:

\[
N_C=60\,000
\]

\[
d_{\text{count}}
=
\frac{2\pi(60)}
{60000}
\]

\[
\boxed{
d_{\text{count}}
=
0.006283\text{ mm/count}
}
\]

or:

\[
\boxed{
d_{\text{count}}
=
6.283\ \mu\text{m/count}
}
\]

> This is **resolution**, not actual positioning accuracy. Wheel slip, deformation, backlash, mounting error and encoder errors can dominate the real-world error.

---

# 9. Velocity Resolution

For a sampling frequency:

\[
f_s=100\text{ Hz}
\]

the sample period is:

\[
\Delta t=\frac{1}{100}=0.01\text{ s}
\]

Using the previous example:

\[
\Delta v
=
\frac{0.006283}
{0.01}
\]

\[
\boxed{
\Delta v
=
0.6283\text{ mm/s per count}
}
\]

Thus, with a 10 ms fixed measurement window, the smallest non-zero measured velocity increment is approximately:

\[
\boxed{
0.6283\text{ mm/s}
}
\]

per wheel.

---

# 10. Worked Differential-Drive Example

Assume:

\[
PPR=500
\]

\[
D=4
\]

\[
G=30:1
\]

\[
r=60\text{ mm}=0.06\text{ m}
\]

\[
b=300\text{ mm}=0.30\text{ m}
\]

\[
f_s=100\text{ Hz}
\]

Therefore:

\[
N_C=60\,000
\]

and:

\[
\Delta t=0.01\text{ s}
\]

Suppose:

\[
\Delta C_L=100
\]

\[
\Delta C_R=110
\]

### Left wheel

\[
v_L=
\frac{2\pi(0.06)(100)}
{60000(0.01)}
\]

\[
\boxed{
v_L=0.06283\text{ m/s}
}
\]

### Right wheel

\[
v_R=
\frac{2\pi(0.06)(110)}
{60000(0.01)}
\]

\[
\boxed{
v_R=0.06912\text{ m/s}
}
\]

### Robot linear velocity

\[
v=\frac{0.06283+0.06912}{2}
\]

\[
\boxed{
v=0.06597\text{ m/s}
}
\]

### Robot angular velocity

\[
\omega=
\frac{0.06912-0.06283}
{0.30}
\]

\[
\boxed{
\omega=0.02096\text{ rad/s}
}
\]

The robot is therefore moving forward while turning slightly.

---

# 11. Fixed-Window vs Time-Between-Edges

## Fixed Time Window

Count encoder pulses during a known interval:

\[
\boxed{
\omega=
\frac{2\pi\Delta C}
{N_C\Delta t}
}
\]

### Advantages

- Simple
- Easy to implement
- Good at moderate/high speed

### Limitation

At low speed, only a few counts may arrive in each window. The resulting velocity becomes highly quantized.

---

## Time Between Encoder Edges

Measure the time between successive encoder edges:

\[
\boxed{
\omega=
\frac{2\pi}
{N_C\Delta t_{\text{edge}}}
}
\]

### Advantages

- Much better low-speed resolution
- One encoder transition can provide useful information

### Limitations

- Sensitive to timestamp jitter
- Sensitive to interrupt latency
- Phantom edges can produce large errors

### Practical approach

A robot can use:

- fixed-window counting at normal/high speed
- edge timing at low speed

provided the implementation is carefully characterized.

---

# 12. Encoder Calibration

The most important practical requirement is:

> **Measure, do not assume.**

The calibration should determine:

1. counts per wheel revolution
2. effective wheel radius
3. effective track width
4. left/right wheel mismatch

---

## 12.1 Measuring Counts per Wheel Revolution

### Procedure

1. Lift the robot or otherwise allow the wheel to rotate freely.
2. Reset the encoder counter.
3. Mark the wheel.
4. Rotate the wheel exactly 10 revolutions.
5. Record the encoder count.
6. Repeat several times.
7. Calculate the average.

The measured value is:

\[
\boxed{
N_C=
\frac{\text{measured counts}}
{\text{wheel revolutions}}
}
\]

### Example

If 10 wheel revolutions produce:

\[
59\,982\text{ counts}
\]

then:

\[
N_C=
\frac{59982}{10}
\]

\[
\boxed{
N_C=5998.2\text{ counts/rev}
}
\]

> The value above is an **illustrative example**. Replace it with the actual measurement from the robot.

---

# 13. Effective Wheel Radius

The geometric radius measured with a ruler is not necessarily the radius used by odometry.

A loaded wheel can deform, changing its effective rolling radius.

Starting from:

\[
D=
\frac{2\pi r\Delta C}{N_C}
\]

the effective radius is:

\[
\boxed{
r_{\text{eff}}
=
\frac{DN_C}
{2\pi\Delta C}
}
\]

For left and right wheels:

\[
\boxed{
r_L=
\frac{DN_C}
{2\pi\Delta C_L}
}
\]

\[
\boxed{
r_R=
\frac{DN_C}
{2\pi\Delta C_R}
}
\]

### Experimental procedure

1. Mark a starting point.
2. Command a known straight distance \(D\).
3. Record the encoder counts.
4. Measure the actual physical distance.
5. Repeat several times.
6. Calculate \(r_L\) and \(r_R\) separately.

---

# 14. Track Width Calibration

The effective track width is not necessarily the same as the CAD centre-to-centre dimension.

For an in-place rotation:

\[
\Delta s_R-\Delta s_L=b\theta
\]

Therefore:

\[
\boxed{
b=
\frac{\Delta s_R-\Delta s_L}
{\theta}
}
\]

Since:

\[
\Delta s=
\frac{2\pi rC}{N_C}
\]

we obtain:

\[
\boxed{
b=
\frac{2\pi r(C_R-C_L)}
{N_C\theta}
}
\]

For \(N\) complete rotations:

\[
\theta=2\pi N
\]

so:

\[
\boxed{
b=
\frac{r(C_R-C_L)}
{N_CN}
}
\]

---

## 14.1 Example

Assume:

\[
r=52\text{ mm}
\]

\[
N_C=6000
\]

and during one full in-place rotation:

\[
C_R=3050
\]

\[
C_L=-3050
\]

Then:

\[
b=
\frac{52(3050-(-3050))}
{6000}
\]

\[
\boxed{
b=52.87\text{ mm}
}
\]

> This number is only an example showing the calculation. Use actual measured counts for the final report.

---

# 15. Why a Robot Curves on a Straight Command

A differential-drive robot can curve even when the software sends equal left and right commands.

## Electrical cause 1 — Encoder scaling mismatch

If:

\[
N_{C,L}\neq N_{C,R}
\]

because of incorrect decoding or configuration, the computed wheel velocities will be different.

## Electrical cause 2 — Missed or phantom pulses

Motor-driver switching and EMI can produce:

- missed encoder transitions
- false transitions
- incorrect direction detection

This produces incorrect odometry.

## Mechanical cause — Different effective wheel radius

If:

\[
r_L\neq r_R
\]

then:

\[
v_L\neq v_R
\]

even when the encoder counts are equal.

Therefore:

\[
\omega=
\frac{v_R-v_L}{b}
\neq0
\]

and the robot gradually curves.

---

# 16. Why Velocity Becomes Noisy at Low Speed

At low speed, very few encoder counts arrive in each measurement window.

For example, a measurement sequence might look like:

```text
0  0  0  1  0  0  1  0  0 ...
```

The calculated velocity can therefore jump between discrete values.

This is **quantization**, not necessarily electrical noise.

Other possible causes include:

- encoder jitter
- timestamp jitter
- EMI
- missed pulses
- phantom pulses
- mechanical vibration

### Possible solutions

- increase encoder resolution
- increase the measurement window
- measure time between edges
- use appropriate filtering
- improve signal integrity

Any filtering or longer measurement window should be evaluated against the additional delay it introduces.

---

# 17. Encoder Selection

## Drive Wheel

A suitable choice is:

\[
\boxed{\text{Incremental magnetic quadrature encoder}}
\]

### Reasons

- Measures wheel motion directly
- Provides direction through A/B quadrature
- High resolution can be obtained
- Magnetic sensing is generally robust in dirty environments
- Absolute position is usually not essential for the drive wheel

---

## Arm Joint

A suitable choice is:

\[
\boxed{\text{Absolute encoder}}
\]

### Reason

The joint angle is available immediately after power-on, without requiring a potentially unsafe homing operation.

For higher precision applications, an absolute optical encoder may be appropriate. For compact and robust designs, a magnetic absolute encoder may also be suitable.

---

# 18. Encoder Mounting Errors

The physical installation of the encoder affects measurement accuracy.

### Shaft Runout

If the shaft axis moves during rotation, the encoder geometry changes periodically.

### Coupling Flex

A flexible coupling can introduce a difference between encoder angle and actual shaft angle, especially during acceleration and load changes.

### Magnetic Misalignment

For a magnetic encoder, errors can arise if the magnet is:

- off-centre
- tilted
- too far from the sensor

### Gearbox Backlash

After reversing direction, encoder motion at the motor shaft may not immediately produce equivalent wheel motion.

---

# 19. Wheel Slip

An encoder measures **wheel rotation**, not actual translation.

Suppose the encoder indicates:

\[
100\text{ mm}
\]

of wheel travel, but the wheel slips and the robot moves only:

\[
70\text{ mm}
\]

Then odometry still reports:

\[
100\text{ mm}
\]

while the real motion is:

\[
70\text{ mm}
\]

Therefore:

\[
\boxed{
\text{wheel odometry cannot directly detect wheel slip by itself}
}
\]

Slip can instead be inferred by comparing wheel odometry with an independent sensor such as:

- IMU
- LiDAR odometry
- visual odometry
- external motion capture
- GPS/RTK where appropriate

---

# 20. Error Budget

A useful error budget should identify the source, its effect and how it can be measured.

| Error Source | Type | Effect | Characterization |
|---|---|---|---|
| Effective wheel-radius error | Systematic | Distance/velocity scale error | Straight-line calibration |
| Track-width error | Systematic | Heading error | In-place rotation |
| Left/right radius mismatch | Systematic | Robot curves | Repeated straight runs |
| Encoder quantization | Quantization | Velocity granularity | Calculate distance/count |
| Missed encoder pulses | Electrical | Under-reported motion | Compare encoder and reference |
| Phantom pulses | Electrical | False motion | Logic analyser/scope |
| Wheel slip | Mechanical/environmental | Odometry drift | Compare with independent sensor |
| Gearbox backlash | Mechanical | Reversal error | Forward/reverse test |
| Shaft/coupling misalignment | Mechanical | Position error | Mechanical inspection |
| EMI | Electrical | Missing/false counts | Test near/away from motor wiring |

---

# 21. Error Propagation

For:

\[
s=\frac{2\pi rC}{N_C}
\]

a first-order relative uncertainty approximation is:

\[
\boxed{
\frac{\delta s}{s}
\approx
\sqrt{
\left(\frac{\delta r}{r}\right)^2+
\left(\frac{\delta C}{C}\right)^2+
\left(\frac{\delta N_C}{N_C}\right)^2
}
}
\]

For:

\[
\theta=
\frac{2\pi r(C_R-C_L)}
{N_Cb}
\]

the approximate relative uncertainty is:

\[
\boxed{
\frac{\delta\theta}{\theta}
\approx
\sqrt{
\left(\frac{\delta r}{r}\right)^2+
\left(
\frac{\delta(C_R-C_L)}
{C_R-C_L}
\right)^2+
\left(\frac{\delta N_C}{N_C}\right)^2+
\left(\frac{\delta b}{b}\right)^2
}
}
\]

---

# 22. Calibration Experiment

## Test 1 — Counts per Revolution

Perform at least three trials:

| Trial | Wheel Revolutions | Encoder Counts | Counts/Rev |
|---:|---:|---:|---:|
| 1 | 10 | *measure* | *calculate* |
| 2 | 10 | *measure* | *calculate* |
| 3 | 10 | *measure* | *calculate* |

Average the measured counts/revolution.

---

## Test 2 — Effective Wheel Radius

Command a known distance:

\[
D=1.000\text{ m}
\]

Perform at least five runs.

| Run | Commanded Distance | Measured Distance | Error |
|---:|---:|---:|---:|
| 1 | 1.000 m | *measure* | *calculate* |
| 2 | 1.000 m | *measure* | *calculate* |
| 3 | 1.000 m | *measure* | *calculate* |
| 4 | 1.000 m | *measure* | *calculate* |
| 5 | 1.000 m | *measure* | *calculate* |

Use the encoder counts and:

\[
r_{\text{eff}}
=
\frac{DN_C}{2\pi C}
\]

to estimate the loaded effective radius.

---

## Test 3 — Track Width

Perform several in-place rotations.

| Trial | Number of Turns | \(C_R\) | \(C_L\) | Calculated \(b\) |
|---:|---:|---:|---:|---:|
| 1 | 1 | *measure* | *measure* | *calculate* |
| 2 | 1 | *measure* | *measure* | *calculate* |
| 3 | 1 | *measure* | *measure* | *calculate* |

Average the measured effective track width.

---

# 23. Before vs After Calibration

The required comparison should contain both straight-line and rotational tests.

| Test | Command | Before Calibration | Error | After Calibration | Error |
|---|---:|---:|---:|---:|---:|
| Straight | 1.000 m | *measure* | *calculate* | *measure* | *calculate* |
| Rotation | 360° | *measure* | *calculate* | *measure* | *calculate* |

Error for a distance test:

\[
\boxed{
e=
\frac{D_{\text{measured}}-D_{\text{commanded}}}
{D_{\text{commanded}}}
\times100\%
}
\]

For a rotational test:

\[
\boxed{
e=
\frac{\theta_{\text{measured}}-\theta_{\text{commanded}}}
{\theta_{\text{commanded}}}
\times100\%
}
\]

---

# 24. Square-Path Calibration

A square-path experiment helps reveal systematic odometry errors.

Example:

```text
        1 m
   ┌───────────┐
   │           │
 90°           │ 90°
   │           │
   └───────────┘
        1 m
```

Run the square:

1. Start at a known pose.
2. Drive 1 m.
3. Rotate 90°.
4. Repeat four times.
5. Record the final position and heading.
6. Repeat in the opposite direction.

The two major intrinsic errors are:

- effective wheel radius
- effective track width

The problem specifically points toward **UMBmark or an equivalent square-path calibration** for separating these effects.

---

# 25. Practical Data Flow

```mermaid
flowchart LR
    A[Encoder A/B signals] --> B[Quadrature decoding]
    B --> C[Encoder counts]
    C --> D[Counts per wheel revolution]
    D --> E[Wheel displacement]
    E --> F[Wheel velocity]
    F --> G[Differential-drive kinematics]
    G --> H[v and omega]
    H --> I[Odometry x y theta]
```

---

# 26. Recommended System Configuration

| Subsystem | Encoder Type | Reason |
|---|---|---|
| Drive wheel | Incremental magnetic quadrature | Velocity + direction + robustness |
| Arm joint | Absolute encoder | Immediate joint angle after power-on |
| BLDC commutation | Hall sensors / encoder | Rotor position/commutation feedback |

---

# 27. Final Results Summary

### Fundamental conversion

\[
\boxed{
N_C=PPR\times D\times G
}
\]

### Distance per count

\[
\boxed{
d_{\text{count}}=
\frac{2\pi r}{N_C}
}
\]

### Wheel velocity

\[
\boxed{
v_w=
\frac{2\pi r\Delta C}
{N_C\Delta t}
}
\]

### Robot linear velocity

\[
\boxed{
v=
\frac{\pi r}
{N_C\Delta t}
(\Delta C_R+\Delta C_L)
}
\]

### Robot angular velocity

\[
\boxed{
\omega=
\frac{2\pi r}
{bN_C\Delta t}
(\Delta C_R-\Delta C_L)
}
\]

### Effective wheel radius

\[
\boxed{
r_{\text{eff}}=
\frac{DN_C}{2\pi C}
}
\]

### Effective track width

\[
\boxed{
b=
\frac{2\pi r(C_R-C_L)}
{N_C\theta}
}
\]

---

# 28. Key Conclusions

1. Encoder counts are only the starting point; calibrated robot motion requires correct values for \(N_C\), effective wheel radius and track width.
2. A factor-of-four error can occur if PPR, CPR and quadrature decoding are confused.
3. Wheel odometry measures wheel rotation, so wheel slip cannot be directly observed from wheel encoders alone.
4. Low-speed velocity measurements are strongly affected by encoder quantization when using a fixed sampling window.
5. A drive wheel generally benefits from incremental quadrature feedback, while an arm joint can benefit from absolute position feedback.
6. Calibration should be performed on the assembled, loaded robot rather than relying only on nominal CAD and datasheet values.

---

## 29. Submission Checklist

- [ ] Encoder type comparison
- [ ] PPR/CPR/X1/X2/X4 explanation
- [ ] Gear-ratio calculation
- [ ] Counts → \(v\) and \(\omega\) derivation
- [ ] mm/count calculation
- [ ] Velocity-resolution calculation
- [ ] Encoder selection for drive wheel
- [ ] Encoder selection for arm joint
- [ ] Measured counts/revolution
- [ ] Measured effective wheel radius
- [ ] Measured effective track width
- [ ] Left/right calibration
- [ ] Before/after straight-line test
- [ ] Before/after rotation test
- [ ] Error budget
- [ ] Square-path / UMBmark-style calibration
- [ ] Experimental data and plots
- [ ] Sources and datasheets

---

## Source

This document is based on **Inter-IIT Tech Meet — Electrical Problem Statements, Problem Statement 02: “Encoders, Odometry and Sensor Characterization”**, especially the sections covering encoder types, counts-to-velocity derivation, intrinsic calibration, error budget and required deliverables.
