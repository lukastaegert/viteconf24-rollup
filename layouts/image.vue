<template>
  <div
    class="slidev-layout default grid justify-center content-center bg-cover bg-center"
    :style="style"
  >
    <slot />
  </div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  image: { type: String },
});

const style = computed(
  () => ({
    color: "white",
    backgroundImage: `linear-gradient(#0005, #000a),url('${import.meta.env.BASE_URL}${props.image}')`,
  })
);
</script>
