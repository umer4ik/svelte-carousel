<script lang="ts">
  import Carousel from '../carousel/Carousel.svelte';
  let className = '';
  const toggleClassName = () => {
    className = className ? '' : 'transition-start'
  }
</script>

<div class="section">
  <Carousel></Carousel>
</div>
<div class="section">
  <div class="playground">
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="slide {className}" on:click={toggleClassName}></div>
  </div>
</div>

<style>
  .section {
    margin: 0 auto;
    max-width: 960px;
    padding: 1.45rem 1.0875rem;
  }
  .playground {
    padding-top: 3rem;
    perspective: 2000px;
  }
  .slide {
    height: 400px;
    background-color: #60a87f;
    transform-origin: 100% 50%;
    
  }
  .slide.transition-start {
    animation: slide 2s ease-in-out 1;
  }
  @keyframes slide {
    0% {
      opacity: 0;
      transform: rotateY(60deg);
    }
    100% {
      opacity: 1;
      transform: rotateY(0deg);
    }
  }
</style>