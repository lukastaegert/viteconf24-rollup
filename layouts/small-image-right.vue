<template>
  <div class="grid grid-cols-[650px_1fr] w-full h-full">
    <div class="slidev-layout default">
      <slot />
    </div>
    <div class="w-full h-full bg-cover bg-center" :style="style"></div>
  </div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  image: { type: String },
  position: { type: String, default: "center" },
});

const style = computed(
  () => ({
    backgroundImage: `linear-gradient(#fff5, #fffa),url('${import.meta.env.BASE_URL}${props.image}')`,
    backgroundPositionX: props.position,
  })
);
</script>
