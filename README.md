# Gesture-Control-Particle
🔍 Project Overview (In Simple Words)

This project is a real-time hand-gesture–controlled 3D particle system built using:

Three.js → 3D graphics & particle rendering

MediaPipe Hands → AI-based hand tracking using webcam

JavaScript + WebGL → Real-time interaction in the browser

Your hand gestures control:

🔹 Particle shape

🔹 Particle size

🔹 Particle color

🔹 Particle rotation

All of this runs live in the browser using a webcam — no backend required.

🧠 Code Explanation (Section by Section)
1️⃣ HTML & CSS – UI Structure
What it does:

Creates a full-screen canvas for 3D particles

Shows project title + your name

Displays a mirrored webcam feed

<canvas> → Three.js renders here
<video id="webcam"> → Live hand tracking

Important CSS:

overflow: hidden → Fullscreen experience

transform: scaleX(-1) → Mirrors webcam (natural interaction)

2️⃣ Importing Libraries
import * as THREE from 'three';
import { HandLandmarker } from "@mediapipe/tasks-vision";

Purpose:

Three.js → Creates 3D particles

MediaPipe HandLandmarker → Detects hand landmarks (21 points per hand)

3️⃣ Global Configuration & State
const PARTICLE_COUNT = 5000;


You render 5000 animated particles (very good performance choice).

State Object:
state = {
  shapeType,
  scale,
  colorHue,
  targetPositions,
  currentPositions
}


This stores:

Current shape

Particle size

Color hue

Smooth morphing positions

👉 This design makes animations smooth and stable.

4️⃣ Three.js Scene Setup
Camera
PerspectiveCamera(75, aspect, 0.1, 1000)


Human-eye-like 3D view

Renderer
WebGLRenderer({ antialias: true })


Smooth edges

GPU-accelerated rendering

5️⃣ Particle System Creation
Geometry
BufferGeometry()


Uses typed arrays (Float32Array) → High performance

Each particle has:

Position (x, y, z)

Color (r, g, b)

Material
PointsMaterial({
  size: 0.05,
  blending: AdditiveBlending
})


✨ Additive blending gives the glowing effect.

6️⃣ Shape Generators (Core Logic)

Function:

updateTargetShape(type)

Supported Shapes:
Fingers	Shape
0–1	Sphere
2	Heart ❤️
3	Saturn 🪐
4+	Flower 🌸

Each shape uses mathematical equations to position particles.

👉 This shows strong math + graphics knowledge.

7️⃣ MediaPipe Hand Tracking
Initialization:
HandLandmarker.createFromOptions()


Uses GPU acceleration

Detects 21 hand landmarks

Runs in VIDEO mode (real-time)

Webcam Access:
navigator.mediaDevices.getUserMedia()

8️⃣ Gesture → Action Mapping (Very Important)
✋ Gesture Controls
🔹 1. Pinch Gesture → Particle Size
distance = thumb_tip ↔ index_tip
scale = distance * 10

🔹 2. Hand X Position → Color
colorHue = hand[0].x


Moves hand left/right → changes color 🌈

🔹 3. Finger Counting → Shape

Uses fingertip vs joint comparison:

if (tip.y < joint.y) finger is open

🔹 4. Hand Movement → Rotation
rotation.x = hand vertical position
rotation.y += constant

9️⃣ Animation Loop
requestAnimationFrame(animate)

Every frame:

Detect hand landmarks

Smoothly morph particles (lerp)

Update color & scale

Render the scene

This creates fluid, cinematic animation.

🔁 Smooth Transitions (Professional Touch)
positions += (target - current) * 0.1


This prevents:
❌ Sudden jumps
✅ Makes transitions look natural

🚀 Why This Project Is Strong

✔ Real-time AI
✔ Computer Vision
✔ 3D Graphics
✔ WebGL Performance
✔ Gesture-based UX
✔ No backend required

Perfect for:

AI Portfolio

Web Graphics

Creative Coding

AR/VR Foundations

🔗 LinkedIn Post (Ready to Upload)

You can copy-paste this directly 👇

🚀 Hand Gesture Controlled 3D Particle System using AI & WebGL ✋✨

I built a real-time hand gesture–controlled 3D particle visualization using Three.js and MediaPipe Hands, where hand movements dynamically control particle shape, size, color, and rotation — all running live in the browser using a webcam.

🔹 Tech Stack
• JavaScript
• Three.js (WebGL)
• MediaPipe Hands (AI Computer Vision)
• HTML & CSS

🔹 Key Features
✔ Real-time hand tracking using AI
✔ Gesture-based shape switching (Sphere, Heart, Saturn, Flower)
✔ Pinch gesture to resize particles
✔ Hand movement controls color and rotation
✔ Smooth particle morphing with high-performance rendering

This project helped me explore the intersection of AI + Creative Coding + 3D Graphics, and it strengthened my understanding of computer vision, real-time interaction, and WebGL optimization.

Looking forward to building more immersive AI-powered experiences 🚀

#ThreeJS #MediaPipe #ComputerVision #AI #WebGL #CreativeCoding #JavaScript #PortfolioProject
