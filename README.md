
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>RyzerDepths – World Prototype</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: #000;
    }
    canvas {
      display: block;
    }
  </style>
</head>
<body>

  <!-- Three.js CDN (stable, non-module) -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>

  <!-- Your game logic -->
  <script src="three.js"></script>

</body>
</html>

└── three.js

 
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

// ----- Camera -----
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  2000
);
camera.position.set(0, 3, 8);

// ----- Renderer -----
const renderer = new THREE.WebGLRenderer({
  antialias: true,
  powerPreference: "high-performance"
});
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

// ----- Lighting -----
const sun = new THREE.DirectionalLight(0xffffff, 1);
sun.position.set(100, 200, 100);
scene.add(sun);

scene.add(new THREE.AmbientLight(0xffffff, 0.4));

// ----- Ground -----
const groundGeo = new THREE.PlaneGeometry(500, 500);
const groundMat = new THREE.MeshStandardMaterial({
  color: 0x2f7d32
});
const ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI / 2;
ground.position.y = 0;
scene.add(ground);

// ----- Test Object -----
const cubeGeo = new THREE.BoxGeometry(1, 1, 1);
const cubeMat = new THREE.MeshStandardMaterial({ color: 0x00ff88 });
const cube = new THREE.Mesh(cubeGeo, cubeMat);
cube.position.y = 0.5;
scene.add(cube);

// ----- Clock -----
const clock = new THREE.Clock();

// ----- Update Loop -----
function update(delta) {
  cube.rotation.y += delta;
}

// ----- Render Loop -----
function animate() {
  requestAnimationFrame(animate);

  const delta = clock.getDelta();
  update(delta);

  renderer.render(scene, camera);
}
animate();

// ----- Resize Handling -----
window.addEventListener("resize", () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
