# Creating an Animated Augmented Reality Experience with AR.js

Augmented Reality (AR) is transforming how we interact with digital content, making it more immersive and interactive. AR.js is a lightweight, open-source library that allows developers to create AR experiences directly in the browser, eliminating the need for additional app installations. In this post, I’ll share my journey of trying to create an animated AR experience for my university’s inauguration display—a project that, despite its challenges, taught me a lot about AR.js and browser-based AR development.

## Why AR.js?
AR.js is a fantastic tool for creating AR experiences due to its versatility and accessibility. It supports multiple tracking methods:

1. **Marker tracking**: Uses predefined markers to anchor AR content.
2. **Image tracking**: Tracks and displays AR content on specific images.
3. **Location tracking**: Anchors AR elements based on GPS data.

However, AR.js does not support SLAM (Simultaneous Localization and Mapping), which is crucial for keeping AR objects stationary relative to the real-world environment when markers are no longer visible. This limitation posed a significant challenge during my project.

## My Project Goals
The project aimed to:
1. Create a 3D animation using Blender and export it as a `.glb` file.
2. Display the 3D animation anchored to a marker using AR.js.
3. Play a video texture within the animation for added interactivity.

While these goals seemed straightforward, the journey was filled with obstacles and learning moments that inspired me to document the experience.

## Exploring AR.js
I began by experimenting with the built-in functionalities of AR.js. Below is a simple example of marker-based tracking:

```html
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

I also explored other features, such as the NFT image tracker and handling events like `devicemotion` and `deviceorientation`. While these were helpful, they didn’t fully solve the challenge of keeping objects stationary relative to the environment when the marker was out of sight—a problem that SLAM would have addressed.

## Challenges and Setbacks
### Animation Support in AR.js
After creating the 3D animation in Blender, I exported it as a `.glb` file, assuming it would work seamlessly in AR.js. However, AR.js does not fully support animations within `.glb` files. The object appeared, but the animation was missing. After consulting a friend, we confirmed that the animation data was intact, as it worked in online viewers built with Three.js.

### Finding a Workaround
Realizing that Three.js supported animations, I devised a workaround: use a dummy object as the marker anchor in AR.js and synchronize its position and rotation with the animation rendered via Three.js. This approach worked beautifully, but time was running out.

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AR.js with Three.js</title>
    <script src="https://aframe.io/releases/1.2.0/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/gh/AR-js-org/AR.js@3.3.2/aframe/build/aframe-ar.js"></script>
    <script type="importmap">
        {
          "imports": {
            "three": "https://cdn.jsdelivr.net/npm/three@latest/build/three.module.js"
          }
        }
    </script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
        }
    </style>
</head>

<body>
    <!-- AR.js Scene -->
    <a-scene embedded arjs>
        <!-- Marker -->
        <a-marker id="marker" preset="custom" type="pattern" url="/assets/ar/pattern-Untitled.patt" smooth="true"
            smoothCount="0" smoothTolerance="0.0005" smoothThreshold="1.5" preload="true">
            <!-- Pseudo-object for positioning -->
            <a-entity id="pseudo-object" geometry="primitive: box; height: 0.1; width: 0.1; depth: 0.1"
                material="color: red" position="0 0.05 0"></a-entity>
        </a-marker>

        <!-- Camera -->
        <a-entity camera></a-entity>
    </a-scene>

    <script type="module">
        import * as THREE from 'three';
        import { GLTFLoader } from 'https://cdn.jsdelivr.net/npm/three@0.152.2/examples/jsm/loaders/GLTFLoader.js';

        // AR.js and Three.js Integration
        window.addEventListener('DOMContentLoaded', () => {
            console.log('AR.js and Three.js are ready.');
            const pseudoObject = document.querySelector('#pseudo-object');

            // Set up Three.js scene
            const scene = pseudoObject.object3D;
            const loader = new GLTFLoader();

            let mixer; // For animations
            let model; // To store the loaded model

            // Load the GLB model
            loader.load('/path/to/glb/file.glb', (gltf) => {
                model = gltf.scene;
                model.position.set(0, 0.2, 0); // Adjust relative position
                model.scale.set(0.5, 0.5, 0.5); // Adjust scale if needed
                scene.add(model);

                // Animation mixer setup
                if (gltf.animations && gltf.animations.length) {
                    mixer = new THREE.AnimationMixer(model);
                    gltf.animations.forEach((clip) => {
                        const action = mixer.clipAction(clip);
                        action.play();
                    });
                }
            }, undefined, (error) => {
                console.error('An error occurred while loading the model:', error);
            });

            // Animation loop
            const clock = new THREE.Clock();
            function animate() {
                requestAnimationFrame(animate);

                // Update the animation mixer
                if (mixer) {
                    const delta = clock.getDelta();
                    mixer.update(delta);
                }

                // Update model position based on marker position
                if (model) {
                    const pseudoObject = document.querySelector('#pseudo-object');
                    const pseudoPosition = pseudoObject.object3D.position;
                    const pseudoRotation = pseudoObject.object3D.rotation;

                    model.position.copy(pseudoPosition);
                    model.rotation.copy(pseudoRotation);
                }
            }

            animate();
        });
    </script>
</body>

</html>
```

### Video Texture Limitations
Adding a video texture to the animation introduced another hurdle. Blender couldn’t export `.glb` files with embedded video textures. To resolve this, I programmatically assigned the video as a texture to a plane in Three.js. While this worked on my laptop, the solution failed on mobile devices due to browser limitations in handling video textures in WebGL.

### Working Example
For a working example of using a video as a texture in AR.js, check out this [demo](https://tharakauj.github.io/AR-inogaration/video-as-texture/). you can find the marker [here](https://tharakauj.github.io/AR-inogaration/).

## Final Hours
With only hours left before the deadline, I considered alternative approaches, such as [displaying the video separately] (https://tharakauj.github.io/AR-inogaration/pattern-model/) and positioning it dynamically. However, the results were far from perfect, and I had to abandon the project due to time constraints.

## Lessons Learned
1. **Plan for limitations**: Understand the capabilities and constraints of the tools you’re using before starting.
2. **Iterate quickly**: Test each step early to avoid last-minute surprises.
3. **Leverage the community**: Resources like documentation, forums, and peers can save valuable time.
4. **Keep it simple**: Sometimes, simpler solutions yield more reliable results, especially under tight deadlines.

## Conclusion
Although I couldn’t deliver the project in time, the experience was invaluable. It deepened my understanding of AR.js, Three.js, and the intricacies of browser-based AR. For anyone venturing into AR development, I encourage experimenting with these tools, understanding their limitations, and sharing your experiences—just as I’ve done here.

AR is a fascinating field, and browser-based AR offers immense potential for accessibility and creativity. With more time and preparation, I hope to revisit this project and bring it to life. Stay tuned!

