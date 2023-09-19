<template>
  <div>
    <div class="wrapper">
      <div ref="containerRef" class="container" />
    </div>
    <div v-for="(content, index) in contents" :key="content" class="message">
      <transition name="fade">
        <div v-show="openBox === index + 1" class="test">
          {{ content.name }}
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import gsap from "gsap";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

const contents = [
  { name: "포트폴리오" },
  { name: "갤러리" },
  { name: "깃허브" },
  { name: "코드팬" },
];

const containerRef = ref();
const openBox = ref(0);

let renderer;
let camera;
let raf;
let mixer;
let fish;
let chest1, chest2, chest3, chest4;

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x555555);
const loader = new GLTFLoader();

const floorGeometry = new THREE.CircleGeometry(22, 32);
const floorMaterial = new THREE.MeshBasicMaterial({ color: 0x0000ff });
const floor = new THREE.Mesh(floorGeometry, floorMaterial);
floor.rotation.set(-Math.PI / 2, 0, 0);
floor.position.set(0, -1, 0);
scene.add(floor);

// light
const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
scene.add(ambientLight);
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(1, 1, 1);
scene.add(light);

const loadChest = (x, z) => {
  return new Promise((resolve) => {
    loader.load("/chest.glb", (gltf) => {
      const chest = gltf;
      if (gltf.animations && gltf.animations.length > 0) {
        chest.mixer = new THREE.AnimationMixer(chest.scene);
        gltf.animations.forEach((clip) => {
          chest.mixer.clipAction(clip).play();
        });
      }

      chest.scene.scale.set(0.02, 0.02, 0.02);
      chest.scene.rotation.y = Math.PI;
      chest.scene.position.set(x, 0, z);
      scene.add(chest.scene);
      resolve(chest);
    });
  });
};

const init = () => {
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(
    containerRef.value.offsetWidth,
    containerRef.value.offsetHeight,
  );
  containerRef.value.appendChild(renderer.domElement);
};

const animate = () => {
  camera.updateMatrixWorld();
  renderer.render(scene, camera);
  raf = requestAnimationFrame(animate);

  if (mixer) mixer.update(0.01);

  if (fish) {
    const targetPosition = fish.position.clone().add(offset);
    camera.position.copy(targetPosition);
    camera.lookAt(fish.position);
  }
};

const cameraY = 13; // obj로부터의 카메라 높이
const cameraZ = 13; // obj로부터의 카메라 거리
const offset = new THREE.Vector3(0, cameraY, -cameraZ);

const onResize = () => {
  camera.aspect =
    containerRef.value.offsetWidth / containerRef.value.offsetHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(
    containerRef.value.offsetWidth,
    containerRef.value.offsetHeight,
  );
};

