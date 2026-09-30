---
excerpt: "Turning a 961-bulb LED wall into a modular retro game console, starting with Snake."
header:
  overlay_image: https://tharakauj.github.io/static-files/posts/31x31_display/led_matrix_overlay_1.jpeg
tags:
  - LED Matrix
  - Games

---

## 🎮 Turning a 31×31 LED Matrix Into a Retro Game Console
During a department music event, our seniors (CSE'22 batch) built a **giant 31×31 LED matrix** with **961 colorful bulbs** to display synchronized light patterns. After the event, it was left unused, so I decided to turn it into a playable **Snake** game.

---

## 💡 The Vision

My goal: transform this sound-reactive wall into a **modular retro LED game console**. I wanted to:

- 🎮 Add more games over time
- 🖥️ Simulate the display on a PC (for development without hardware)
- 🧩 Keep the code modular so others can contribute

---

## ⚙️ Architecture Overview

The system uses **three threads**:

- **Render Thread**: Sends visuals to the LED matrix
- **Game Thread**: Runs game logic (Snake, Galaxia, Tetris, etc.)
- **Input Thread**: Handles user input via a web controller (your phone can be the D-pad)

All threads share a **31×31 framebuffer**:

```cpp
<pixel type> framebuffer[31][31];
```

Games write to this array; the renderer displays it on the LEDs or in a web browser. This separation makes adding new games easy, no changes to rendering needed.

---

## 🐍 Playing Snake on a Wall of Lights

Here’s Snake running on the LED wall! (Press play below.)

<div style="display: flex; align-items: flex-start; gap: 1em;">
    <video src="https://tharakauj.github.io/static-files/posts/31x31_display/snake.mp4" controls width="180" style="margin-top: 0.5em; border-radius: 0.5em; box-shadow: 0 0 8px rgba(0,0,0,0.1);">
        Your browser does not support the video tag.
    </video>
    <div>
        🐍 Snake slithers around the grid, eating food and growing.<br>
        💥 Game over if it crashes into itself.<br>
        ⏩ Speed increases as you score.<br>
        🎯 Score appears in the center at the end.<br>
        Nostalgic fun, on a glowing 961-pixel canvas.
    </div>
</div>

---

## 🧪 PC Simulation Mode

To develop without the hardware, I built a web-based simulator:

- The ESP32 serves the framebuffer as JSON
- A web UI fetches and displays it in real time

Same code, same framebuffer, just a different display!

---

## 🌌 Adding More Games

After Snake, I added:

- **Galaxia**: A pixel space shooter
- **Tetris**: Classic falling blocks

All games use the same pattern: write to the framebuffer, read input, done! A menu lets you switch games, no code upload needed.

---

## 🔭 What’s Next?

- 🕹️ New controllers: gamepads or keyboards
- 🎧 Sound effects & animations in simulation
- 🧠 More games: Pong, mini RPG, maybe multiplayer

---

## ❤️ Wrapping Up

A leftover event prop became a retro pixel playground. I got to build a console, play old favorites, and experiment with rendering and threading. There’s more to come!

---

## 🤝 Want to Contribute?

I’ll open-source the project soon. If you want to:

- Create LED games
- Improve the simulation
- Chat about pixel art

Let’s connect!

**🧑‍💻 [Check it Out on GitHub](https://github.com/TharakaUJ/31x31_Display)**
