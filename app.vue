<template>
  <div>
    <div class="container">
      <div class="button-wrapper">
        <div id="w" ref="wRef" @mousedown="onMouseDown('w')">
          <p>W</p>
        </div>
        <div id="a" ref="aRef" @mousedown="onMouseDown('a')">
          <p>A</p>
        </div>
        <div id="s" ref="sRef" @mousedown="onMouseDown('s')">
          <p>S</p>
        </div>
        <div id="d" ref="dRef" @mousedown="onMouseDown('d')">
          <p>D</p>
        </div>
      </div>
      <div ref="canvasRef" class="wrapper" />
    </div>
    <div v-for="(content, index) in contents" :key="content" class="message">
      <transition name="fade">
        <div v-show="openBox === index + 1" class="content-wrapper">
          <p>{{ content.name }}</p>
          <button @click="moveToPage(content.route)">보러가기</button>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import gsap from "gsap";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

const wRef = ref();
const aRef = ref();
const sRef = ref();
const dRef = ref();

const contents = [
  { name: "포트폴리오", route: "newsungpf.firebaseapp.com" },
  { name: "갤러리", route: "sung-gallery.firebaseapp.com" },
  { name: "깃허브", route: "github.com/swc9803" },
  { name: "코드팬", route: "codepen.io/swc9803" },
];

const moveToPage = (route) => {
  open(`https://${route}`);
};

const canvasRef = ref();
const openBox = ref(0);

let renderer;
let camera;
let raf;
let mixer;
let fish;
let chest1, chest2, chest3, chest4;

const fishSpeed = 8;
const cameraY = 13; // obj로부터의 카메라 높이
const cameraZ = 13; // obj로부터의 카메라 거리

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x0c6ceb);

// 안개
scene.fog = new THREE.FogExp2(0x0c6ceb, 0.02); // 밀도

// 바닥
const floorGeometry = new THREE.CircleGeometry(26, 50);
const floorMaterial = new THREE.MeshBasicMaterial({ color: 0x0c48fa });
const floor = new THREE.Mesh(floorGeometry, floorMaterial);
floor.rotation.set(-Math.PI / 2, 0, 0);
floor.position.set(0, -1, 0);
scene.add(floor);

// light
const ambientLight = new THREE.AmbientLight(0xffffff, 1.6);
scene.add(ambientLight);
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(1, 1, 1);
scene.add(light);

const gltfLoader = new GLTFLoader();
const loadChest = (x, z) => {
  return new Promise((resolve) => {
    gltfLoader.load("/chest.glb", (gltf) => {
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

const offset = new THREE.Vector3(0, cameraY, -cameraZ);

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

const onResize = () => {
  const vh = window.innerHeight * 0.01;
  document.documentElement.style.setProperty("--vh", `${vh}px`);

  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(canvasRef.value.offsetWidth, canvasRef.value.offsetHeight);
  canvasRef.value.appendChild(renderer.domElement);
  camera.aspect = canvasRef.value.offsetWidth / canvasRef.value.offsetHeight;
  camera.updateProjectionMatrix();
};

let box1AnimationPlayed = false;
let box2AnimationPlayed = false;
let box3AnimationPlayed = false;
let box4AnimationPlayed = false;

const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

const onClick = (e) => {
  mouse.x = (e.clientX / canvasRef.value.offsetWidth) * 2 - 1;
  mouse.y = -(e.clientY / canvasRef.value.offsetHeight) * 2 + 1;

  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(floor);

  if (intersects.length > 0) {
    const intersectionPoint = intersects[0].point;
    console.log("클릭 좌표:", intersectionPoint);

    const distance = fish.position.distanceTo(intersectionPoint);
    const duration = distance / fishSpeed;

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

const onKeyDown = (e) => {
  if (e.key === "w" || e === "w") {
    wRef.value.style.background = "gray";
    gsap.to(fish.position, {
      z: "+=2",
    });
  } else if (e.key === "a" || e === "a") {
    aRef.value.style.background = "gray";
    gsap.to(fish.position, {
      x: "+=2",
    });
  } else if (e.key === "s" || e === "s") {
    gsap.to(fish.position, {
      z: "-=2",
    });
    sRef.value.style.background = "gray";
  } else if (e.key === "d" || e === "d") {
    gsap.to(fish.position, {
      x: "-=2",
    });
    dRef.value.style.background = "gray";
  }
};
const onKeyUp = (e) => {
  if (e.key === "w") {
    wRef.value.style.background = "white";
  } else if (e.key === "a") {
    aRef.value.style.background = "white";
  } else if (e.key === "s") {
    sRef.value.style.background = "white";
  } else if (e.key === "d") {
    dRef.value.style.background = "white";
  }
};
const onMouseDown = (key) => {
  onKeyDown(key);
};
const onMouseUp = () => {
  wRef.value.style.background = "white";
  aRef.value.style.background = "white";
  sRef.value.style.background = "white";
  dRef.value.style.background = "white";
};
onMounted(async () => {
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });

  gltfLoader.load("/fish.glb", (gltf) => {
    fish = gltf.scene;
    fish.rotation.set(0, Math.PI, 0);
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
    canvasRef.value.offsetWidth / canvasRef.value.offsetHeight,
    0.1,
    1000,
  );

  onResize();
  animate();

  window.addEventListener("keydown", onKeyDown);
  window.addEventListener("keyup", onKeyUp);
  window.addEventListener("mouseup", onMouseUp);
  window.addEventListener("resize", onResize);
  canvasRef.value.addEventListener("click", onClick);
});

onBeforeUnmount(() => {
  cancelAnimationFrame(raf);
  renderer.dispose();

  window.removeEventListener("keydown", onKeyDown);
  window.removeEventListener("keyup", onKeyUp);
  window.removeEventListener("mouseup", onMouseUp);
  window.removeEventListener("resize", onResize);
});
</script>

<style lang="scss" scoped>
.container {
  position: relative;
  width: 100%;
  height: calc(var(--vh) * 100);
  .button-wrapper {
    position: absolute;
    top: 5%;
    left: 5%;
    display: grid;
    grid-template-areas:
      ". w ."
      "a s d";
    gap: 8px;
    div {
      display: flex;
      justify-content: center;
      align-items: center;
      border: 2px solid black;
      border-radius: 10px;
      width: 50px;
      height: 50px;
      background: white;
      cursor: pointer;
      p {
        font-size: 1.2em;
        font-weight: 700;
        // pointer-events: none;
        user-select: none;
      }
    }
    #w {
      grid-area: w;
    }
    #a {
      grid-area: a;
    }
    #s {
      grid-area: s;
    }
    #d {
      grid-area: d;
    }
  }
  .wrapper {
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
  .content-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    width: 100%;
    height: 100%;
    padding: 20px;
    transform-origin: center;
    text-align: center;
    font-size: 2em;
    color: white;
    background: rgb(175, 175, 175);
    opacity: 0.8;
    button {
      border-radius: 16px;
      background: white;
      pointer-events: auto;
    }
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
