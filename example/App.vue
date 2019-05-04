<template>
  <div style="background: #eee;">
    <Carousel style="width:100%; height: 80vh;">
      <Slide>
        <div class="sample bg-primary">
          <h1>slide 1</h1>
          <p>test test test</p>
        </div>
      </Slide>
      <Slide>
        <div class="sample bg-warning">
          <h1>slide 2</h1>
          <p>test test test</p>
        </div>
      </Slide>
      <Slide>
        <div class="sample bg-danger">
          <h1>slide 3</h1>
          <p>test test test</p>
        </div>
      </Slide>
    </Carousel>
  </div>
</template>

<script>
import { Carousel, Slide } from '../src/index';

export default {
  components: {
    Carousel,
    Slide
  }
}
</script>

<style>
.sample { 
  box-sizing: border-box;
  margin: 0; 
  padding: 30px 60px; 
  height: 100%;
}
.bg-primary { background: hsl(171, 100%, 41%) }
.bg-warning { background: hsl( 48, 100%, 67%) }
.bg-danger  { background: hsl(348, 100%, 61%) }

h1 { 
  margin: 0;
  padding: 0;
}
</style>

