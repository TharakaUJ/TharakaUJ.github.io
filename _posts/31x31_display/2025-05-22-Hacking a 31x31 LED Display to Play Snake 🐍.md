## Turning a 31×31 LED Matrix Into a Retro Game Console

During a department music event, our seniors (CSE'22 batch) built a custom 31×31 LED matrix display with 961 bulbs to show synchronized light patterns. After the event, the display sat unused—so I decided to repurpose it for something nostalgic: playing Snake!

### The Vision

I wanted to transform this music-visualization hardware into a modular retro LED game console. My goals:

- Add more games in the future
- Simulate games on a PC when hardware isn’t available
- Make it easy for others to contribute or build their own games

### Architecture Overview

The project is organized into three main threads:

- **Render Thread:** Updates the physical LED matrix using a shared framebuffer.
- **Game Thread:** Runs the active game logic (Snake, Galaxia, Tetris, etc.).
- **Input Thread:** Listens for user inputs (up, down, left, right, select, back) via web-based controls.

The display and game logic are separated by a 31×31 2D framebuffer:

```cpp
<pixel type> framebuffer[31][31];
```
(*here I used a placeholder for type of the data because it might change in the future)

Games write to this array; the rendering thread handles converting it to the LED strip’s format (using FastLED on ESP32) or simulating it in a browser.

### Snake Gameplay

Here’s a video of Snake running on the 31x31 LED grid:

<video src="https://tharakauj.github.io/static-files/posts/31x31_display/snake.mp4" controls width="480" style="margin-top: 1em;">
    Your browser does not support the video tag.
</video>

Game logic highlights:

- Snake moves and grows when eating food
- Game over on self-collision
- Speed increases with score
- Score is displayed at the center when the game ends

### PC Simulation Mode

Hardware access was limited, so I built a PC simulation:

- ESP32 serves framebuffer data as JSON via a web server
- A web-based UI fetches and displays the framebuffer, mimicking the LED grid
- The same framebuffer structure works for both ESP32 and PC modes

### Adding More Games

After Snake, I added:

- **Galaxia:** Space shooter with falling enemies
- **Tetris:** Arrange falling blocks to clear lines

All games use the same structure: write to the framebuffer and listen for input. A menu screen lets you select the game using the input buttons.

### What’s Next?

Planned improvements:

- **New controller:** Add support for physical controllers or keyboards (currently using a mobile web interface)
- **Improved simulation:** Enhance PC/web simulation with audio and animations
- **More games:** Add Pong, and maybe a mini RPG!

### Conclusion

Repurposing leftover hardware into a playable game console was a rewarding experience. I learned about LED displays, modular architecture, and got to play retro games on a unique display.

---

**Want to contribute?**  
I’ll be open-sourcing this project soon! If you’re interested in building games, improving the simulation, or optimizing LED control, reach out!

---

**Check out the code:**  
[GitHub Repository – TharakaUJ/31x31_Display](https://github.com/TharakaUJ/31x31_Display)
