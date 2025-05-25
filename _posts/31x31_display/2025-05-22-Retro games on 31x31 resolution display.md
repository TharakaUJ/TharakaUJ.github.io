There was a musical event from our department. And for that, the immediate senior batch of ours, the cse'22 created a display with 961 bulbs, a 31x31 display to show variuos patterns with the music. The event was soon over and I saw the display hanging around and wanted something childish. Yes, I wanted to play the snake game in it.

The next weekend, I started looking into how the LED strip in the display works. And I found a library that makes it easier for me to handle the display. Then I thought of how make things more modular. 

I thouhgt of having a seperate function loop for rendering into display and seperate loop for games and a seperate loop for taking inputs and outputs. Therefore it has three main threads.

1. Render thread
2. Game thread
3. Input thread

there was also anotehr issue, I don't have access to the display everytime. So i wnted to create a simulation for the game so that I can easily debug my games. So using esp32 as a webserver I created a endpoint that serves the information about the display.

Ah.. speaking of display I did something there too. The game will write the display information to 31x31 array and thn from the render loop it will be convertede to long 1D array which can be fed as data to the display.

By this method I can easily add a new game without having to code for redndering logic each time. Also I used the same 31x31 array which I call framebuffer, for the simulation too.

Later I add few more games, galaxia and tetris. and a menu to select fom these 3 games. 

for the future iprovements I am trying tohave a better controller, because now we are usinga mobile phone connected to oe of the enpoints of esp32 webserver. 

Also i am trying to add better simulation options and many more games. until next time!