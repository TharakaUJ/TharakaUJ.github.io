Augmented Reality (AR) is transforming how we interact with digital content, making it more immersive and interactive. AR.js, a lightweight open-source library, allows developers to create AR experiences directly in the browser, eliminating the need for additional app installations. In this post, I’ll share my journey of creating an animated AR experience for my university’s inauguration display—a project that, despite its challenges, taught me a lot about AR.js and browser-based AR development.

---

## Why AR.js?

AR.js is a fantastic tool for creating AR experiences due to its versatility and accessibility. It supports multiple tracking methods:

1. **Marker tracking**: Uses predefined markers to anchor AR content.
2. **Image tracking**: Tracks and displays AR content on specific images.
3. **Location tracking**: Anchors AR elements based on GPS data.

However, AR.js does not support SLAM (Simultaneous Localization and Mapping), which is crucial for keeping AR objects stationary relative to the real-world environment when markers are no longer visible. This limitation posed a significant challenge during my project.

---

## My Project Goals

The project aimed to:
1. Create a 3D animation using Blender and export it as a `.glb` file.
2. Display the 3D animation anchored to a marker using AR.js.
3. Play a video texture within the animation for added interactivity.

While these goals seemed straightforward, the journey was filled with obstacles and learning moments that inspired me to document the experience.

---

## Exploring AR.js

### Code Snippet: Marker-Based Tracking
Here's a simple code snippet to get you started with AR.js, similar to what I tried when I first learned about AR.js. This example demonstrates marker-based tracking, where AR content is anchored to a predefined marker. You can create your own marker pattern using the [AR.js Marker Training Tool](https://jeromeetienne.github.io/AR.js/three.js/examples/marker-training/examples/generator.html).

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
---

## Challenges and Setbacks

### Animation Support in AR.js

After creating the 3D animation in Blender, I exported it as a `.glb` file, assuming it would work seamlessly in AR.js. However, AR.js does not fully support animations within `.glb` files. The object appeared, but the animation was missing.

### Finding a Workaround

I used a dummy object as the marker anchor in AR.js and synchronized its position and rotation with the animation rendered via Three.js. Below is the code:

#### Code Snippet: AR.js and Three.js Integration

```html
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

Blender couldn’t export `.glb` files with embedded video textures. I programmatically assigned the video as a texture to a plane in Three.js. While this worked on desktop, it failed on mobile devices due to browser limitations.

### Final Hours

With hours left before the deadline, I considered alternative approaches like dynamically displaying the video separately. However, the results were far from perfect, and I had to abandon the project due to time constraints.

![Final Project Screenshot]({{ site.baseUrl }}/assets/images/arjs-post-2025-01-14-01.png)

---

## Lessons Learned

- **Plan for limitations**: Understand the capabilities and constraints of the tools you’re using.
- **Iterate quickly**: Test each step early to avoid last-minute surprises.
- **Leverage the community**: Resources like documentation, forums, and peers can save valuable time.
- **Keep it simple**: Sometimes, simpler solutions yield better results, especially under tight deadlines.


---

## Conclusion

Although I couldn’t deliver the project in time, the experience was invaluable. It deepened my understanding of AR.js, Three.js, and browser-based AR. For anyone venturing into AR development, I encourage experimenting with these tools, understanding their limitations, and sharing your experiences.

With more time and preparation, I hope to revisit this project and bring it to life. Stay tuned!

---

## Versions Used

| Tool      | Version |
|-----------|---------|
| Blender   | 4.3.2   |
| AR.js     | 3.3.2   |
| A-Frame   | 1.2.0   |
| Three.js  | 0.172.0 |

---

### Demo and Resources

- **GitHub Repository**: [AR.js Animation Project](https://github.com/TharakaUJ/AR-inogaration)
- **Live Demo**: [GitHub Pages](https://tharakauj.github.io/AR-inogaration/)

