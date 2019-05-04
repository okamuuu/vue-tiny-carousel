<template>
  <div class="carousel">
    <div ref="carousel-wrapper" class="carousel-wrapper">
      <div 
        ref="carousel-inner"
        class="carousel-inner"
        :style="{
          'transform': `translate(${currentOffset}px, 0)`,
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
    this.computeCarouselHeight();
  },
  beforeUpdate() {
    this.computeCarouselWidth();
    this.computeCarouselHeight();
  },
  provide() {
    return {
      carousel: this
    };
  },
  data() {
    return {
      browserWidth: null,
      carouselWidth: 0,
      currentPage: 1,
      dragging: false,
      offset: 0,
      speed: 500,
      refreshRate: 16,
      slideCount: 0,
      transitionstart: "transitionstart",
      transitionend: "transitionend",

      // 固定値
      currentHeight: "auto",
      currentPerPage: 1,

      easing: "ease",
      tagName: "slide",
      spacePadding: 0,
      scrollPerPage: 1,
      spacePaddingMaxOffsetFactor: 0,
    };
  },
  computed: {
    maxOffset() {
      return Math.max(
        this.slideWidth * (this.slideCount - this.currentPerPage) -
          this.spacePadding * this.spacePaddingMaxOffsetFactor,
        0
      );
    },
    pageCount() {
      return this.scrollPerPage
        ? Math.ceil(this.slideCount / this.currentPerPage)
        : this.slideCount - this.currentPerPage + 1;
    },
    slideWidth() {
      return this.carouselWidth - this.spacePadding * 2;
    },
    isNavigationRequired() {
      return this.slideCount > this.currentPerPage;
    },
    isCenterModeEnabled() {
      return this.centerMode && !this.isNavigationRequired;
    },
    transitionStyle() {
      const speed = `${this.speed / 1000}s`;
      const transtion = `${speed} ${this.easing} transform`;
      if (this.adjustableHeight) {
        return `${transtion}, height ${speed} ${this.adjustableHeightEasing ||
          this.easing}`;
      }
      return transtion;
    },
    padding() {
      const padding = this.spacePadding;
      return padding > 0 ? padding : false;
    },
    currentOffset() {
      return this.carouselWidth * (this.currentPage - 1) * -1 
    },
    canAdvanceForward() {
      return this.slideCount - this.currentPage > 0
    },
    canAdvanceBackward() {
      console.log(this.currentPage)
      return this.currentPage > 1;
    }
  },
  methods: {

    // STEP 1
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
    computeCarouselHeight() {
      let carouselInnerElements = this.$el.getElementsByClassName("carousel-inner");
      this.carouselHeight = carouselInnerElements[0].clientHeight || 0;
    },

    handleNavigation(direction) {
      this.advancePage(direction);
    },

    advancePage(direction) {
      if (direction && direction === "backward") {
        this.goPreviousPage()
      } else if (direction && direction === "forward") {
        this.goNextPage()
      }
    },

    // STEP 2
    onResize() {
      this.computeCarouselWidth();
      this.computeCarouselHeight();
      this.dragging = true; // force a dragging to disable animation
      
      // clear dragging after refresh rate
      setTimeout(() => {
        this.dragging = false;
      }, this.refreshRate);
    },
    render() {
      // add extra slides depending on the momemtum speed
      if (this.rtl) {
        this.offset -=
          Math.max(
            -this.currentPerPage + 1,
            Math.min(Math.round(this.dragMomentum), this.currentPerPage - 1)
          ) * this.slideWidth;
      } else {
        this.offset +=
          Math.max(
            -this.currentPerPage + 1,
            Math.min(Math.round(this.dragMomentum), this.currentPerPage - 1)
          ) * this.slideWidth;
      }
      // & snap the new offset on a slide or page if scrollPerPage
      const width = this.scrollPerPage
        ? this.slideWidth * this.currentPerPage
        : this.slideWidth;
      // lock offset to either the nearest page, or to the last slide
      const lastFullPageOffset =
        width * Math.floor(this.slideCount / (this.currentPerPage - 1));
      const remainderOffset =
        lastFullPageOffset +
        this.slideWidth * (this.slideCount % this.currentPerPage);
      if (this.offset > (lastFullPageOffset + remainderOffset) / 2) {
        this.offset = remainderOffset;
      } else {
        this.offset = width * Math.round(this.offset / width);
      }
      // clamp the offset between 0 -> maxOffset
      this.offset = Math.max(0, Math.min(this.offset, this.maxOffset));
      // update the current page
      this.currentPage = this.scrollPerPage
        ? Math.round(this.offset / this.slideWidth / this.currentPerPage)
        : Math.round(this.offset / this.slideWidth);
    },
    goToPage(page, advanceType) {
      if (page >= 0 && page <= this.pageCount) {
        this.offset = this.scrollPerPage
          ? Math.min(
              this.slideWidth * this.currentPerPage * page,
              this.maxOffset
            )
          : this.slideWidth * page;

        this.currentPage = page
      }
    },
    getNextPage() {
      if (this.currentPage < this.pageCount) {
        return this.currentPage + 1;
      }
      return this.currentPage;
    },
    getPreviousPage() {
      if (this.currentPage > 1) {
        return this.currentPage - 1;
      }
      return this.currentPage;
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
