<script setup lang="ts">
import * as THREE from 'three';
import { ref, onMounted } from 'vue';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const canvasWrapper = ref();
let scene = null;
let camera = null;
let renderer = null;
let sphere = null;
let mouseX = 0;
let mouseY = 0;

// 定义三个点
const points1 = [
    { x: 0, y: 0, z: 0 },
    { x: 2, y: 0, z: 0 },
    { x: 2, y: 2, z: 0 },
    { x: 2, y: 2, z: 2 },
    { x: 0, y: 2, z: 2 },
    { x: 0, y: 0, z: 2 }
];

const points2 = [
    { x: 0, y: 0, z: 0 },
    { x: 0, y: 0, z: 2 },
    { x: 0, y: -2, z: 2 },
    { x: 2, y: -2, z: 2 },
    { x: 2, y: -2, z: 0 },
    { x: 2, y: 0, z: 0 }
];

let points = points1;

// 当前目标点索引
let targetIndex = 0;

// 球体移动速度
const speed = 0.02;

onMounted(() => {
  initScene();
  initCamera();
  initRenderer();
  addSphere();
  animate();
  //setInterval(() => {
  //  if(points === points1) {
  //    points = points2;
  //  } else {
  //    points = points1;
  //  }
  //}, 10000);
  //document.addEventListener('mousemove', onMouseMove, false);

  document.addEventListener('mousemove', getMouse, false)
  // 设置相机控件轨道控制器OrbitControls
  const controls = new OrbitControls(camera, renderer.domElement);
  // 如果OrbitControls改变了相机参数，重新调用渲染器渲染三维场景
  controls.addEventListener('change', function () {
    renderer.render(scene, camera); //执行渲染操作
  });//监听鼠标、键盘事件
});

function initScene() {
  scene = new THREE.Scene();
}

function initCamera() {
  const width = canvasWrapper.value.clientWidth;
  const height = canvasWrapper.value.clientHeight;
  camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
  const axesHelper = new THREE.AxesHelper(150); // AxesHelper：辅助观察的坐标系
  scene.add(axesHelper);
}

function initRenderer() {
  const width = canvasWrapper.value.clientWidth;
  const height = canvasWrapper.value.clientHeight;
  renderer = new THREE.WebGLRenderer();
  renderer.setSize(width, height);
  canvasWrapper.value.appendChild(renderer.domElement);
}

function addSphere() {
  //const geometry = new THREE.SphereGeometry(1, 10, 10);
  //const geometry = new THREE.CylinderGeometry(1,1,5);
  const geometry = new THREE.BoxGeometry(1, 1, 1);
  const material = new THREE.MeshBasicMaterial({ 
    color: 0xff0000, 
    transparent: true,//开启透明
    opacity: 0.5,//设置透明度
    side: THREE.DoubleSide, //两面可见
    //side: THREE.FrontSide, //默认只有正面可见 
  });
  sphere = new THREE.Mesh(geometry, material);
  camera.position.z = 5;
  camera.lookAt(0, 0, 0); 
  scene.add(sphere);
}
function animate() {
  //requestAnimationFrame(animate);

  // 球体移动逻辑
  sphere.rotation.x += 0.01;
  sphere.rotation.y += 0.01;
  
  //renderer.render(scene, camera);
  requestAnimationFrame(animate);

  moveBySpeed();

  renderer.render(scene, camera);
}

function move(x, y, z) {
  sphere.position.x = x;
  sphere.position.y = y;
  sphere.position.z = z;
  console.log(x, y, z);
}

function moveBySpeed() {
  // 当前点和目标点
  const currentPoint = { x: sphere.position.x, y: sphere.position.y, z: sphere.position.z };
  const targetPoint = points[targetIndex]

  // 计算方向向量
  const direction = {
    x: targetPoint.x - currentPoint.x,
    y: targetPoint.y - currentPoint.y,
    z: targetPoint.z - currentPoint.z
  };

  // 计算距离
  const distance = Math.sqrt(direction.x * direction.x + direction.y * direction.y + direction.z * direction.z);

  // 如果距离很小，切换到下一个目标点
  if (distance < 0.1) {
    targetIndex = (targetIndex + 1) % points.length;
  } else {
    // 归一化方向向量
    const normalizedDirection = {
      x: direction.x / distance,
      y: direction.y / distance,
      z: direction.z / distance
    };

    // 更新球体位置
    sphere.position.x += normalizedDirection.x * speed;
    sphere.position.y += normalizedDirection.y * speed;
    sphere.position.z += normalizedDirection.z * speed;
  }
}

function onMouseMove(event) {
  // 计算鼠标在屏幕上的位置，转换为 Three.js 坐标系中的位置；计算鼠标在归一化设备坐标中的位置（-1 到 +1）
  mouseX = (event.clientX / window.innerWidth) * 2 - 1;
  mouseY = -(event.clientY / window.innerHeight) * 2 + 1;

  // 将归一化后的鼠标位置映射到 Three.js 场景中的球体位置
  const movementScale = 2;
  sphere.position.x = mouseX * movementScale;
  sphere.position.y = mouseY * movementScale;
}

function getMouse(event) {
  // 计算鼠标在屏幕上的位置，转换为 Three.js 坐标系中的位置；计算鼠标在归一化设备坐标中的位置（-1 到 +1）
  mouseX = (event.clientX / window.innerWidth) * 10 - 1;
  mouseY = -(event.clientY / window.innerHeight) * 10 + 1;
  //mouseZ = 
  console.log(mouseX, mouseY);
  points.push({x: mouseX, y: mouseY, z: 0 })
}

</script>

<template>
  <div ref="canvasWrapper" class="canvas-wrapper"></div>
</template>

<style scoped>
.canvas-wrapper {
  border: 1px red solid;
  width: 1000px;
  height: 1000px;
}
</style>
