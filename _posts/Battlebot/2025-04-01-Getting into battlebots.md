---
categories:
  - Robotics
  - Battlebot
tags:
  - Battlebot
  - Competition
---

This is the story of how I got into BattleBots, and how we faced our very first competition.

A few of my friends and I had wanted to build a BattleBot for a long time. We tossed around several ideas: something like Minotaur, the powerful Brazilian spinner; or Duck, simple but surprisingly deadly.

Eventually, we agreed on a design that somewhat resembled Tombstone:  
A horizontal spinning blade at the front, with two wheels enclosed and protected at the back.

## The Build

We used a Radiolink transmitter and receiver, commonly used for drones, as our remote control system. I was responsible for the electronics and programming, while my friends focused on the design and hardware (which is arguably the hard part... it is a robot, after all).

The bot ended up working perfectly. Controls were responsive, and we added a few switches:

- A kill switch for the weapon
- A main kill switch for the robot
- A start-up sequence after shutdown
- Inverted control logic in case the bot got flipped

## The Competition Day

The day of the competition arrived. Up until that morning, we didn’t have a power supply strong enough to charge our batteries quickly. So I had to go out searching for one, and luckily, I found it just in time.

<div style="text-align: center;">
    <img src="https://tharakauj.github.io/static-files/posts/Battlebot/GettingIntoBattlebots/battlebot%20random.jpeg" alt="Battlebot Random" style="width: 80%; max-width: 600px; height: auto; aspect-ratio: 16/9; object-fit: cover; object-position: 0px -350px;">
</div>

My friends did a fantastic job on the robot. The weight limit was 10 kg, and our bot came in at 9.97 kg, perfectly balanced.

### Round 1: The Flipper

Our first opponent was a flipper bot. It didn’t look that fast or destructive, so we felt confident.

But before we could even spin up our weapon, it rushed straight at us and flipped us over.  
Turns out the flipper wasn’t slow, we were just fooled.

We lost the round quickly.

### Between Matches

We had another match coming up, so we quickly bolted an iron bar to the bot to prevent it from getting stuck sideways. That was my first taste of the fast-paced, frantic nature of mid-competition repairs.

In all the rush, we forgot to test the weapon.  

No time left, we had to go in blind.

### Round 2: Weapon Failure... and Luck

Right before the match started, our driver was checking the controls...  

Oh no! the weapon wasn’t working.

We tried restarting, but no luck.

<div style="text-align: center;">
    <img src="https://tharakauj.github.io/static-files/posts/Battlebot/GettingIntoBattlebots/battlebot%20round%202.jpeg" alt="Battlebot 2nd round" style="width: 80%; max-width: 600px; height: auto;">
</div>

Fortunately, our opponent’s weapon also malfunctioned. We won that round by simply pinning them and avoiding damage.


## The Fixing Frenzy

We rushed back to our desk.  
What’s wrong with the weapon?

We started testing everything: checking voltages, ports, and microcontroller logic. Some of us were tearing off covers, others probing terminals with multimeters. A few other teams joined in to help, holding torches and tools. I was debugging the microcontroller via laptop, it was working fine.

The issue had to be with the motor driver.

Luckily, we had a spare.

The organizers announced we had 2 minutes left.

We swapped the driver. Still nothing.

Then we realized, the connections were loose!

No time to properly mount it, so we hot-glued it in place. No time to test. We threw the bot back together and ran to the arena.

### Round 3: Revenge Shot

We got to test the weapon just before the match. It spun up perfectly.

Our operator revved it up and went straight for the opponent’s wheel.  
**Boom.** The wheel flew off. Their motor shaft was twisted and destroyed.

<div style="text-align: center;">
    <video controls style="width: 80%; max-width: 600px; height: auto;">
        <source src="https://tharakauj.github.io/static-files/posts/Battlebot/GettingIntoBattlebots/final%20battle.mp4#t=0,15" type="video/mp4">
        Your browser does not support the video tag.
    </video>
</div>

But it came at a cost.

Our motor mount wasn’t strong enough and snapped. The chain driving the weapon was loose, and the blade stopped. Thankfully, nothing else was damaged.

<div style="text-align: center;">
    <img src="https://tharakauj.github.io/static-files/posts/Battlebot/GettingIntoBattlebots/battlebot%20chain%20wrecked.jpeg" alt="Battlebot Chain Wrecked" style="width: 80%; max-width: 600px; height: auto;">
</div>

It became a pushing match.

## Final Result

The judges awarded the match to our opponent.

And that was our final battle of the day.

## Looking Ahead

It was a wild ride, full of mistakes, adrenaline, and last-minute hacks. But we learned a lot.

Can’t wait to take part in more of these.
