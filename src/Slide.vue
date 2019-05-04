<template>
    <div class="slide">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "slide",
}
</script>

<style scoped>
.slide {
  flex-basis: inherit;
  flex-grow: 0;
  flex-shrink: 0;
  user-select: none;
  backface-visibility: hidden;
  -webkit-touch-callout: none;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  outline: none;
}
</style>
