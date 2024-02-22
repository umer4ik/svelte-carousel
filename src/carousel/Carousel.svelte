<script lang="ts">
  import { tweened } from 'svelte/motion';
  import { swipe } from 'svelte-gestures';

  const array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
  let pointer = 0;
  $: current = array[pointer];
  $: previous = pointer - 1 < 0 ? array[0] : array[pointer - 1];
  $: next = pointer + 1 > array.length - 1 ? array.at(-1) : array[pointer + 1];
  const DURATION = 400;
  const ANGLE = 30;
  const SHIFT = 50;

  let fExpected = pointer;
  let bExpected = pointer;
  const fProgress = tweened(0, {
    duration: DURATION,
  });
  const bProgress = tweened(0, {
    duration: DURATION,
  });
  const goForward = async () => {
    if (fExpected === array.length - 1) {
      return;
    }
    await bProgress.set(0, { duration: 0 });
    pointer = fExpected;
    fExpected += 1;
    bExpected = fExpected;
    await fProgress.set(0, { duration: 0 });
    await fProgress.set(1);
    await fProgress.set(0, { duration: 0 });
    pointer = fExpected;
  };

  const goBackward = async () => {
    if (bExpected === 0) {
      return;
    }
    await fProgress.set(0, { duration: 0 });
    pointer = bExpected;
    bExpected -= 1;
    fExpected = bExpected;
    await bProgress.set(0, { duration: 0 });
    await bProgress.set(1);
    await bProgress.set(0, { duration: 0 });
    pointer = bExpected;
  };
  const handleSwipe = (event: CustomEvent<{
    direction: 'top' | 'right' | 'bottom' | 'left';
    target: EventTarget;
  }>) => {
    console.log(event.detail)
    const { direction } = event.detail;
    if (direction === 'left') {
      goForward();
    } else if (direction === 'right') {
      goBackward();
    }
  }
</script>

<div>pointer: {pointer}</div>
<div
  class="slider"
  use:swipe
  on:swipe={handleSwipe}
>
  <div class="slide previous-slide" style:opacity={$bProgress}>{previous}</div>
  <div
    class="slide current-slide"
    style:transform={`
      rotateY(${-ANGLE * $bProgress}deg)
      translateX(${SHIFT * $bProgress}%)
    `}
    style:opacity={1 - $bProgress}
  >
    {current}
  </div>
  <div
    class="slide next-slide"
    style:transform={`
      rotateY(${-ANGLE + ANGLE * $fProgress}deg)
      translateX(${SHIFT - SHIFT * $fProgress}%)
    `}
    style:opacity={$fProgress}
  >
    {next}
  </div>
</div>
<div class="controls">
  <button on:click={goBackward}>previous</button>
  <button on:click={goForward}>next</button>
</div>

<style>
  .slider {
    height: 400px;
    width: 200px;
    perspective: 2000px;
    position: relative;
  }
  .slide {
    height: 400px;
    background-color: #60a87f;
  }
  .next-slide {
    transform-origin: 50% 50%;
    border: 1px solid red;
  }
  .current-slide {
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
  
  .controls {
    display: flex;
    margin-top: 1rem;
    justify-content: flex-end;
    gap: 1rem;
    position: relative;
    z-index: 100;
  }
  .controls button {
    font-size: 3rem;
  }
</style>
