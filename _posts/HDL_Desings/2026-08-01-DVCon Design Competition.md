# Building a Hardware Accelerator for Prompt-Based Object Detection – Our DVCon Journey

The Design and Verification Conference (DVCon) is a premier event for the design and verification community, bringing together engineers, researchers, and industry experts to share knowledge, best practices, and innovative solutions. The DVCon Design Competition is an exciting opportunity for participants to showcase their skills in digital design and verification through a series of challenging tasks and projects.

This year, we participated in the Design Challenge as a group of second-year Computer Science students who had only completed two modules on digital design and verification and computer architechture. Despite our limited experience, we were eager to take on the challenge and, more importantly, learn from the process.

The challenge was to build a hardware accelerator for a given processor to perform object detection. But not just ordinary object detection: it had to be **prompt-based**.

For example, we would be given an image and a prompt such as "something to eat". Our system would then have to identify the object that best matched the prompt and draw a bounding box around it.

![A kitchen counter with the query "Something to drink tea from" and a bounding box around a cup](/static-files/posts/HDL_Desings/DVConDesignCompetition/prompt2.jpg)
*Query: "Something to drink tea from". The pipeline picks out the cup.*

To keep the scope manageable, the competition limited us to 80 COCO classes.

The challenge had several stages. I'll try to go as deep as possible into each of them without making this post unnecessarily boring.

## The First Stage: Choosing Our Approach

First, we had to decide which approach we were going to take. We had several ideas to explore. The challenge itself referenced a research paper that used approaches similar to Faster R-CNN.

The thing was, eventually, we would have to implement at least part of the neural network in our accelerator. But we didn't really have a clue which part we should implement, where the processor would fit into the picture, or how the processor and accelerator would communicate with each other.

After doing some more research, we came across several variations of YOLO models. What eventually caught our attention was the nano-sized models, particularly YOLOE and YOLO-World.

The YOLOE-26n model seemed particularly suitable for our task. It was already available in an INT8 quantized version, supported the 80 COCO classes we needed, primarily used 3×3 and 1×1 convolutions, and was relatively small compared to many other object detection models.

You might wonder what quantization is. We also came across another concept called pruning.

Let's start with quantization.

Normally, neural networks use floating-point numbers to represent their weights. A typical FP32 number takes 32 bits. But do we really need all those bits to represent every weight? Probably not.

That's where quantization comes in. Instead of representing weights using 32-bit floating-point numbers, we can use lower-precision representations, such as 8-bit integers (INT8). This reduces the amount of memory required to store the weights and can also make computations cheaper, which is particularly useful when trying to fit a neural network into a relatively small hardware accelerator.

Of course, it's not as simple as throwing away the other 24 bits and calling it a day! There are different ways of performing quantization. One approach is Quantization-Aware Training (QAT), where the model is trained while accounting for the effects of quantization. Luckily, we found that an already-quantized version of YOLOE was available.

Then there's pruning. This is another technique used to make neural networks smaller and more efficient. In simple terms, pruning removes parts of a neural network that contribute relatively little to its output. For example, structured pruning can remove entire channels from a network.

*Structured pruning: whole channels are removed, not just individual weights.*

Wait, what's a channel?

When talking about neural networks, you might have heard about something called tensors. A tensor is essentially a multidimensional array of values. Of course, that's a simplified explanation, so don't take my word for it without looking it up!

Let's consider an image as an example.

An image consists of pixels. Let's say we have an image with a resolution of 480 × 600 pixels. Each pixel contains three values representing Red, Green, and Blue, or RGB for short.

So, we can represent this image as a 480 × 600 × 3 structure. Imagine those three channels extending in the depth direction. We now have a three-dimensional structure. This is essentially our image tensor.

![An image as a 480 × 600 × 3 tensor with R, G and B channels](/static-files/posts/HDL_Desings/DVConDesignCompetition/image-tensor.svg)
*An RGB image is a 480 × 600 × 3 tensor: three channels stacked in the depth direction.*

Now, let's see what happens when we feed this image into a convolutional neural network (CNN).

The first layer applies convolution operations using filters, often called kernels. These are essentially small matrices that slide across the input image, producing new representations of it.

Let's say we're applying a 3×3 convolution. Since our input image has three channels, the kernel actually has a shape of 3×3×3.

We place this kernel over the top-left corner of the image. Each value in the kernel is multiplied by the corresponding overlapping input value, and all the results are added together to produce a single output value. We then slide the kernel across the image and repeat the process.

If we do this using eight different kernels, we get an output with eight channels. Of course, the output's width and height might also be different, and it won't look like a normal image anymore.

![A 3×3×3 kernel sliding over a 6×6×3 input to produce a 4×4×8 output](/static-files/posts/HDL_Desings/DVConDesignCompetition/convolution.svg)
*Each kernel position gives one output value. Eight kernels give eight output channels.*

There's also something called stride. Instead of moving the kernel one pixel at a time, we can move it by two, three, or more pixels. For example, a stride of two means we skip every other position, producing a smaller output.

![Stride 1 gives a 5×5 output, stride 2 gives a 3×3 output](/static-files/posts/HDL_Desings/DVConDesignCompetition/stride.svg)
*The same 3×3 kernel on a 7×7 input: stride 1 gives a 5×5 output and stride 2 gives 3×3.*

And that's essentially what CNNs do, but across several layers and with different operations in between. These might include activation functions, which are also an important part of neural networks.

Now, why did we spend so much time looking at these operations?

Because eventually, we would have to implement some of them in hardware. Understanding what happens inside a CNN is the first step towards figuring out how to build an accelerator for it.

Anyway, we had our YOLOE model. But there was still another part of the challenge: the prompt.

We combined the YOLOE model with a small language model component that could calculate the similarity between a given prompt and the 80 COCO classes.

I won't go into too much detail about this part, but here's the interesting bit.

The language model converts the given prompt into a vector representation using techniques from language modelling. This vector is called an *embedding*. The idea is that it captures some of the semantic meaning of the prompt.

We can also generate embeddings for all 80 COCO classes and compare them with the prompt embedding. By calculating their similarity, we can identify which classes are closest to the given prompt.

We can then use the selected class or classes to guide YOLOE's object detection process.

![Prompt-based detection pipeline: MiniLM ranks the 80 COCO classes, and YOLOE searches in batches of 10](/static-files/posts/HDL_Desings/DVConDesignCompetition/pipeline.svg)
*Our stage-one pipeline. MiniLM ranks all 80 classes against the prompt, and YOLOE checks them in batches of 10, stopping as soon as it finds something.*

Somehow, we managed to put all of this together for the first stage!

We had a working software pipeline, and YOLOE-26n turned out to be quite lightweight. We even tried running it on a Raspberry Pi just to get an idea of its resource requirements and performance. The initial results looked promising.

Well, mostly promising. Here's one of our test runs:

![Pipeline log for the query "Something to eat": found a match in batch 2 of 8](/static-files/posts/HDL_Desings/DVConDesignCompetition/prompt%201%20terminal.png)
*The log for "Something to eat". A match was found in the second batch, so the remaining 60 classes were skipped (2 re-parameterizations instead of 8).*

![Query "Something to eat" with a low-confidence box labelled bird](/static-files/posts/HDL_Desings/DVConDesignCompetition/prompt1.jpg)
*And the result: a bird, with a score of 0.02. Technically, something might eat it...*

At this point, we had a working model and a reasonably good understanding of what we wanted to build.

Now came the interesting part: figuring out how to move from a working software model to an actual hardware accelerator.