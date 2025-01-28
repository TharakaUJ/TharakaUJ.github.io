This article is based on my experience while trying to build a micromouse. I had a team of six, and since none of us had prior experience, we decided to start with a larger micromouse and then gradually scale it down to a smaller version.

## Gathering Components

We gathered some essential parts, including:

- An Arduino Mega (which we eventually replaced with an ESP32)
- Motor driver
- IR sensors (Time-of-Flight)
- Motors
- Battery

Since this was a larger model, we did not worry about the motor or battery size (which were comparatively huge).

## Challenges and Unexpected Issues

There are plenty of guides online on how to build a micromouse, so I won’t focus on that. Instead, I’ll share the challenges and unexpected issues we faced during the process.

### Scaling Down to a Smaller Model

The large-scale model performed well for its intended behavior. So, we decided to move to a smaller version. The chassis of the new model was made of cardboard, and the motors and battery were much smaller. The new model had only the width of an Arduino Uno.

One of my teammates spent an entire night wiring everything while I was trying to figure out how to upload the script to the Arduino from a Linux machine. The wiring was done, and we attempted to upload our code, but we received an error: **"Not enough SRAM in Arduino Uno."**

This was even before adding the maze-solving algorithm, so it became clear that the Arduino Uno wouldn't work for us.

### Switching to ESP32

We decided to use an ESP32 instead. The ESP32 is more powerful than the Arduino, with more SRAM, a higher clock speed, and built-in WiFi. While my teammates worked on the next prototype, I figured out how to upload code to the ESP32.

For Linux users, uploading to ESP32 is straightforward, but for Windows users, it was quite challenging.

In the process, we accidentally burned our sensors because we plugged the ESP32 in the wrong orientation, causing the sensors to receive **5V instead of 3.3V.** That set us back a bit, but we eventually proceeded with building another prototype.

### Implementing Over-the-Air (OTA) Updates

Oh... the prototype is working! But plugging and unplugging the ESP32 every time we wanted to upload code was annoying. So, we decided to enable a way to upload code over WiFi.

I tried a few examples from online blogs, but luckily, I found a library called **ArduinoOTA**, which was designed exactly for this purpose. I implemented the feature, and it worked well.

While doing that, we also realized that we needed a way to change variables while the micromouse was running. So, I wanted to implement **a WebSocket server** along with the OTA feature. However, when the WebSocket server was running, **ArduinoOTA stopped working correctly.**

After some debugging, I figured out that I had to stop the WebSocket server when it was time to upload new code. You can find how I did it [here]("/Websocket-and-OTA-code-update/").



---

This was an exciting project filled with unexpected challenges, but we learned a lot in the process. If you're planning to build a micromouse, expect to run into issues like these—and be ready to troubleshoot along the way!

