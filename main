<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gesture Controlled Particles</title>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: sans-serif; }
        #info {
            position: absolute; top: 20px; left: 20px; color: white;
            pointer-events: none; text-shadow: 1px 1px 2px black;
        }
        canvas { display: block; }
        #video-container {
            position: absolute; bottom: 20px; right: 20px;
            width: 200px; height: 150px; border: 2px solid #444; border-radius: 8px;
            transform: scaleX(-1); /* Mirror the webcam */
        }
        video { width: 100%; height: 100%; object-fit: cover; }
    </style>
</head>
<body>

    <div id="info">
        <h1>Hand Gesture Particles</h1>
	<h1>Omkar Sanas</h1>
        <p>1  | 2  | 3  | 4 </p>
        <p>: Resize Particles</p>
    </div>

    <div id="video-container">
        <video id="webcam" autoplay playsinline></video>
    </div>

    <!-- Import Maps for Three.js and MediaPipe -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { HandLandmarker, FilesetResolver } from "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.0";

        // --- Configuration ---
        const PARTICLE_COUNT = 5000;
        let handLandmarker;
        let webcamRunning = false;
        let lastVideoTime = -1;
        
        // State
        const state = {
            shapeType: 'sphere',
            scale: 1,
            colorHue: 0.5,
            targetPositions: new Float32Array(PARTICLE_COUNT * 3),
            currentPositions: new Float32Array(PARTICLE_COUNT * 3)
        };

        // --- Three.js Setup ---
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.z = 5;

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);

        // Particle Geometry
        const geometry = new THREE.BufferGeometry();
        const positions = new Float32Array(PARTICLE_COUNT * 3);
        const colors = new Float32Array(PARTICLE_COUNT * 3);

        for (let i = 0; i < PARTICLE_COUNT; i++) {
            positions[i * 3] = (Math.random() - 0.5) * 10;
            positions[i * 3 + 1] = (Math.random() - 0.5) * 10;
            positions[i * 3 + 2] = (Math.random() - 0.5) * 10;
        }

        geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
        geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

        const material = new THREE.PointsMaterial({
            size: 0.05,
            vertexColors: true,
            transparent: true,
            blending: THREE.AdditiveBlending,
            depthWrite: false
        });

        const particleSystem = new THREE.Points(geometry, material);
        scene.add(particleSystem);

        // --- Shape Generators ---
        function updateTargetShape(type) {
            const pos = state.targetPositions;
            for (let i = 0; i < PARTICLE_COUNT; i++) {
                const i3 = i * 3;
                if (type === 'heart') {
                    const t = Math.random() * Math.PI * 2;
                    const x = 16 * Math.pow(Math.sin(t), 3);
                    const y = 13 * Math.cos(t) - 5 * Math.cos(2 * t) - 2 * Math.cos(3 * t) - Math.cos(4 * t);
                    pos[i3] = x * 0.1;
                    pos[i3 + 1] = y * 0.1;
                    pos[i3 + 2] = (Math.random() - 0.5) * 0.5;
                } else if (type === 'saturn') {
                    if (i < PARTICLE_COUNT * 0.6) { // Planet
                        const phi = Math.acos(-1 + (2 * i) / (PARTICLE_COUNT * 0.6));
                        const theta = Math.sqrt(PARTICLE_COUNT * 0.6 * Math.PI) * phi;
                        pos[i3] = Math.cos(theta) * Math.sin(phi) * 1.5;
                        pos[i3 + 1] = Math.sin(theta) * Math.sin(phi) * 1.5;
                        pos[i3 + 2] = Math.cos(phi) * 1.5;
                    } else { // Rings
                        const angle = Math.random() * Math.PI * 2;
                        const radius = 2.2 + Math.random() * 0.8;
                        pos[i3] = Math.cos(angle) * radius;
                        pos[i3 + 1] = Math.sin(angle) * radius * 0.4; // Tilted
                        pos[i3 + 2] = Math.sin(angle) * radius * 0.2;
                    }
                } else if (type === 'flower') {
                    const t = Math.random() * Math.PI * 2;
                    const r = 2 * Math.cos(5 * t);
                    pos[i3] = r * Math.cos(t);
                    pos[i3 + 1] = r * Math.sin(t);
                    pos[i3 + 2] = (Math.random() - 0.5) * 0.5;
                } else { // Sphere
                    const phi = Math.acos(-1 + (2 * i) / PARTICLE_COUNT);
                    const theta = Math.sqrt(PARTICLE_COUNT * Math.PI) * phi;
                    pos[i3] = Math.cos(theta) * Math.sin(phi) * 2;
                    pos[i3 + 1] = Math.sin(theta) * Math.sin(phi) * 2;
                    pos[i3 + 2] = Math.cos(phi) * 2;
                }
            }
        }

        // --- MediaPipe Hand Tracking Setup ---
        async function initHandTracking() {
            const vision = await FilesetResolver.forVisionTasks("https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.0/wasm");
            handLandmarker = await HandLandmarker.createFromOptions(vision, {
                baseOptions: { modelAssetPath: `https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task`, delegate: "GPU" },
                runningMode: "VIDEO",
                numHands: 1
            });
            startWebcam();
        }

        async function startWebcam() {
            const video = document.getElementById("webcam");
            const constraints = { video: true };
            const stream = await navigator.mediaDevices.getUserMedia(constraints);
            video.srcObject = stream;
            video.addEventListener("loadeddata", () => { webcamRunning = true; });
        }

        // --- Logic: Gesture to Action ---
        function processHands(results) {
            if (results.landmarks && results.landmarks.length > 0) {
                const hand = results.landmarks[0];
                
                // 1. Expansion via Pinch (Thumb tip to Index tip)
                const distance = Math.hypot(hand[4].x - hand[8].x, hand[4].y - hand[8].y);
                state.scale = THREE.MathUtils.lerp(state.scale, distance * 10, 0.1);

                // 2. Color via Hand Position (X coordinate)
                state.colorHue = hand[0].x;

                // 3. Shape via Finger Counting
                // Simple logic: if finger tip is higher than the joint below it
                let count = 0;
                if (hand[8].y < hand[6].y) count++;  // Index
                if (hand[12].y < hand[10].y) count++; // Middle
                if (hand[16].y < hand[14].y) count++; // Ring
                if (hand[20].y < hand[18].y) count++; // Pinky

                const lastShape = state.shapeType;
                if (count === 0 || count === 1) state.shapeType = 'sphere';
                else if (count === 2) state.shapeType = 'heart';
                else if (count === 3) state.shapeType = 'saturn';
                else if (count >= 4) state.shapeType = 'flower';

                if (lastShape !== state.shapeType) updateTargetShape(state.shapeType);

                // 4. Rotation
                particleSystem.rotation.y += 0.01;
                particleSystem.rotation.x = (hand[0].y - 0.5) * 2;
            }
        }

        // --- Animation Loop ---
        function animate() {
            if (webcamRunning) {
                const video = document.getElementById("webcam");
                if (video.currentTime !== lastVideoTime) {
                    const results = handLandmarker.detectForVideo(video, performance.now());
                    processHands(results);
                    lastVideoTime = video.currentTime;
                }
            }

            // Morph particles toward target
            const positions = geometry.attributes.position.array;
            const colors = geometry.attributes.color.array;
            const colorObj = new THREE.Color().setHSL(state.colorHue, 0.8, 0.5);

            for (let i = 0; i < PARTICLE_COUNT; i++) {
                const i3 = i * 3;
                // Smooth interpolation (lerp)
                positions[i3] += (state.targetPositions[i3] * state.scale - positions[i3]) * 0.1;
                positions[i3 + 1] += (state.targetPositions[i3 + 1] * state.scale - positions[i3 + 1]) * 0.1;
                positions[i3 + 2] += (state.targetPositions[i3 + 2] * state.scale - positions[i3 + 2]) * 0.1;

                // Update colors
                colors[i3] = colorObj.r;
                colors[i3 + 1] = colorObj.g;
                colors[i3 + 2] = colorObj.b;
            }
            
            geometry.attributes.position.needsUpdate = true;
            geometry.attributes.color.needsUpdate = true;

            renderer.render(scene, camera);
            requestAnimationFrame(animate);
        }

        // Handle Resize
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        // Initialize
        updateTargetShape('sphere');
        initHandTracking();
        animate();

    </script>
</body>
</html>
