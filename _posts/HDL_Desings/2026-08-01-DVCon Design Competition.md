Design and Verification Conference (DVCon) is a premier event for the design and verification community, bringing together engineers, researchers, and industry experts to share knowledge, best practices, and innovative solutions. The DVCon Design Competition is an exciting opportunity for participants to showcase their skills in digital design and verification through a series of challenging tasks and projects.
This year, we participated in the design challenge, just some 2nd year CS students who just had two modules of digital design and verification under our belts. Despite our limited experience, we were eager to take on the challenge and learn from the process.

The challenge was to create hardware accelerator for given processor to perform a object detection. But not just a normal object detection, it had to do the detection based on a prompt. They would give us an image and a prompt like something to eat, and we would have to find closest thing to food and draw a bounding box around that object.
To limit the scope, they only wanted us to do the task only on 80 coco classes.


It had few several stages. i'll try to go as deep as possible in everything, without making this post boring.

## the first stage.
First we had to lock the method we are going to follow for the process. We had few several ideas, The task itself referenced a research paper using faster RCNN like approaches. 

The thing is eventually we have to implement some part of it in our accelrator. But we didnt really have clue which part, where will be the processor, how the accelrator and processor will communicate.

After finding some more, we came across several types of YOLO models out there. What eventually caought us attention was the nano model, with yolo E and yolo world.
the yoloe-26n version seems really suitable for our task.  its already quantised int 8, support 80 coco classes, and only use 3x3 and 1x1 convolutions and the size was really small comparatively.

you might wonder what is quantization and we also leaned something named pruning. so if we used a normal yolo model intended for every day cpus, the weights are in float32, as a 32 bit floating point number, actually we dont need that much bits to keep the accuracy. Thats where quantization comes in. what it does is use less bits for wieghts, like 8 bits (thats where int 8 comes from). well its not that easy to do actually, you just get rid of other 24 bits and call it a day. there are ways to do quantization, which are quantization aware training. and luckily, there is yoloe already quantized version.

then there is a thing called pruning i came across, thats where we remove unwated channels altogether. whats a channel? 
when talking about neural networks you might have heard about tensors. Tensor is a some multi dimentional vector. ofcourse these are not the most accurate descriptions so better have a look before believing everything i say. but its not just a vecotr, let's consider how an image is a tensor.
the image have pixels, lets say there are 480x600 pixels in the image. each pixel contains 3 channels namely Red,Green, Blue or RGB in short. what we actually have is a 480x600x3 like structure. Now imagine those 3 are in depth direction, so you have a 3D structure. Actually this is our image tensor. and for our yolo model this is the actual tensor for the first layer.

The first layer apply some kernel aperations, those are the that 3x3 matrices like things sliding thorugh the image creating some changed versions of same image. lets say we are applying a 3x3 kernel, one way to do it is, have kernel that have 3x3x3 shape, and multiply each value that overlap with the kernel when we place it on the left top corner and add them all toghether and put it in a new image and repeat the procedeur by sliding it across the image. if we do the same thing for 8 kernels, we will get a new image with 8 channels. of course the image width and height will be little different and it wont look like a natural image anymore. and while we are doing it we can skip evey second row (or in general do it once n rows) and get a somewhat smaller output image. that skipping paramter is called stried.

what cnn are doing is almost that same thing, but over few several steps. well in between those steps you might do some other things like activation, which is also key part of neural networs.

Anyway, we got out yoloe model, and we combined it with a small language model that can check the similarity between the prompt and the 80 coco classes. i wont geet to too much details about that, but what cool about it is,it get a vector from prompt, using some language models related thing called attention. that vector is called a embedding, because that supposed to contain the meaning like thing of the prompt. what we can do is calculate similarly the vectors for 80 coco classes and and compare those embeddings, the ones which are closest, to the prmpt embedding gives us the answer. so we can give that class to the yoloe to search for.

somehow for the first stage we managed to fix this up. the pipeline works, the yoloe-26n is very light weight. we even tried running it in a raspberry pi just to see how it uses resources, and it was showing some good results.