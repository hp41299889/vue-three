<template>
  <div ref="container" class="container" @scroll="onScroll"></div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { SVGLoader } from "three/addons/loaders/SVGLoader.js";

const container = ref<HTMLDivElement | null>(null);

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  1,
  1000
);
const renderer = new THREE.WebGL1Renderer();
const line1 = new THREE.Group();

const init = () => {
  scene.background = new THREE.Color(0xb0b0b0);
  camera.position.set(0, 0, 100);

  if (container.value) {
    const { offsetWidth, offsetHeight } = container.value;
    renderer.setSize(window.innerWidth, window.innerHeight);
    container.value.appendChild(renderer.domElement);
  }

  const loader = new SVGLoader();
  loader.load("/line1.svg", (data) => {
    line1.scale.multiplyScalar(0.25);
    line1.position.x = -15;
    line1.position.y = 30;
    line1.scale.y *= -1;
    for (const path of data.paths) {
      const fillColor = path.userData?.style.fill;
      if (fillColor !== undefined && fillColor !== "none") {
        const material = new THREE.MeshBasicMaterial({
          color: new THREE.Color().setStyle(fillColor),
          opacity: path.userData?.style.fillOpacity,
          transparent: true,
          side: THREE.DoubleSide,
          depthWrite: false,
        });
        const shapes = SVGLoader.createShapes(path);
        for (const shape of shapes) {
          const geometry = new THREE.ShapeGeometry(shape);
          const mesh = new THREE.Mesh(geometry, material);

          line1.add(mesh);
        }
      }
      const strokeColor = path.userData?.style.stroke;
      if (strokeColor !== undefined && strokeColor !== "none") {
        const material = new THREE.MeshBasicMaterial({
          color: new THREE.Color().setStyle(strokeColor),
          opacity: path.userData?.style.strokeOpacity,
          transparent: true,
          side: THREE.DoubleSide,
          depthWrite: false,
        });

        for (const subPath of path.subPaths) {
          const geometry = SVGLoader.pointsToStroke(
            subPath.getPoints(),
            path.userData?.style
          );

          if (geometry) {
            const mesh = new THREE.Mesh(geometry, material);

            line1.add(mesh);
          }
        }
      }
    }
    scene.add(line1);

    const box = new THREE.Box3().setFromObject(line1);
    const center = new THREE.Vector3();
    box.getCenter(center); // 计算中心点
    line1.position.sub(center);

    renderCanvas();
  });
  window.addEventListener("scroll", onScroll);
  window.addEventListener("scroll", onScrollFixCanvas);
  window.addEventListener("resize", onWindowResize);
};
const renderCanvas = () => renderer.render(scene, camera);

onMounted(() => {
  init();
});

const onScroll = () => {
  const scrollY = window.scrollY || document.documentElement.scrollTop;
  const rotation = scrollY * 0.01; // 根据需要调整旋转速度
  line1.rotation.y = rotation; // 更新Y轴旋转
  console.log(line1);

  renderCanvas();
};

const onScrollFixCanvas = () => {
  const yOffset = window.scrollY || document.documentElement.scrollTop;
  const yCenter =
    yOffset + window.innerHeight / 2 - renderer.domElement.offsetHeight / 2;
  const xCenter = window.innerWidth / 2 - renderer.domElement.offsetWidth / 2;

  renderer.domElement.style.position = "absolute";
  renderer.domElement.style.top = `${yCenter}px`;
  renderer.domElement.style.left = `${xCenter}px`;
};

const onWindowResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();

  renderer.setSize(window.innerWidth, window.innerHeight);
  renderCanvas();
};
</script>

<style>
.container {
  height: 3000px;
}
</style>
