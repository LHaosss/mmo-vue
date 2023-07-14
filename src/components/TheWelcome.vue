<script setup>
import { onMounted } from "vue";
import * as THREE from "three";

// 初始化渲染器
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innnerHeight);

// 初始化场景
const scene = new THREE.Scene();
scene.background = "#0x000000";
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innnerHeight,
  0.1,
  1000
);
// 设置相机位置
camera.position.y = 10;
camera.position.z = 10;

const geometry = new THREE.BoxGeometry(2,2,2);
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}

animate();

onMounted(() => {
  // 将renderer添加到页面中
  const e = document.getElementsByClassName("scene");
  e[0].appendChild(renderer.domElement);
});
</script>

<template>
  <div class="scene"></div>
</template>
