---
title: "From Neural Networks to Hardware: Our DVCon Journey (Part 2)"
excerpt: "The later stages of the DVCon Design Challenge: systolic arrays, AXI, a RISC-V processor running inside an FPGA, and a very long final night."

categories:
  - Hardware
  - DVCon
tags:
  - DVCon
  - FPGA
  - RISC-V
  - Machine Learning
  - Competition

header:
  overlay_image: /static-files/posts/HDL_Desings/DVConDesignCompetition/part2/bus-detections.png
---
## Getting a Better Picture

And we moved on to the next stage.

The organizers gave us a somewhat better idea of how everything was supposed to work. To be honest, looking back now, it all seems rather obvious. But I still remember how exciting it was back then. We were being introduced to so many new concepts, and almost everything felt unfamiliar.

As we progressed through the next few stages, we slowly started to understand how all the pieces connected. Not completely, of course, but our understanding grew with every step.

It wasn't just hardware that I was learning about. I was also getting deeper into neural networks, particularly CNNs, and how their operations could be mapped to general matrix multiplication (GEMM). From there, we explored how GEMM operations could be implemented using something called a systolic array.

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/conv-to-gemm.svg" alt="A 3×3×3 patch of the input is flattened into a row of matrix A, which is multiplied by weight matrix W to give output matrix C" caption="Each kernel position becomes a row of A and each kernel a column of W, so the whole convolution becomes one matrix multiplication." %}

Of course, we eventually decided to use a systolic array. Partly because we had heard about it in one of our modules, and partly because it seemed like a natural fit for the convolution operations used in YOLO.

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/systolic-array.svg" alt="A 4×4 grid of processing elements. Activations flow in from the left and partial sums flow downward to the results" caption="A weight-stationary systolic array, drawn at 4×4. Activations move right, partial sums move down, and each column ends with one result. Ours started at 32×32 and was cut to 24×24 to fit on the FPGA." %}

But while we were getting a clearer picture of the hardware and software separately, I felt like we were missing something in between.

YOLO was running in Python, and our accelerator was, well, hardware. Even if I managed to build something that could perform a GEMM operation, how would I get YOLOE, which was currently running in Python, to actually use it? How would the processor know when to delegate work to our accelerator? How would we connect everything together?

We had a working software pipeline and were slowly figuring out the hardware. But we hadn't quite figured out how to bring them together.

## Exploring Different Ideas

Meanwhile, we were still experimenting with different approaches.

On the software side, we wanted to explore alternatives to the small language model component we had initially used for prompt processing. We looked into knowledge graphs and experimented with different CLIP models, including MobileCLIP.

We were also exploring ways to make the accelerator itself more efficient.

Remember how I mentioned that YOLO mostly uses 1×1 and 3×3 convolutions? While researching these operations, I came across an algorithm called **Winograd's algorithm**.

In simple terms, Winograd's algorithm reduces the number of multiplications required for certain convolution operations, at the cost of additional additions and some computational overhead. Since multiplication is relatively expensive in hardware, this seemed like a promising direction.

I went quite deep into trying to understand and implement it. I think I was almost there, but it was taking a considerable amount of time.

Then we had a moment of realization. We hadn't even built a working accelerator yet! Spending so much time optimizing something before having a basic implementation seemed a little absurd.

So we decided to put those optimizations aside and focus on building a basic accelerator first. We could always come back to them later.

## The Microcode Detour

We had another idea, though. What if we could get almost the entire YOLOE execution pipeline to run on the hardware itself, with minimal intervention from the CPU?

Part of this idea came from our lack of understanding of how the processor was supposed to delegate work to the accelerator. Having the accelerator handle everything seemed like a simpler and potentially faster approach.

Unfortunately, we underestimated how difficult this would be.

A simple finite state machine (FSM) was hardly going to be enough to manage all the operations and dependencies involved in executing a neural network. We needed something more flexible.

That's when we started thinking about microcode.

The idea was to have a sequence of microinstructions describing the operations the accelerator needed to perform. We could potentially translate the YOLO execution into these instructions, store them somewhere, and let the accelerator execute them.

In theory, this would give us a programmable accelerator without having to hardwire every single operation into a fixed sequence of states.

It sounded pretty cool.

In practice, though, it was much more complicated than we expected. Getting the controller and state machine to correctly fetch, interpret and execute those instructions was particularly challenging.

