# Creating a animated Augmented reality experience with ARjs.

ARjs is a great tool to create augmented reality experence without the need of installing additional applications you can create or experience AR in your browser.

It has several ways to display the 3D model namrly,
    1. marker traking
    2. Image tracking
    3. Location tracking

but it lacks SLAM(Simultaneous Localization and Mapping), which is also a very handy method of showing AR models.

## My goals
In my project I tried to create a animation with blender and export it as a .glb file to display along with a marker. The best thing about using a marker is that, ARjs handle the change of oietation or distance to marker which is crucial for better AR experiance. And as a cherry on top I am plannig to play a video in the animation.

Eventhough my goals are simple enough along the journey I faced several challenges which inspired me to write this post.

### Playing around

I first played around with the biult in functions of ARjs like the following code. If you want you can create your own marker and try this yourself.

```
<!DOCTYPE html>
<html>
<head>
    <script src="https://aframe.io/releases/1.2.0/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/gh/jeromeetienne/ar.js@2.2.1/aframe/build/aframe-ar.min.js"></script>
</head>
<body style="margin: 0; overflow: hidden;">
    <a-scene embedded arjs="debugUIEnabled: false;" vr-mode-ui="enabled: false">
        <a-marker id="marker" preset="custom" type="pattern"
            url="/path/to/your/pattern.patt" smooth="true" smoothCount="10" smoothTolerance="0.01" smoothThreshold="2">
            <a-box id="model" position="5 0.5 0" material="color: red;" data-raycastable></a-box>
        </a-marker>
        <a-entity camera></a-entity>
    </a-scene>
</body>
</html>

```

Even further I tried the the NFT image trakcer and few events other things lised in the documentation. I even tried to keep the 3D model on place even after the marker lost from the sight. Eventhough it is possible to avoid 3D object from disappearing whne the marker is lost, we cannot precisely keep it stationary relative to the real environment only with events like "devicemotion", and "deviceorientaion". This where SLAM comes into play which harder to implement all by my self within in a week.

So decided go with the marker based tracking.

### Disaster happens

I spent my most of the time creating and planing the the 3D animation. (I am not a 3D artist, so this part is by default hard for me). With the help of blender and few friends I was able to create a 3D animation that worked nicely. So all I have to do is export it as a .glb file and use it in my project.

So I thought.. But appearently the .glb file animations are not so supported by ARjs. The object is there but no animation. I asked my friend who helped me with the blender and 3D objects, the animation has lost during the process of exporting. But he somehow opened it in a online 3D object viewer. And the animation is there. That's when I learnt that ARjs does not support animations fully. There is only few hours to my project deadline. Is this the end? I wondered.

I noticed that the online preview site was based on threejs. So threejs support the animation I thought. I opened the file in browser with threejs. yes.. the animations works well. 

### Desperate attempts
why don't I use a dummy object with the marker and just open the animation with threejs and copy the dummy objects position and rotation to the aniamtion. 
Yes I did that. And it worked like a charm.


okay now everthing is working. only few adjustments left. Ah I forgot, I haven't included the video to the animation. I asked my friend to add that and took a break. 

### Disater 2
unfortunately when I export the animation, it appears to left out the video. upon further reasearch I got to know that I cannot export to .glb with a video texture.

Only hours left,panicked again! I remebered that I can programatically asign a texture to the plane playing the video with threejs. and I did that. In my preview, in my laptop it is working. Calm.

okay I am ready to ship. I let others try the marker. everthing works well but not the the video texture. But it still works on my laptop. Unfortunately the video textures are not supported for mobile phones. Panicked again.

Out of my desperate attempts, I am running out of time and sleep deprived. I had to stop. So I couldn't deliver the project.

Later I remembered that I could display the video seperately as a video and try to place it on the plane. I tried that, but it still far from perfect.