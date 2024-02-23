<template>
  <div class="resize-svg" v-html="svgContent"></div>
</template>

<script lang="ts">
import type { PropType } from "vue";
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    svgPath: {
      type: String as PropType<string>,
      required: true,
    },
  },
  data() {
    return {
      svgContent: "",
    };
  },
  async mounted() {
    try {
      const response = await fetch(this.svgPath);
      this.svgContent = await response.text();
    } catch (error) {
      console.error("Error fetching SVG:", error);
    }
  },
});
</script>

<style>
.resize-svg svg {
  width: 100px;
  height: auto;
}
</style>
