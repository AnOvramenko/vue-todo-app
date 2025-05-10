<script setup>
import { onBeforeMount, onMounted, ref } from 'vue';

const emit = defineEmits("close");
const text = ref('');
let timer = null;

const hide = () => {
  text.value = '';

  if (timer) {
    clearTimeout(timer);
    timer = null;
  }
}

const show = (newText) => {
  if (timer) {
    clearTimeout(timer);
  }

  timer = setTimeout(() => {
    hide();
  }, 3000);

  text.value = newText;
}

defineExpose({ show, hide});

</script>

<template>
  <article class="message" :class="{ 'message--hidden': !text }">
    <div class="message-header">
      <p>Error</p>
      <button class="delete" @click="hide"></button>
    </div>

    <div class="message-body">{{ text }}</div>
  </article>
</template>

<style scoped lang="scss">
.message {
  transform-origin: top center;
  transition-property: opacity transform;
  transition-duration: 0.3s;

  &--hidden {
    transform: scaleY(0);
    opacity: 0;
    pointer-events: none;
  }
}
</style>
