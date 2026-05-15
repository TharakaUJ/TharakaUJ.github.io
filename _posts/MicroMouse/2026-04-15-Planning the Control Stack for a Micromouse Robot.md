---
categories:
  - Robotics
  - Micromouse
tags:
  - Micromouse
  - Architechture
  - Robotics
  - Competition
---

Micromouse robots look deceptively simple from the outside. A small robot enters a maze, explores it, finds the shortest path, and eventually performs a high-speed run to the center. In practice, however, building a reliable micromouse involves several interacting systems working together in real time.

This post describes the software and control architecture I am planning for my micromouse project. The focus is not on the maze-solving algorithm itself, but rather on how the lower-level robotics stack will be organized to support reliable motion, localization, and future expansion.

The goal is to design the system in layers, where each layer abstracts complexity away from the layer above it.

---

# High-Level Architecture

At a high level, the software stack can be divided into several layers:

```text
Maze Solving Logic
        ↓
Path Planning / Motion Commands
        ↓
Localization + State Estimation
        ↓
Motion Control Loops
        ↓
Motor Driver Interface
        ↓
Hardware
```

The maze-solving logic should not need to know how motors are driven, how encoder ticks are counted, or how sensor timing works internally. Instead, it should operate on higher-level abstractions such as:

* current maze cell
* robot orientation
* estimated pose
* motion primitives
* wall information

Similarly, the lower-level hardware drivers should not know anything about maze-solving algorithms.

This separation makes the system easier to debug, extend, and optimize independently.

---

# Hardware Abstraction

The bottom layer of the system consists of hardware drivers:

* wheel encoders
* IMU
* TOF or IR distance sensors
* motor drivers
* timers and interrupts

These components will expose clean interfaces to the upper layers.

For example:

* Encoder drivers expose wheel tick counts or wheel velocities
* IMU drivers expose angular velocity and orientation estimates
* Distance sensors expose filtered wall distances
* Motor drivers expose wheel speed or PWM interfaces

The higher-level logic should never directly manipulate GPIOs or timers.

---

# Encoder Processing Using Hardware Timers

Wheel encoders are one of the most important sensors in the system because they provide local motion information.

The current plan is to connect the quadrature encoders directly to hardware timers configured in encoder mode. This allows the microcontroller to count encoder pulses in hardware instead of relying on software interrupts for every tick.

Using hardware timers provides several advantages:

* reduced CPU overhead
* more reliable counting at high speed
* improved timing consistency
* better velocity estimation

From the encoder counts, wheel displacement can be computed:

d=\frac{2\pi rN}{P}

where:

* ( r ) is wheel radius
* ( N ) is encoder counts
* ( P ) is counts per revolution

These wheel displacements can then be used for odometry.

---

# Differential Drive Modeling

The micromouse can be modeled as a differential drive robot.

The robot pose consists of:

* position in the world frame
* heading angle

The state is represented as:

(x,y,\theta)

The robot motion model is:

\dot{x}=v\cos\theta,\quad \dot{y}=v\sin\theta,\quad \dot{\theta}=\omega

where:

* ( v ) is forward velocity
* ( \omega ) is angular velocity

Wheel encoder readings provide estimates of wheel motion, which can be converted into linear and angular velocity estimates.

---

# Sensor Fusion for Pose Estimation

Encoders alone are not sufficient for accurate localization because wheel slip and accumulated integration errors introduce drift over time.

To improve local pose estimation, the current plan is to fuse:

* wheel encoder odometry
* IMU angular measurements
* wall sensor corrections

The IMU can provide short-term rotational stability, while the wheel encoders provide translational motion estimates.

The localization layer will combine these measurements to estimate the robot pose continuously during movement.

Initially, the fusion may use relatively simple filtering approaches, but the architecture should allow more advanced estimators later, such as:

* complementary filters
* Kalman filters
* extended Kalman filters

The important design goal is to separate sensor acquisition from state estimation logic.

---

# Coordinate Frames and Transformations

Different parts of the system operate in different coordinate frames.

Examples include:

* world frame
* robot frame
* sensor frame

The robot frame moves with the mouse, while the world frame remains fixed relative to the maze.

Higher-level maze logic should operate in the world frame. However, sensors naturally measure distances relative to the robot frame.

This means transformations between frames are required.

A point measured in the robot frame can be transformed into the world frame using rotation and translation:

\begin{bmatrix}x_w\y_w\end{bmatrix}=\begin{bmatrix}\cos\theta & -\sin\theta\\sin\theta & \cos\theta\end{bmatrix}\begin{bmatrix}x_r\y_r\end{bmatrix}+\begin{bmatrix}x_0\y_0\end{bmatrix}

This abstraction becomes increasingly important as the system grows more complex.

For example:

* wall measurements may exist in sensor coordinates
* motion control may operate in robot coordinates
* path planning may operate in maze coordinates

Keeping frame transformations explicit helps avoid many localization and control bugs.

---

# Control Loop Organization

Instead of using a single controller for all movement, the current design separates the control system into multiple specialized loops.

Different movement modes require different feedback strategies.

## Speed Control Loop

The lowest-level motion controller regulates wheel velocity.

This loop uses encoder feedback to maintain target wheel speeds and compensate for battery voltage changes or motor inconsistencies.

---

## Straight-Line Correction

When moving forward through a corridor, the robot must maintain heading stability.

This controller may use:

* encoder differences
* IMU heading
* side wall measurements

to keep the robot centered and aligned.

---

## Wall Following Controller

When walls are available, wall sensors provide additional positional correction.

This controller attempts to maintain equal spacing from nearby walls while moving forward.

This is particularly useful for reducing long-term drift.

---

## Turning Controller

Turns require a different control strategy from straight motion.

During turns, the robot primarily cares about rotational accuracy rather than wall alignment.

The turning controller may rely heavily on:

* IMU angular measurements
* encoder-derived angular displacement

to achieve accurate 90-degree or 180-degree rotations.

---

# Motion Abstractions

The higher-level planner should not directly command PWM values.

Instead, it should issue higher-level motion primitives such as:

* move forward one cell
* rotate 90 degrees
* align to wall
* perform speed run trajectory

These abstractions isolate maze logic from low-level control implementation details.

This separation also makes simulation easier because the higher-level planner can operate independently from the physical hardware drivers.

---

# Future Plans

The initial goal is reliability and modularity rather than raw speed.

Once the lower-level architecture is stable, future work may include:

* trajectory planning
* smoother cornering
* better state estimation
* faster speed runs
* simulation tools
* advanced sensor fusion
* adaptive control systems

A strong software architecture early on should make these improvements easier to integrate later.

---

# Closing Thoughts

One of the interesting aspects of micromouse is that it combines multiple engineering disciplines into a very small robot:

* embedded systems
* control systems
* real-time software
* localization
* robotics
* motion planning

Even though the maze-solving algorithm often receives most of the attention, the lower-level control stack is what ultimately determines whether the robot moves accurately and reliably inside the maze.

This post mainly documents the current design direction and architectural ideas for the project. The implementation will likely evolve over time as the robot is tested and improved.

## References

- Repository: https://github.com/N1K0-micromouse/n1k0_v0.1 — private; util development complete.

- Past blog: https://tharakauj.github.io/robotics/micromouse/When-the-Robot-Turned-Around/
- Past blog: https://tharakauj.github.io/robotics/micromouse/From-one-foot-mouse-to-micro-mouse/