const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let box1AnimationPlayed = false;
let box2AnimationPlayed = false;
let box3AnimationPlayed = false;
let box4AnimationPlayed = false;
const onClick = (e) => {
  mouse.x = (e.clientX / containerRef.value.offsetWidth) * 2 - 1;
  mouse.y = -(e.clientY / containerRef.value.offsetHeight) * 2 + 1;

  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(floor);

  if (intersects.length > 0) {
    const intersectionPoint = intersects[0].point;
    console.log("클릭 좌표:", intersectionPoint);

    const distance = fish.position.distanceTo(intersectionPoint);
    const speed = 15;
    const duration = distance / speed;

    gsap.killTweensOf(fish.position);
    gsap.to(fish.position, {
      x: intersectionPoint.x,
      z: intersectionPoint.z,
      duration,
      onUpdate: () => {
        const box1distance = fish.position.distanceTo(chest1.scene.position);
        const box2distance = fish.position.distanceTo(chest2.scene.position);
        const box3distance = fish.position.distanceTo(chest3.scene.position);
        const box4distance = fish.position.distanceTo(chest4.scene.position);

        if (box1distance <= 5 && !box1AnimationPlayed) {
          mixer = new THREE.AnimationMixer(chest1.scene);
          const action = mixer.clipAction(chest1.animations[1]);
          action.setLoop(THREE.LoopOnce);
          action.clampWhenFinished = true;
          action.play();
          openBox.value = 1;
          box1AnimationPlayed = true;
        } else if (box1distance > 5 && box1AnimationPlayed) {
          mixer = new THREE.AnimationMixer(chest1.scene);
          const action = mixer.clipAction(chest1.animations[3]);
          action.setLoop(THREE.LoopOnce);
          action.clampWhenFinished = true;
          action.play();
          openBox.value = 0;
          box1AnimationPlayed = false;
        }

        if (box2distance <= 5 && !box2AnimationPlayed) {
          mixer = new THREE.AnimationMixer(chest2.scene);
          const action = mixer.clipAction(chest2.animations[1]);
          action.setLoop(THREE.LoopOnce);
          action.clampWhenFinished = true;
          action.play();
          openBox.value = 2;
          box2AnimationPlayed = true;
        } else if (box2distance > 5 && box2AnimationPlayed) {
          mixer = new THREE.AnimationMixer(chest2.scene);
          const action = mixer.clipAction(chest2.animations[3]);
          action.setLoop(THREE.LoopOnce);
          action.clampWhenFinished = true;
          action.play();
          openBox.value = 0;
          box2AnimationPlayed = false;
        }

        if (box3distance <= 5 && !box3AnimationPlayed) {
          mixer = new THREE.AnimationMixer(chest3.scene);
          const action = mixer.clipAction(chest3.animations[1]);
          action.setLoop(THREE.LoopOnce);
          action.clampWhenFinished = true;
          action.play();
          openBox.value = 3;
          box3AnimationPlayed = true;
        } else if (box3distance > 5 && box3AnimationPlayed) {
          mixer = new THREE.AnimationMixer(chest3.scene);
          const action = mixer.clipAction(chest3.animations[3]);
          action.setLoop(THREE.LoopOnce);
          action.clampWhenFinished = true;
          action.play();
          openBox.value = 0;
          box3AnimationPlayed = false;
        }

        if (box4distance <= 5 && !box4AnimationPlayed) {
          mixer = new THREE.AnimationMixer(chest4.scene);
          const action = mixer.clipAction(chest4.animations[1]);
          action.setLoop(THREE.LoopOnce);
          action.clampWhenFinished = true;
          action.play();
          openBox.value = 4;
          box4AnimationPlayed = true;
        } else if (box4distance > 5 && box4AnimationPlayed) {
          mixer = new THREE.AnimationMixer(chest4.scene);
          const action = mixer.clipAction(chest4.animations[3]);
          action.setLoop(THREE.LoopOnce);
          action.clampWhenFinished = true;
          action.play();
          openBox.value = 0;
          box4AnimationPlayed = false;
        }
      },
    });

    const lookAtPoint = new THREE.Vector3(
      intersectionPoint.x,
      fish.position.y,
      intersectionPoint.z,
    );
    fish.lookAt(lookAtPoint);
  }
};

onMounted(async () => {
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });

  loader.load("/fish.glb", (gltf) => {
    fish = gltf.scene;
    scene.add(fish);
  });

  [chest1, chest2, chest3, chest4] = await Promise.all([
    loadChest(15, 15),
    loadChest(-15, 15),
    loadChest(15, -15),
    loadChest(-15, -15),
  ]);

  camera = new THREE.PerspectiveCamera(
    80,
    containerRef.value.offsetWidth / containerRef.value.offsetHeight,
    0.1,
    1000,
  );
  // camera.position.set(0, 2, 3);
  // camera.lookAt(0, 0, 0);

  init();
  animate();

  window.addEventListener("resize", onResize);
  containerRef.value.addEventListener("click", onClick);
});

onBeforeUnmount(() => {
  cancelAnimationFrame(raf);
  renderer.dispose();

  window.removeEventListener("resize", onResize);
});
</script>

<style lang="scss" scoped>
.wrapper {
  width: 100%;
  height: 100vh;
  .container {
    width: 100%;
    height: 100%;
  }
}
.message {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 60%;
  height: 60%;
  pointer-events: none;
  z-index: 3;
  .test {
    width: 100%;
    height: 100%;
    padding: 20px;
    font-size: 2em;
    text-align: center;
    color: white;
    background: rgb(175, 175, 175);
    transform-origin: center;
    opacity: 0.2;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: 0.7s ease-in-out;
}
.fade-enter-from,
.fade-leave-to {
  // opacity: 0;
  transform: scaleY(0);
}
</style>
