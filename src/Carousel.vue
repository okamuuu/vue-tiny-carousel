<template>
  <div class="carousel">
    <div ref="carousel-wrapper" class="carousel-wrapper">
      <div 
        ref="carousel-inner"
        class="carousel-inner"
        :style="{
          'transform': `translate(${offset}px, 0)`,
          'transition': dragging ? 'none' : `0.4s ease transform`,
          'flex-basis': `${carouselWidth}px`,
        }"
      >
        <slot></slot>
      </div>
    </div>

    <navigation
      :clickTargetSize=8
      :nextLabel="`>`"
      :prevLabel="`<`"
      @navigationclick="handleNavigation"
    />

  </div>
</template>

<script>
/* eslint-disable no-console  */
import debounce from "./utils/debounce";
import Navigation from "./Navigation.vue";

export default {
  name: "carousel",
  components: {
    Navigation
  },
  mounted() {
    window.addEventListener(
      "resize",
      debounce(this.onResize, this.refreshRate)
    );
    this.setSlideCount()
    this.computeCarouselWidth();
  },
  beforeUpdate() {
    // this.computeCarouselWidth();
  },
  provide() {
    return {
      carousel: this
    };
  },
  data() {
    return {
      slideCount: 0,
      currentPage: 1,
      carouselWidth: 0,
      dragging: false,
      refreshRate: 16,
    };
  },
  computed: {
    offset() {
      return this.carouselWidth * (this.currentPage - 1) * -1 
    },
    canAdvanceForward() {
      return this.slideCount - this.currentPage > 0
    },
    canAdvanceBackward() {
      return this.currentPage > 1;
    }
  },
  methods: {
    setSlideCount() {
      const slots = this.$slots
      const tagName = "slide"
      const isSlide = slot => slot.tag && slot.tag.match(`^vue-component-\\d+-${tagName}$`) !== null
      this.slideCount = (slots && slots.default && slots.default.filter(isSlide)).length || 0
    },
    computeCarouselWidth() {
      let carouselInnerElements = this.$el.getElementsByClassName("carousel-inner");
      this.carouselWidth = carouselInnerElements[0].clientWidth || 0;
    },

    handleNavigation(direction) {
      if (direction && direction === "backward") {
        this.goPreviousPage()
      } else if (direction && direction === "forward") {
        this.goNextPage()
      }
    },

    onResize() {
      this.computeCarouselWidth();
      this.dragging = true; // force a dragging to disable animation
      
      // clear dragging after refresh rate
      const refreshRate = 16
      setTimeout(() => {
        this.dragging = false;
      }, refreshRate);
    },
    goToPage(page) {
      this.currentPage = page
    },
    getNextPage() {
      return this.currentPage < this.slideCount ? this.currentPage + 1 : this.currentPage;
    },
    getPreviousPage() {
      return this.currentPage > 1 ? this.currentPage - 1 : this.currentPage
    },
    goNextPage() {
      this.goToPage(this.getNextPage())
    },
    goPreviousPage() {
      this.goToPage(this.getPreviousPage())
    }
  }
}
</script>

<style scoped>
.carousel {
  height: 100%;
  display: flex;
  flex-direction: column;
  position: relative;
}
.carousel-wrapper {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
}
.carousel-inner {
  height: 100%;
  display: flex;
  flex-direction: row;
  backface-visibility: hidden;
}
</style>
