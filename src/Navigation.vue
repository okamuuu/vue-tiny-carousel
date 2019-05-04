<template>
  <div class="carousel-navigation">
    <button
      type="button"
      aria-label="Previous page"
      class="carousel-navigation-button carousel-navigation-prev"
      v-on:click.prevent="triggerPageAdvance('backward')"
      v-bind:class="{ 'carousel-navigation--disabled': !canAdvanceBackward }"
      v-bind:style="`padding: ${clickTargetSize}px; margin-right: -${clickTargetSize}px;`"
      v-html="prevLabel"></button>
    <button
      type="button"
      aria-label="Next page"
      class="carousel-navigation-button carousel-navigation-next"
      v-on:click.prevent="triggerPageAdvance('forward')"
      v-bind:class="{ 'carousel-navigation--disabled': !canAdvanceForward }"
      v-bind:style="`padding: ${clickTargetSize}px; margin-left: -${clickTargetSize}px;`"
      v-html="nextLabel"></button>
  </div>
</template>

<script>
/* eslint-disable no-console  */
export default {
  name: "navigation",
  inject: ["carousel"],
  props: {
    clickTargetSize: {
      type: Number,
      default: 8
    },
    nextLabel: {
      type: String,
      default: "&#9654"
    },
    prevLabel: {
      type: String,
      default: "&#9664"
    }
  },
  computed: {
    canAdvanceForward() {
      return this.carousel.canAdvanceForward || false;
    },
    canAdvanceBackward() {
      return this.carousel.canAdvanceBackward || false;
    }
  },
  methods: {
    triggerPageAdvance(direction) {
      this.$emit("navigationclick", direction);
    }
  }
};
</script>

<style scoped>
.carousel-navigation-button {
  position: absolute;
  top: 50%;
  box-sizing: border-box;
  color: #000;
  text-decoration: none;
  appearance: none;
  border: none;
  background-color: transparent;
  padding: 0;
  cursor: pointer;
  outline: none;
}
.carousel-navigation-button:focus {
  outline: 1px solid lightblue;
}
.carousel-navigation-next {
  right: 45px ;
  font-size: 20px;
  transform: translateY(-50%) translateX(100%);
  font-family: "system";
}
.carousel-navigation-prev {
  left: 45px;
  font-size: 20px;
  transform: translateY(-50%) translateX(-100%);
  font-family: "system";
}
.carousel-navigation--disabled {
  opacity: 0.2;
  cursor: default;
}
/* Define the "system" font family */
@font-face {
  font-family: system;
  font-style: normal;
  font-weight: 300;
  src: local(".SFNSText-Light"), local(".HelveticaNeueDeskInterface-Light"),
    local(".LucidaGrandeUI"), local("Ubuntu Light"), local("Segoe UI Symbol"),
    local("Roboto-Light"), local("DroidSans"), local("Tahoma");
}
</style>
