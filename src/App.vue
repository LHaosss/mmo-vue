<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";
import { TGALoader } from "three/examples/jsm/loaders/TGALoader";
import Stats from "three/examples/jsm/libs/stats.module";
import { onMounted } from "vue";

// 初始化配置
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.z = 180;
camera.position.y = 120;
// 挂载页面
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setClearColor(0xd3df56, 1);
renderer.shadowMap.enable = true;
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
document.body.appendChild(renderer.domElement);

const stats = new Stats();
renderer.domElement.appendChild(stats.dom);

// 添加相机轨道
const controls = new OrbitControls(camera, renderer.domElement);
controls.update();

// 添加光源
const dirLight = new THREE.DirectionalLight(0xffffff, 3);
dirLight.position.set(-3, 10, -10);
dirLight.castShadow = true;
dirLight.shadow.camera.top = 4;
dirLight.shadow.camera.bottom = -4;
dirLight.shadow.camera.left = -4;
dirLight.shadow.camera.right = 4;
dirLight.shadow.camera.near = 0.1;
dirLight.shadow.camera.far = 40;
scene.add(dirLight);

const ambientlight = new THREE.AmbientLight(0xededed, 0.5); // 柔和的白光
scene.add(ambientlight);

let mixer = null;

// 加载一个含有贴图的场地
var loader = new THREE.TextureLoader();
const texture = loader.load("/map.jpg");

const mesh = new THREE.Mesh(
  new THREE.PlaneGeometry(600, 600),
  new THREE.MeshPhongMaterial({
    color: 0xcbcbcb,
    depthWrite: false,
    map: texture,
  })
);
mesh.rotation.x = -Math.PI / 2;

mesh.receiveShadow = true;
scene.add(mesh);

// 加载模型并添加到场景中
let obj = null;
var runAction = null
var stopAction = null
const fbxLoader = new FBXLoader();

fbxLoader.load("/Fast Run.fbx", (fbx) => {
  obj = fbx;
  scene.add(fbx);

  mixer = new THREE.AnimationMixer(fbx);
  console.log(fbx)
  runAction = mixer.clipAction(fbx.animations[0]);
  runAction.timeScale = 0.7;
  stopAction = mixer.clipAction(fbx.animations[1])

  fbx.traverse((child) => {
    if (child.isMesh) {
      child.castShadow = true;
      const tgaLoader = new TGALoader();
      tgaLoader.load("/robo/textures/robo_albedo.tga", (texture) => {
        texture.needsUpdate = true;
        child.material.map = texture;
      });
    }
  });

  // fbx.scale.set(0.05, 0.05, 0.05);
  fbx.rotateY(110);
  // fbx.position.set(0, -3, 0)
  fbx.translateY(-2);

  // fbx.position.y = -2;

  const skeletion = new THREE.SkeletonHelper(fbx);
  skeletion.visible = true;
  scene.add(skeletion);
});

const clock = new THREE.Clock();
const lod = new THREE.LOD();

function animate() {
  requestAnimationFrame(animate);
  lod.update(camera);

  const delta = clock.getDelta();
  if (mixer) mixer.update(delta);
  controls.update();
  stats.update();

  renderer.render(scene, camera);
}

renderer.render(scene, camera);

animate();

onMounted(() => {
  document.onkeydown = function (event) {
    let key = window.event.keyCode;
    if (key === 87) {
      // W
      // runAction.crossFadeFrom(stopAction, 1, true)
      runAction.play();
      obj.rotation.set(0, Math.PI, 0);
      obj.translateZ(6);
    } else if (key === 65) {
      // A
      runAction.play();
      obj.rotation.set(0, 0 - Math.PI * 0.5, 0);
      obj.translateZ(6);
    } else if (key === 83) {
      // S
      runAction.play();
      obj.rotation.set(0, 0, 0);
      obj.translateZ(6);
    } else if (key === 68) {
      // D
      runAction.play();
      obj.rotation.set(0, Math.PI * 0.5, 0);
      obj.translateZ(6);
    }
  };

  window.onkeyup = function () {
    runAction.crossFadeTo(stopAction, 0.5)
    setTimeout(()=>{
      runAction.stop();
    }, 500)
  }
});
</script>

<template></template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
