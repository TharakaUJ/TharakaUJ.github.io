---
categories:
  - Robotics
  - Micromouse
tags:
  - Micromouse
  - Competition
---

# Building a Micromouse: Challenges, Lessons, and Unexpected Issues  

This article is based on my experience building a micromouse. I worked with a team of six, and since none of us had prior experience, we decided to start with a larger micromouse and gradually scale it down.  

The goal of these articles is to document our learning process over time. This probably won’t be the best guide for optimizing your micromouse, but hopefully, I’ll be able to share more insights in the future.  

## Gathering Components  

We started by gathering some essential components:  

- **An Arduino Mega** (which we later replaced with an ESP32)  
- **Motor driver**  
- **IR sensors** (Time-of-Flight)  
- **Motors with encoders**  
- **Battery**  

Since this was a larger prototype, we didn’t worry too much about motor or battery size—both were relatively large.  

## Challenges and Unexpected Issues  

There are plenty of online guides on how to build a micromouse, so I won’t go into the basics. Instead, I’ll share the challenges and unexpected issues we faced along the way.  

### Scaling Down to a Smaller Model  

Our large-scale model worked well for its intended purpose, so we decided to move to a smaller version. The new model’s chassis was made of cardboard, with much smaller motors and a compact battery. The entire bot was only as wide as an Arduino Uno.  

One of my teammates spent an entire night wiring everything while I was trying to figure out how to upload the script from a Linux machine. Once the wiring was done, we attempted to upload our code—only to be met with an error:  

> **"Not enough SRAM in Arduino Uno."**  

This was before we even added the maze-solving algorithm, making it clear that the Arduino Uno wouldn’t work for us.  

**Edit:** Later, I found a micromouse that ran on an Arduino Nano. I learned that optimizing code can help reduce SRAM usage, and flash memory can store maze data so it doesn’t reset every time.  

### Switching to ESP32  

We decided to switch to an ESP32 instead. With more SRAM, a higher clock speed, and built-in WiFi, it was a clear upgrade from the Arduino. While my teammates worked on the next prototype, I figured out how to upload code to the ESP32.  

For Linux users, the process was straightforward, but Windows users had a much harder time.  

In the process, we accidentally burned out our sensors by plugging the ESP32 in the wrong orientation. That setback cost us some time, but we eventually moved forward with a new prototype.  

### Implementing Over-the-Air (OTA) Updates  

Oh… the prototype was finally working! But constantly plugging and unplugging the ESP32 to upload code quickly became annoying. So, we looked for a way to upload code over WiFi.  

I tested a few examples from online blogs before finding **ArduinoOTA**, a library designed specifically for this purpose. It worked perfectly.  

While implementing OTA updates, we realized we also needed a way to adjust variables while the micromouse was running. I wanted to add a **WebSocket server**, but when it was active, **ArduinoOTA stopped working correctly.**  

After some debugging, I figured out that I needed to stop the WebSocket server whenever we uploaded new code.

## The Final Stretch Before the Competition  

With less than two weeks before the competition, we built our final prototype—much smaller for more precise turns. However, in the process, we **ruined our remaining IR (ToF) sensors** (yeah, we need to improve our building process). At the same time, our **motor encoders became unreliable.**  

To make things worse, we were now way over budget. So, we decided to work with what we had.  

We had a bunch of **ultrasonic sensors**, which were slower but got the job done. I had to **reduce the bot’s speed significantly**. After a few hours of tuning, we finally got it to move in a straight line.  

We also had a **gyroscope sensor lying around**, so we used that to improve turning accuracy. We could have used it to measure distance traveled, but we didn’t finish implementing that feature before the competition.  

### The Competition  

We made it to the competition. Even though we had the maze-solving algorithm coded, our encoders were unreliable, so we had to remove the **flood-fill algorithm**. Our bot was now just a simple wall-avoiding robot—basically a zombie blindly navigating the maze.  

I didn’t have high hopes, but somehow, **it didn’t crash into any walls!** We stopped it after about four minutes because we hadn’t implemented the full algorithm.  

Even though we didn’t win, the experience wasn’t a complete disaster. We got to see some of the best micromouse designs, learn about their challenges, and explore different levels of sophistication in micromouse engineering.  

## Final Thoughts  

This project was filled with unexpected challenges, but we learned a lot along the way. If you’re planning to build a micromouse, expect plenty of obstacles—and be ready to troubleshoot every step of the way!  

Stay tuned to see if we improve our micromouse!