For a first attempt at building a hardware accelerator, we had probably bitten off more than we could chew.

Eventually, we decided to step back and focus one step at a time.

## Understanding the FPGA Architecture

I should probably explain how everything was organized inside the FPGA, because this was another important part of what we learned.

The organizers provided us with a processor and most of the surrounding infrastructure. The CPU, accelerator, memory interfaces, UART, Ethernet interfaces, AXI interconnects and other supporting components were all part of the FPGA design.

The FPGA itself was essentially hosting an entire small computer system.

There was a RISC-V processor, memory, peripherals and, eventually, our custom accelerator. The processor could communicate with these components through memory-mapped interfaces.

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/vega-soc.svg" alt="VEGA SoC block diagram: the AS1061 core and our accelerator connect through a 64-bit AXI4 interconnect to PLIC, Ethernet, DDR3, boot ROM, RAM and UART" caption="The VEGA SoC. Our accelerator is the only block we built, and it connects as both a master and a slave." %}

One of the things I found particularly interesting was how each module had its own address space. There were address regions for RAM, ROM, the accelerator, UART, Ethernet and other peripherals.

| Component | Start address | End address |
|---|---|---|
| Boot ROM | `0x0001_0000` | `0x0001_7FFF` |
| Internal RAM | `0x0002_0000` | `0x0003_FFFF` |
| UART | `0x1000_1000` | `0x1000_1FFF` |
| 1G Ethernet | `0x2003_0000` | `0x2003_FFFF` |
| Accelerator | `0x2004_0000` | `0x2004_FFFF` |
| DDR3 controller | `0x8000_0000` | `0xBFFF_FFFF` |

This meant that, from the processor's perspective, communicating with these modules was often just a matter of reading from or writing to particular memory addresses.

That was fascinating to me. I had never really thought about how communication between different hardware components could be organized this way.

The communication infrastructure used AXI, which was also relatively new to me.

AXI is an industry-standard interface protocol commonly used to connect different components in an SoC. It uses a handshake mechanism based on VALID and READY signals, allowing components to communicate while accounting for differences in when they are ready to transfer data.

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/axi-handshake.svg" alt="Timing diagram of CLK, VALID, READY and DATA. Data transfers only in the cycles where VALID and READY are both high" caption="The sender holds its data steady until the receiver is ready. A transfer only happens in a cycle where both signals are high." %}

There are different AXI interfaces for different purposes, including memory-mapped communication and high-throughput data transfers.

The organizers had already configured the AXI interconnect, address spaces and most of the supporting infrastructure. We were given the interfaces we needed to connect our accelerator.

This was exactly the missing piece we had been looking for.

We were slowly starting to understand how the processor could communicate with our accelerator and delegate computationally intensive operations to it.

## Building the Actual Accelerator

Over the next few stages, we started putting everything together.

We connected AXI interfaces to our accelerator, including the master and slave interfaces, and tested them using the dummy code provided by the organizers.

Then we gradually started integrating the actual computational components.

We had our systolic array for GEMM operations, followed by a vector unit for operations such as activations. We implemented the necessary datapaths, buffers and supporting logic.

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/accelerator.svg" alt="Accelerator block diagram: slave interface to controller, master interface to weight and activation buffers, then systolic array, accumulation buffer, vector unit and output buffer back to the master interface" caption="Inside the accelerator, simplified. Data comes in from DDR through the master interface, passes through the array and vector unit, and goes back out the same way." %}

But one thing kept giving us trouble: the controller.

Everything was starting to look like a pipeline. Different resources could perform different operations, and I wanted to make sure we weren't leaving hardware idle unnecessarily.

For example, while one part of the accelerator was waiting for data, another part might already be able to perform some useful work.

So I tried building a controller that could issue up to four instructions, taking into account resource availability and data dependencies.

This introduced a whole new set of problems.

We had to keep track of which instructions were executing, which resources were occupied, and whether the data required by an instruction was actually available. We also had to prevent data corruption when multiple operations were happening simultaneously.

This meant implementing things like dependency tracking and a scoreboard.

It was getting complicated rather quickly.

There was another interesting learning experience along the way. Until then, most of the Verilog I had written was structural. I was used to connecting modules, instantiating components and wiring signals together.

