---
categories:
  - Robotics
  - Battlebot
tags:
  - Battlebot
  - Competition
  - stm32
  
---

Last time, our battlebot used an ESP32 as its brain. It worked fine, but ESP32s are delicate. This year, I wanted something tougher, something worthy of a robot that gets physically attacked. So I picked the STM32 Blue Pill, which everyone swears is more resilient.

And since we had learned our lesson about jumper wires, I wanted to create a proper soldered circuit this time.


---

Somewhere along the journey, I had changed the flashing options. The STM32 stopped responding. I assumed I had fried the supposedly “resilient” chip.

We didn’t have a spare.

Panic mode activated.

I hacked together a fallback ESP32 circuit. Since I’d tested code before, I didn’t need to think much. It felt like betrayal, but at least we had something working.

Still refusing to accept defeat, I went hunting for another Blue Pill. For Sri Lankans, this is not a simple task. After a small pilgrimage, I found one.

I plugged it in.

It didn’t work either.

That’s when it hit me, I hadn’t burnt the old chip. I had burnt time. Hours of it. The issue was my own flashing settings.

Perfect. I had defeated myself.

While my teammates were grinding metal, drilling new armor, and doing the exciting mechanical work, I was soldering alone like a gremlin, chasing my dream PCB. To make it fancy, I added an RGB LED and a buzzer. Around 12 AM, the board was almost ready. All that remained was connecting it to the radio receiver.

Then reality struck.

Most drone receivers invert their TX signal to save power. The STM32 does not have hardware to invert the RX signal. And doing it in software is… painful. I tried inverting bits, patching libraries, even consulting AI, which hallucinated as hard as I did.

No success.

It was another all-nighter. My teammates were working on the robot. Me, still trying to bend physics.

By morning, we gave up and fell back to the ESP32 again. I fixed the power supply on the way to the competition. Yes, literally working in a moving vehicle.

We also had changed the weapon motor, and I forgot to test it. The ESP32 library had a known issue. The weapon refused to spin. Perfect timing.

Then, the universe took one more bite:

Our backup ESP32, the one we had just fallen back to, died. Maybe power surge, maybe fate. The brain was gone.

I still refused to quit. We had a Plan C: the receiver had PWM channels. If we could somehow wire them directly to the STM32, we might bypass all our previous problems.

With minutes left on the clock, we soldered cables directly to pins. It actually almost worked. But the power supply was still an issue, and this time, the STM32 really did die.

No more chips. No more time.

We weren’t beaten by another robot.

We were beaten by our own ambition.

And weirdly, that’s what makes it exciting.

Until next time.

![Midnight build, friends working on the robot at the middle of the night](/static-files/posts/Battlebot/A Battlebot Postmortem/making-all-nighter.jpeg)

