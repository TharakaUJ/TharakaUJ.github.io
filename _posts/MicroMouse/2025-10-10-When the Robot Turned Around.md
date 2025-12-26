---
categories:
  - Robotics
  - Micromouse
tags:
  - Micromouse
  - ESP32
  - PID Control
  - Competition

header:
  overlay_image: https://tharakauj.github.io/static-files/posts/MicroMouse/N1K0.jpeg
---

I'm writing this with the hope that it's the last time I talk about the experience rather than the achievement. From here on, I want my articles to be about technical breakthroughs, not emotional autopsies. But before that happens, here's the story of how we failed again, and why we're more excited than ever.

<!-- ![Micromouse, held on hand](https://tharakauj.github.io/static-files/posts/MicroMouse/N1K0.jpeg) -->

---

## The Plan That Started Early… and Still Felt Late

This time, we began early. We spent weeks designing and fabricating a custom PCB, researching components, choosing coreless motors, and testing individual modules. The hardware team did an incredible job, the board worked beautifully.

The problem? We hadn't touched the software until almost a week before the competition.

There were mid-semester exams and other events. For some exams, we showed up without studying because every spare hour went into this micromouse. When we finally sat down to write the code, the clock was already ticking loudly.

---

## Our Software Architecture

We wanted a layered system, something clean, scalable, and modular. Our plan looked like this:

- **Layer 1 — Maze Logic**  
  The top layer deals with abstraction: turning, going straight one cell, detecting walls, and making decisions based on the map.

- **Layer 2 — Motion Control**  
  This layer translates high-level instructions into precise physical actions. When Layer 1 says "move one cell", Layer 2 ensures it happens accurately using sensors and feedback. It also exposes a clean interface: walls become true/false, not raw numbers.

- **Layer 3 — Hardware & Control Systems**  
  The lowest level. It reads TOF sensors, drives the motors, and runs the PID controller. This layer is closest to the metal, literally.

The idea was solid. And surprisingly, despite the rush, most of it worked.

---

## Velocity Profiling and PID

One of the proudest parts of this attempt was our velocity profiling technique. After calibration, it worked beautifully.

PID initially behaved well, too. But somewhere along development, something changed, maybe sampling frequency, maybe arithmetic overhead. Suddenly the PID wasn't as responsive as it was before. Not broken, just… not sharp.

We noted it. We planned to fix it.

But the universe had other plans.

---

## Disaster #1: The Board That Betrayed Us

As if tradition demanded it, disaster visited us again.

The wire connecting the battery broke at the solder joint. A simple break, but it created a surge that killed our ESP32.

To make matters worse, we had one full day left.

My friends somehow sourced another ESP32 by noon the next day. We had limited tools, shaky hands, and desperation. They manually soldered pins even without magnification and guesswork.

Hours vanished. Eighteen of them, gone.

Now we had a working board again.

But the clock was unforgiving.

---

## Disaster #2: The Algorithm That Refused to Behave

We still had a major bug in the maze-solving layer. I won't dive into details here, let's just say it was the kind of bug that lurks quietly and only shows up when your sanity is already stretched thin.

We debugged until the sky started changing color.

---

## The Last All-Nighter

There were several late nights during the build. But this one, this was the final overnighter.

After fixing the algorithm, we jumped straight into testing and tuning:

- PID recalibration
- Wall detection thresholds
- Clamping TOF readings
- Removing minor glitches
- Questioning our life choices

I had used a velocity profiling algorithm that, in hindsight, was too computationally expensive. It was just one line of code, and I kept telling myself I'd replace it. But sleep deprivation does strange things, I forgot.

I also forgot to remove a debugging tone that beeps every time the mouse detects a wall. Imagine a robot screaming at you through a maze. That was our mouse.

No food, no shower, no break, we just kept pushing.

---

## Competition Day

We rushed to the venue like zombies with a robot.

**Attempt 1:**  
The floor messed with our encoder counts. Long straights confused the mouse, but our dead-end correction saved us.

**Attempt 2:**  
Looked promising, until the mouse turned around where no wall existed. Why?

- Light interference?
- Sensor reflection issues?
- A logic flaw?

We couldn't test. Competition rules didn't allow experimentation with the environment.

Every attempt ended the same way, the mouse panicked at the same spot, convinced there was a wall that didn't exist. It was heartbreaking and strangely hilarious.

And just like that, we failed again.

---

## So… Why Are We Still Excited?

Because this wasn't a wasted effort.

Every failure was a data point:

- Timing issues in PID
- The importance of profiling code
- Sensor calibration under different lighting
- Never leaving software till the last week
- Hardware resilience
- Sleep is a feature, not a luxury

We didn't bring home a trophy. We brought home experience, painfully earned, deeply internalized, and oddly motivating.

We're not stepping away from micromouse. We're stepping closer.

Failure didn't kill the hype. It amplified it.