Now I was getting into behavioural Verilog.

Initially, the concept of behavioural logic didn't make much sense to me. But once I started writing it and understood how it worked, things began to click. It was actually quite fun.

Looking back, behavioural logic is now the approach that makes the most sense to me when designing more complex control logic.

With some help from AI tools, particularly given how tight our deadlines were becoming, I eventually managed to get the controller working.

We tested it repeatedly, and there were quite a few bugs along the way. Some of them required manually going through waveform diagrams, tracing signals and figuring out exactly where things were going wrong.

But eventually, we got there.

**We could finally perform a GEMM operation using our own hardware accelerator.**

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/gemm-tests-passed.png" alt="Terminal test report: 29 checks run, 29 passed, all tests passed" caption="Our test program for the 24×24 array: signed values, stale state, random matrices against a CPU reference, and a repeatability check. 29 out of 29." %}

## The Night Before the Deadline

There was just one problem.

By the time we got our accelerator to perform GEMM, it was already the night before the final competition deadline.

We were among the top 12 teams, and only six would be selected to move on to the next round.

We had access to a remote FPGA board, which we had to share with another team. The organizers had provided a working processor system, and we could access a remote server in India through a remote desktop connection. The FPGA was connected to the server through UART and Ethernet.

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/remote-setup.svg" alt="Our laptops connect over the internet to a remote server in India, which connects to the Genesys 2 board through UART and Ethernet" caption="Our remote setup: everything went through that one server." %}

We could upload different FPGA bitstreams, run our software and test things through the UART interface.

We started by testing the system with a dummy accelerator.

And then something happened that genuinely surprised me.

We realized that once we uploaded the bitstream to the FPGA, we could actually interact with the processor running inside it through the UART interface. We could execute terminal commands on the RISC-V processor itself!

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/uart-terminal.png" alt="Remote desktop with a Minicom session over ttyUSB2 showing a shell on the board, listing files and starting run_on_board.sh" caption="The window at the top right is Minicom on a serial port, and that `#` prompt is Linux running on the RISC-V core inside the FPGA." %}

OMG.

I know, this probably sounds completely normal to anyone who has worked with embedded systems or SoCs. But for us, seeing an entire processor system running inside an FPGA and being able to interact with it through a terminal was incredibly cool.

Nothing unexpected was happening, of course. But it was one of those moments where something you had previously only learned about theoretically suddenly became real.

There was another funny thing that happened while we were using the remote setup.

Since we had to share the remote desktop with another team, we eventually came across the people on the other end. We started calling one of them "Bro".

It started when he sent us a message along the lines of:

"Bro... sorry, we know it's not our time now, but we just need to get our files and stuff."

We ended up communicating through an open text file on the remote desktop. He would type something, we'd read it and reply in the same file. It was almost like having a conversation through a shared diary.

At some point, we even got to know their names, and one of my friends shared his Instagram with them.

The whole thing reminded me a little of Tom Riddle's diary from Harry Potter. Except, instead of communicating with a mysterious person through a magical diary, we were talking to another team through a text file on a remote desktop, somewhere on the other side of the world.

Eventually, Bro went offline, and that was that.

It's a pretty small thing, but it's one of those random memories from the competition that I still find amusing.

## Meanwhile, on the Software Side...

While we were working on the hardware, we were also trying to figure out how to get YOLOE running on the provided processor and accelerator.

We couldn't just leave everything in Python. We needed some way to translate the model's operations into something our hardware could execute.

We initially started by porting YOLOE to C. Our thinking was that if we could get it working in C, we might be able to compile it for the provided RISC-V processor and eventually find a way to translate the computationally intensive operations into our accelerator's instruction format.

It was a bit of a roundabout plan, but we wanted to get something working.

Eventually, we successfully ported YOLOE to C, and it worked!

Well, on our laptops, at least.

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/c-port-log.png" alt="Terminal log from the C runtime: 384 instructions and 629 tensors loaded, 5 detections above confidence 0.25" caption="YOLO26n in plain C, with no Python involved." %}

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/bus-detections.png" alt="A street scene with bounding boxes around a bus and four people" caption="And its output: the bus and four people, drawn from those five detections." %}

Next, we tried compiling it for the provided RISC-V processor.

