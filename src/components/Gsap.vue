<template>
  <div class="container" ref="container">
    <Svg ref="line1" svgPath="src/assets/line1.svg"></Svg>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import { gsap } from "gsap";

import Svg from "@/components/Svg.vue";

const container = ref<any>(null);
const line1 = ref<any>(null);

const init = () => {
  window.addEventListener("scroll", onScroll);
};

const onScroll = () => {
  const y = window.scrollY;
  gsap.to(line1.value.$el.querySelector("svg"), {
    rotation: y * 0.05,
    transformOrigin: "center center",
  });
};

onMounted(() => {
  init();
});
</script>

<style>
.container {
  height: 300vh;
}
</style>
