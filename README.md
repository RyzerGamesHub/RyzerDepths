// @ts-nocheck
// src/worker.js
export default {
  async fetch(request) {
    const url = new URL(request.url);

    // Serve HTML
    if (url.pathname === "/") {
      return new Response(INDEX_HTML, {
        headers: { "Content-Type": "text/html" }
      });
    }

    // Serve Three.js game code
    if (url.pathname === "/main.js") {
      return new Response(MAIN_JS, {
        headers: { "Content-Type": "application/javascript" }
      });
    }

    return new Response("Not found", { status: 404 });
  }
};

// ------------------------
// HTML (browser entry)
// ------------------------
const INDEX_HTML = `
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Prototype</title>
  <style>
    body { margin: 0; overflow: hidden; background: #000; }
  </style>
</head>
<body>
  <script type="module" src="/main.js"></script>
</body>
</html>
`;

const MAIN_JS = `
import * as THREE from "https://unpkg.com/three@0.161.0/build/three.module.js";

// --------------------
// Scene
// --------------------
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87CEEB);

// --------------------
// Camera hierarchy
// --------------------
const cameraHolder = new THREE.Object3D(); // yaw + position
const cameraPivot = new THREE.Object3D();  // pitch only
scene.add(cameraHolder);
cameraHolder.add(cameraPivot);

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.y = 0.2;
cameraPivot.add(camera);

// --------------------
// Renderer
// --------------------
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// --------------------
// Clock
// --------------------
const clock = new THREE.Clock();

// Player
const player = {
  position: new THREE.Vector3(0, 1.7, 5),
  velocity: new THREE.Vector3(),
  speed: 5,
  gravity: -30,
  jumpForce: 10,
  height: 1.7,
  onGround: false
};

// Visible body (optional but useful)
const bodyMesh = new THREE.Mesh(
  new THREE.CapsuleGeometry(0.3, 1.0, 4, 8),
  new THREE.MeshStandardMaterial({ color: 0x44aa88 })
);
scene.add(bodyMesh);

// Ground
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(1000, 1000),
  new THREE.MeshStandardMaterial({ color: 0x4CAF50 })
);

ground.rotation.x = -Math.PI / 2;
scene.add(ground);

// Light
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(10, 20, 10);
scene.add(light);

// Input
const keys = {};
window.addEventListener("keydown", e => keys[e.code] = true);
window.addEventListener("keyup", e => keys[e.code] = false);

document.body.addEventListener("click", () => {
  document.body.requestPointerLock();
});

let yaw = 0;
let pitch = 0;

window.addEventListener("mousemove", e => {
  if (document.pointerLockElement !== document.body) return;

  yaw   -= e.movementX * 0.002;
  pitch -= e.movementY * 0.002;

  pitch = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, pitch));

  cameraHolder.rotation.y = yaw;   // left / right
  cameraPivot.rotation.x = pitch;  // up / down
});

// Resize
window.addEventListener("resize", () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});


// Animate
function animate() {
  requestAnimationFrame(animate);
  const delta = clock.getDelta();

  // Movement direction
  const dir = new THREE.Vector3();
  if (keys["KeyW"]) dir.z -= 1;
  if (keys["KeyS"]) dir.z += 1;
  if (keys["KeyA"]) dir.x -= 1;
  if (keys["KeyD"]) dir.x += 1;

  if (dir.length() > 0) {
    dir.normalize();
    dir.applyAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
    player.velocity.x = dir.x * player.speed;
    player.velocity.z = dir.z * player.speed;
  } else {
    player.velocity.x = 0;
    player.velocity.z = 0;
  }

  // Jump
  if (keys["Space"] && player.onGround) {
    player.velocity.y = player.jumpForce;
    player.onGround = false;
  }

  // Gravity
  player.velocity.y += player.gravity * delta;

  // Apply movement
  player.position.addScaledVector(player.velocity, delta);

  // Ground collision
  if (player.position.y < player.height) {
    player.position.y = player.height;
    player.velocity.y = 0;
    player.onGround = true;
  }

  // Sync visuals
  bodyMesh.position.copy(player.position);
  bodyMesh.rotation.y = yaw;

  cameraHolder.position.copy(player.position);

  renderer.render(scene, camera);
}

animate(); `;