And yes, it ran.

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/board-process-list.png" alt="ps output on the board showing yoloe_detect running under run_on_board.sh" caption="`yoloe_detect` as PID 264, running on the board. We never got a complete run on the FPGA before the deadline." %}

But it was incredibly slow.

It took so long that it became quite obvious why the competition required us to build a hardware accelerator in the first place. Running an entire object detection model on a small embedded processor without acceleration simply wasn't practical.

Still, getting it to compile and execute on the target processor was a useful milestone. We had at least demonstrated that the software could run on the target architecture.

What we hadn't managed to do yet was connect its computationally intensive operations to our accelerator.

## When Reality Catches Up

By this point, things were becoming increasingly difficult.

We had a working GEMM accelerator, a C implementation of YOLOE, and a processor system that we could interact with. But we still had a considerable amount of work left to integrate everything.

And we had one night.

Even synthesizing the FPGA design could take hours. Every major change meant potentially waiting a long time before we could test it again.

{% include figure image_path="/static-files/posts/HDL_Desings/DVConDesignCompetition/part2/file-transfer.png" alt="Terminal showing files copied to the board at a few KB/s, with weights.bin taking 22 minutes 53 seconds" caption="Synthesis took 2–3 hours, and copying files wasn't quick either. At about 7 KB/s, `weights.bin` alone took almost 23 minutes." %}

We knew that completing everything within the remaining time was probably unrealistic.

But we didn't want to give up.

We agreed to show whatever we could get working and make the best of what we had.

So, as usual, we stayed up the entire night.

At some point, though, we started becoming increasingly unproductive. One by one, we started falling asleep. After another hour or two of barely getting anything done, we finally decided to put together our report.

And, as usual, we ended up doing that last part sleep-deprived, barely functioning, and probably looking like zombies.

Somehow, we submitted everything.

Unfortunately, we didn't make it into the final six.

Maybe we messed up the report. Maybe our implementation wasn't good enough. Or maybe there was simply too much left to complete in the time available before the finals, which were scheduled for around the first week of September.

I don't really know.

## Was It Worth It?

Absolutely.

Even though we didn't make it to the finals, I learned an incredible amount throughout this competition.

Not just about hardware design. Not just about software. Not just about machine learning or CNNs.

I learned how all these things connect.

I went from barely understanding how an FPGA-based system was organized to working with AXI interfaces, designing datapaths, implementing a systolic array, debugging behavioural Verilog and getting a custom accelerator to perform GEMM operations.

I also learned how much there is between having a neural network running in Python and actually getting it to execute efficiently on custom hardware.

And perhaps most importantly, I got a glimpse of what it feels like to build something that spans multiple layers of engineering, where understanding one component isn't enough. You need to understand how all of them work together.

## What I'd Do Differently

Looking back, there's one thing I'd tell my past self: **start with GEMM.**

Instead of trying to make the entire YOLOE execution pipeline work on our custom hardware, we should have started by getting the processor to offload just the matrix multiplication operations to our accelerator.

Let the CPU handle the rest of the YOLO logic. Get the basic communication working, make sure the accelerator can receive data, perform GEMM and return the results correctly. Then gradually move more operations into hardware as we gain confidence.

In hindsight, this seems almost painfully obvious.

We had the time, the hardware and, eventually, enough understanding to build a working GEMM accelerator. What we lacked was a sensible progression of milestones.

We were trying to solve too many problems simultaneously: designing the accelerator, figuring out the processor interface, implementing the controller, translating the neural network and optimizing the computations.

If we had started with a simple matrix multiplication offload and built everything incrementally from there, I genuinely think we could have achieved much more within the time we had.

But I'm not entirely sure my past self would have listened.

I guess these are classic engineering mistakes. You get excited about an ambitious idea, underestimate the complexity, and try to build too much at once. And I'm pretty sure I'll make these mistakes again, several times over.

Hopefully, I'll get a little better at recognizing them each time.

We didn't get the result we wanted, but I wouldn't trade that experience.

I went into this competition knowing very little about hardware accelerators. I came out with a much better understanding of how processors, memory, buses, hardware datapaths, neural networks and software all fit together.

More importantly, I got to experience the entire process of trying to build something ambitious, making mistakes, hitting unexpected problems, and figuring things out along the way.

There was still a lot left to learn.

And honestly, that's probably my favourite thing about the whole experience.