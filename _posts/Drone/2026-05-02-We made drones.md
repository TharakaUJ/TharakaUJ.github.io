---

categories:
    - Robotics
    - Drones
tags:
    - Drones
    - Robogames

header:
  overlay_image: https://tharakauj.github.io/static-files/posts/Drone/drone cover photo.jpeg

---

# Building the Drone Challenge for Robogames

This time we started something new.

Yes, I know there are already far too many unfinished projects lying around. But we wanted to do this anyway.

Every year, our department organizes a robotics competition called **Robogames**. This year, the technical committee was particularly ambitious. Not necessarily because I was part of it, but I was part of it.

One of the biggest additions we wanted to introduce was a **drone challenge**.

Traditionally, Robogames contestants focus only on writing code and solving the problem itself. The organizers provide the platform. That means the challenge is not about building hardware, but about programming and autonomous behavior.

That sounds simple until you are the one responsible for making the platform work.

Giving contestants a clean interface, maintaining drones that fly reliably, repairing hardware after crashes, fixing software issues, and troubleshooting every unexpected problem is a lot more work than it sounds.

## The Beginning

We started fairly early.

At first, we weren't even thinking about autonomy. Simply getting the drone into the air was a challenge.

None of us really knew what we were doing when we started. We got help from many people along the way, especially the big brothers at DXP Lab.

Initially, the drone would only fly if it remained connected to a laptop. Obviously, that was not practical. We couldn't exactly send a laptop along with the drone.

After a lot of parameter tuning and experimentation, we finally reached the point where the drone could fly using a standard RC transmitter.

That was our first major milestone.

## Enter the Companion Computer

The next step was autonomy.

Our plan was to use a Raspberry Pi as a companion computer connected to the Pixhawk autopilot.

Getting the Raspberry Pi connected was easy.

Getting the drone to actually take off autonomously was not.

The first time the companion computer connected, I only wanted to test a few MAVProxy commands. Everything appeared to work correctly. But commanding an actual takeoff was a completely different story.

Once again, we found ourselves digging through parameters and documentation.

From the beginning, I wanted the system to be safe:

* If the companion computer stopped responding, the drone should land.
* If RC control was regained, the drone should give priority to the pilot.
* The companion computer should not be able to arbitrarily change flight modes.
* Critical actions should remain under RC control.

I was extremely cautious because by that point we had already crashed the drone multiple times.

One crash was particularly painful.

The drone ended up on the roof of our department building.

That incident set us back significantly, not only because the drone was physically damaged, but because we were emotionally exhausted as well.

{% include figure image_path="/static-files/posts/Drone/drone%20on%20the%20floor.png" alt="Close-up of the drone on the floor with my leg in frame" %}


## The Unexpected Indoor Takeoff

Eventually, after enough trial and error, we managed to autonomously arm, take off, and land the drone using scripts.

One incident still stands out.

I was sitting on the floor in a lab with the drone beside me. One of my friends had SSH access to the Raspberry Pi and was testing a script remotely.

I did not expect the script to initiate a takeoff.

Suddenly I heard the arming tone.

My immediate reaction was:

"Oh no. It's actually going to fly."

I quickly grabbed one of the drone's landing legs while the motors spun up.

For several seconds I was effectively wrestling a drone that desperately wanted to leave the ground while someone hurriedly terminated the script.

Fortunately, nothing was damaged.

## Preparing for the Competition

By about a week before the competition, we had two drones capable of autonomously following a line using onboard vision and scripting.

At that point, we began letting contestants test their solutions.

We provided a sandbox environment accessible through SSH. Contestants received the exact same interfaces they would use in simulation:

* MAVLink telemetry streams
* Camera streams
* Companion-computer access

This meant teams could develop and test code in a nearly identical environment.

Of course, issues still appeared regularly: position estimate errors, loose connections, sensor problems, and configuration mistakes.

We became quite good at diagnosing and fixing problems on the floor within minutes.

## The Most Productive 30 Minutes

The day before the competition was chaos.

Five or six teams arrived wanting to test their solutions simultaneously.

The system had never been designed for that many users.

We also couldn't simply allow everyone into the same environment because teams could potentially access each other's work.

So we called a short lunch break. Just thirty minutes.

Within those thirty minutes, our team somehow pulled off what felt impossible.

One person configured a better 5 GHz Wi-Fi network capable of supporting many users.

The problem was that the router had no internet connection.

His solution was wonderfully improvised.

He connected another router through his laptop, bridged it to the campus network, adjusted the gateway configuration, and suddenly the entire network had internet access.

At the same time, several others worked on creating isolated persistent sessions for each team.

Before lunch ended, we had:

* Multiple concurrent users.
* Team-specific workspaces.
* Persistent storage.
* Isolation between teams.
* Network connectivity.

Looking back, that was probably the most impressive thirty minutes of the entire project.

## Competition Day

Throughout the event, small issues continued to appear.

Whenever something broke, one of us would be on the floor beside the drone with a laptop trying to diagnose the problem while contestants waited.

![Us fixing the drone during troubleshooting](/static-files/posts/Drone/fixing%20drone.png)

Most of the time we managed to get everything running again within minutes.

From the outside, it probably looked smooth.

From the inside, it felt like balancing on a tightrope.

At any moment:

* A sensor could fail.
* A parameter could change unexpectedly.
* A connection could come loose.
* A drone could crash.

For weeks I carried that stress around.

I knew how fragile the entire setup was.

There were nights when I literally had nightmares about things failing during the competition.

## Looking Back

Despite all the crashes, repairs, debugging sessions, and stressful moments, we accomplished something that initially seemed far beyond our experience.

We built a platform where contestants could write code, connect to a real drone, access camera feeds and telemetry, and test autonomous behavior without needing to understand every hardware detail underneath.

The contestants only saw the challenge.

Behind that challenge was a small team trying to keep everything together one fix at a time.

And somehow, most of the time, it worked.
