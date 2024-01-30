<script lang="ts">
  /*
  renders only 3 slides - previous, current and next
  shows only the current slide
  when transition to the direction of the next slide starts:
    - all slides receive the class animation-setup & forward/backward class
    - forward/backward is responsible for the direction of the animation and animation name, the slide class is also involved,
      because animation of current slide differs from the animation of the next slide
      only 2 slides participate in animation - current and next/previous
      also need to consider z-index - the heap of slides should be in the correct order
  
  animation duration should match changing the classes
    .animation-setup {
      animation-iteration-count: 1;
      animation-fill-mode: forwards;
      animation-timing-function: ease-in-out;
      animation-duration: 0.5s;
    }

    .next-slide.forward {
      transform-origin: 100% 50%;
      animation-name: next-to-next;
    }
    .current-slide.forward {
      animation-name: current-forward;
    }
    for cards alike animation
    @keyframes next-to-next {
      0% {
        transform: rotateY(60deg);
        opacity: 0;
      }
      100% {
        transform: rotateY(0deg);
        opacity: 1;
      }
    }
    @keyframes current-to-next {
      0% {
        opacity: 1;
      }
      100% {
        opacity: 0;
        transform: translateY(-50%);
      }
    }

    the opposite direction 
    .previous-slide.backward {
      animation-name: previous-backward;
    }
    .current-slide.backward {
      transform-origin: 100% 50%;
      animation-name: current-backward;
    }
    @keyframes previous-backward {
      0% {
        opacity: 0;
        transform: translateY(-50%);
      }
      100% {
        opacity: 1;
        transform: translateY(0);
      }
    }
    @keyframes current-backward {
      0% {
        opacity: 1;
      }
      100% {
        opacity: 0;
        transform: rotateY(-60deg);
      }
    }

    animation names can be read as: [slide by order]-[direction] - next-forward, current-forward, current-backwards, previous-backward

    after animation is done, the class animation-setup is removed from all slides

    From algorithm perspective:
     - we have an array of slides, type doesn't matter
     - when slide change was called slides receive their classes
     - after animation finished, classes are removed and previous, current and next slides are updated
  */
  
  /*
  
  */
  import { spring, tweened } from 'svelte/motion'
  
  const DURATION = 2000
  const array = [0, 1, 2, 3, 4]
  let pointer = 0
  $: current = array[pointer]
  $: previous = pointer - 1 < 0 ? array[0] : array[pointer - 1]
  $: next = pointer + 1 > array.length - 1 ? array.at(-1) : array[pointer + 1]
  let previousSlideClass = ''
  let currentSlideClass = ''
  let nextSlideClass = ''
  let timeout: number | null = null
  const goNext = () => {
    const clearAnimation = () => {
      pointer = pointer + 1 > array.length - 1 ? pointer : pointer + 1
      currentSlideClass = ''
      nextSlideClass = ''
    }
    if (timeout) {
      clearTimeout(timeout)
    }
    if (pointer + 1 > array.length - 1) return
    nextSlideClass = 'animation-setup forward'
    currentSlideClass = 'animation-setup forward'
    timeout = setTimeout(clearAnimation, DURATION)
  }
  const goPrev = () => {
    const clearAnimation = () => {
      pointer = pointer - 1 < 0 ? pointer : pointer - 1
      currentSlideClass = ''
      previousSlideClass = ''
    }
    if (timeout) {
      clearTimeout(timeout)
    }
    if (pointer - 1 < 0) return
    previousSlideClass = 'animation-setup backward'
    currentSlideClass = 'animation-setup backward'
    timeout = setTimeout(clearAnimation, DURATION)
  }

  const progress = tweened(0, {
    duration: DURATION,
  })
  const increase = () => {
    pointer = pointer + 1 > array.length - 1 ? pointer : pointer + 1
  }
  const goNextTweened = async () => {
    if (timeout && pointer !== array.length - 2) {
      clearTimeout(timeout)
      increase()
    }
    if (pointer + 1 > array.length - 1) {
      return
    }
    timeout = setTimeout(() => {
      increase()
      timeout = null
    }, DURATION)
    await progress.set(0, { duration: 0 })
    await progress.set(1)
    await progress.set(0, { duration: 0 })
  }

  const goPrevTweened = async () => {
    await progress.set(1, { duration: 0 })
    await progress.set(0)
  }
</script>
<div>pointer: {pointer}</div>
<div class="slider">
  <div
    class="slide previous-slide {previousSlideClass}">{previous}</div>
  <div
    class="slide current-slide {currentSlideClass}"
  >{current}</div>
  <div
    class="slide next-slide {nextSlideClass}"
    style:transform={`rotateY(${60 - 60 * $progress}deg)`}
    style:opacity={$progress}
  >{next}</div>
</div>
<div class="controls">
  <button on:click={goPrevTweened}>previous</button>
  <button on:click={goNextTweened}>next</button>
</div>
<style>
  .slider {
    height: 400px;
    perspective: 2000px;
    position: relative;
  }
  .slide {
    height: 400px;
    background-color: #60a87f;
  }
  .next-slide {
    transform-origin: 100% 50%;
  }
  .previous-slide,
  .next-slide {
    opacity: 0;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
  .animation-setup {
    animation-iteration-count: 1;
    animation-fill-mode: forwards;
    animation-timing-function: ease-in-out;
    animation-duration: 2s;
  }
  .next-slide.forward {
    transform-origin: 100% 50%;
    animation-name: next-forward;
  }
  .current-slide.forward {
    animation-name: current-forward;
  }

  @keyframes next-forward {
    0% {
      transform: rotateY(60deg);
      opacity: 0;
    }
    100% {
      transform: rotateY(0deg);
      opacity: 1;
    }
  }
  @keyframes current-forward {
    0% {
      opacity: 1;
    }
    100% {
      opacity: 0;
      transform: translateX(-20%);
    }
  }

  .previous-slide.backward {
    animation-name: previous-backward;
  }
  .current-slide.backward {
    transform-origin: 100% 50%;
    animation-name: current-backward;
  }
  @keyframes previous-backward {
    0% {
      opacity: 0;
      transform: translateX(-20%);
    }
    100% {
      opacity: 1;
      transform: translateY(0);
    }
  }
  @keyframes current-backward {
    0% {
      opacity: 1;
    }
    100% {
      opacity: 0;
      transform: rotateY(60deg);
    }
  }
  .controls {
    display: flex;
    margin-top: 1rem;
    justify-content: flex-end;
    gap: 1rem;
  }
</style>