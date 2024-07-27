<script>
  import { onDestroy } from 'svelte';
  import {getSvgPathFromStroke} from '../utils/getSvgPathFromStroke'
  import { getStroke } from 'perfect-freehand'

  let points = []
  let pathData
  let paths = []
  let moves = []
  let mode = "dark"
  let catSmootch = false

  let start = null
  let end = null

  let size 

  function handleClear(){
      paths = []
      points= []
      moves = []
  }

  function handleUndo() {
      if (moves.length === 0) return;

      const lastMove = moves[moves.length - 1];
      const amount = (lastMove.end - lastMove.start);

      const index = lastMove.start
      paths.splice(index -1, amount +2);
      paths = paths
      moves.splice(moves.length-1, 1)
      moves = moves
  }

  function handlePointerDown(e) {
      start = paths.length+1
      e.target.setPointerCapture(e.pointerId)
      points = [[e.clientX-6, e.clientY-10, e.pressure]]
  }

  function handlePointerUp(){
      end = paths.length - 1
      if (end >= start){
          moves = [...moves, {start, end}]
      }
  }

  function handlePointerMove(e) {
      if (e.buttons !== 1) return
      points = [...points, [e.clientX-6, e.clientY-10, e.pressure]]
      const stroke = getStroke(points, {
          size: 7,
          thinning: 0.5,
          smoothing: 1,
          streamline: 0.6,
      })
      pathData = getSvgPathFromStroke(stroke)
      const newPath = pathData
      paths = [...paths, newPath]
  }

  function handleDark(){
      mode = mode === "light" ? "dark" : "light";
  }

  let timeout;

  function handleSmootches(){
    catSmootch = true
    timeout = setTimeout(()=>{
      catSmootch = false
    }, 2600)
  }

  onDestroy(()=>{
    if(timeout){
      clearTimeout(timeout)
    }
  })
</script>

<main>
  <div class="tool-bar">
      <button on:click={handleUndo}>Undo</button>
      <button on:click={handleClear}>Clear</button>
      <button on:click={handleDark}>
          {mode === "light" ? "Dark" : "Light"}
      </button>
      <button on:click={handleSmootches}>Smootches</button>
  </div>
  <div class="canvas-container">
    {#if catSmootch}
    <img src="/smooch.webp" alt="" class="full-size image">
    {:else}
      <svg
          class={`full-size ${mode}`}
          on:pointerdown={handlePointerDown}
          on:pointerup={handlePointerUp}
          on:touchstart={handlePointerDown}
          on:touchmove={handlePointerMove}
          on:pointermove={handlePointerMove}
          style='touch_action: none'
      >
          {#each paths as path }
              <path d={path} />
          {/each}
      </svg>
      {/if}
      <input type="number" bind:this={size}>
  </div>
</main>

<style>
  .canvas-container{
      position: relative;
      height: 100vh;
      width: 100vw;
  }

  .image{
    filter: brightness(70%)
  }

  .tool-bar{
      position: fixed;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 10px;
      top: 6;
      left: 0;
      width: 100%;
      height: 50px;
      z-index: 1000;
      box-sizing: border-box;
      background-color: rgba(0, 0, 0, 0.288);
  }

  .full-size{
      position: absolute;
      height: 100vh;
      width: 100vw;
      top: 0px;
      left: 0px;
      stroke: 4px;
      cursor: crosshair;
  }

  svg.light {
      background-color: lightblue;
      fill: black;
  }

  svg.dark {
      background-color: rgb(42, 50, 53);
      fill: rgb(143, 143, 143);
  }

  input{
      height: 44px;
  }
</style>