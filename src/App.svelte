<script>
// @ts-nocheck

  import { onDestroy, onMount } from 'svelte';
  import { getSvgPathFromStroke } from '../utils/getSvgPathFromStroke'
  import { getStroke } from 'perfect-freehand'
  import { v4 as uuidv4 } from 'uuid';
  import TextBox from './components/Text_Box.svelte';
  import SmallCheck from './graphics/SmallCheck.svelte';
  import { createNewTextBox, textBoxesStore, updateTextBox, clearAllTextBoxes, deleteTextBox } from '../stores/migmaStore';
  import Marker from './graphics/Marker.svelte';
  import TextIcon from './graphics/TextIcon.svelte';
  import {event_state_store} from '../stores/eventState'
  import CursorPointer from './graphics/CursorPointer.svelte'
  

  let points = []
  let paths = []
  let moves = []
  let mode = "dark"
  let pathData
  let catSmootch = false

  let start = null
  let end = null
  let canvas
  let ctx
  let size = 7
  let textBoxes
  let cursor
  let event_state
  let selected = null

  const unsubcribe = textBoxesStore.subscribe((value)=>{
    textBoxes = value
  })

  const unsubcribe2 = event_state_store.subscribe((value)=>{
    event_state = value
  })

  $: {
    switch(event_state){
      case "createTextBox":
        cursor = "text";
        break;
      case "typing":
        cursor = "test";
        break;
      case "drawing":
        cursor = "crosshair";
        break;
      case "arrow":
        cursor = "default";
        break;
    }
  }

  let user_mode = "drawing"
  clearAllTextBoxes
  const handleClick = (e) => {
    switch(event_state){
      case "createTextBox":
        createNewTextBox(e)
        break
    }
  }

  function handle_delete(){
    if(event_state === "typing")return
    deleteTextBox(selected)
  }

  function handleKeyup(e){
    if(event_state==="typing")return
    switch (e.key){
      case "p":
        handle_drawing_mode();
      break;
      case "t":
        handle_textbox_mode();
        break;
      case "a":
        handle_arrow_mode();
        break
      case "Delete":
        handle_delete()
    }
  }

  onMount(() => {
    ctx = canvas.getContext('2d')
    resizeCanvas()
    window.addEventListener('resize', resizeCanvas)
    window.addEventListener('keyup', handleKeyup)
    canvas.addEventListener("click", handleClick)
  })

  onDestroy(() => {
    unsubcribe();
    unsubcribe2();
    if (timeout) {
      clearTimeout(timeout)
    }
    window.removeEventListener('resize', resizeCanvas)
    window.removeEventListener('keyup', handleKeyup)
    canvas.removeEventListener("click", handleClick)
  })
  
  function resizeCanvas() {
    canvas.width = window.innerWidth
    canvas.height = window.innerHeight
  }

  function redrawCanvas() {
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    paths.forEach(path => {
      const canvasPath = new Path2D(path)
      ctx.fillStyle = mode === 'light' ? 'black' : 'rgb(143, 143, 143)'
      ctx.fill(canvasPath)
    })
  }

  function textbox_write(id, e){
    const {value} = e.target
    textBoxes[id].text = value
  }

  function handleClear() {
    paths = []
    points = []
    moves = []
    textBoxes = {}
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    clearAllTextBoxes()
  }

  function handleUndo() {
    if (moves.length === 0) return;

    const lastMove = moves[moves.length - 1];
    const amount = (lastMove.end - lastMove.start);
    const index = lastMove.start

    paths.splice(index - 1, amount + 2);
    paths = paths
    moves.splice(moves.length - 1, 1)
    moves = moves
    redrawCanvas()
  }

  function handlePointerDown(e) {
    if(event_state !== "drawing")return
    start = paths.length + 1
    e.target.setPointerCapture(e.pointerId)
    points = [[e.clientX-3, e.clientY-3, e.pressure]]
  }

  function handlePointerUp() {
    end = paths.length - 1
    if (end >= start) {
      moves = [...moves, { start, end }]
    }
  }

  function handlePointerMove(e) {
    if(event_state !== "drawing")return
    if (e.buttons !== 1) return
    points = [...points, [e.clientX-3, e.clientY-3, e.pressure]]
    const stroke = getStroke(points, {
      size: size,
      thinning: 0.5,
      smoothing: 1,
      streamline: 0.6,
    })
    pathData = getSvgPathFromStroke(stroke)
    paths = [...paths, pathData]

    const canvasPath = new Path2D(pathData)
    ctx.fillStyle = mode === 'light' ? 'black' : 'rgb(143, 143, 143)'
    ctx.fill(canvasPath)
  }

  function handleDark() {
    mode = mode === "light" ? "dark" : "light";
    redrawCanvas()
  }

  let timeout;

  function handleSmootches() {
    if (timeout) {
      clearTimeout(timeout)
    }
    catSmootch = true
    timeout = setTimeout(() => {
      catSmootch = false
    }, 2600)
  }

  function handle_textbox_mode(){
    if(event_state === "createTextBox"){
      event_state_store.set("arrow")
    } else {
      event_state_store.set("createTextBox")
    }
  }

  function handle_drawing_mode() {
  if (event_state === "drawing") {
    event_state_store.set("arrow")
  } else {
    event_state_store.set("drawing")
  }
}

  function handle_arrow_mode(){
    event_state_store.set("arrow")
  }

  function handleSelect(e){
    if (e.detail){
      const [_, id] = e.detail.split('&')
      selected = id
    } else {
      selected = null
    }
  }

</script>

<main>
  <div class="tool-bar">
    <div class="top-row">
    <button on:click={handleUndo}>Undo</button>
    <button on:click={handleClear}>Clear</button>
    <button on:click={handleDark}>
      {mode === "light" ? "Dark" : "Light"}
    </button>
    <button on:click={handleSmootches}>Smootches</button>
  </div>
  <div class="second-row">
    <button on:click={handle_arrow_mode} style="background-color: {event_state==="arrow"? '#9096ff': ""}">
      <CursorPointer/>
    </button>
    <button on:click={handle_drawing_mode} style="background-color: {event_state==="drawing"? '#9096ff': ""}">
      <Marker/> 
    </button>
    <button on:click={handle_textbox_mode} >
      <TextIcon/>
      {#if event_state === "createTextBox" || event_state==="typing"}<SmallCheck/>{/if}
    </button>
  </div>
  </div>
  <div class="canvas-container">
    {#if catSmootch}
    smooch
      <img src="/smooch.webp" alt="" class="full-size image">
    {:else}
      {#each Object.values(textBoxes) as textBox (textBox.id)}
        <TextBox data={textBox} {updateTextBox} on:select={handleSelect}/>
      {/each}
      <canvas
        style="cursor: {cursor}"
        class="full-size"
        class:light={mode === 'light'}
        class:dark={mode === 'dark'}
        bind:this={canvas}
        on:pointerdown={handlePointerDown}
        on:pointerup={handlePointerUp}
        on:pointermove={handlePointerMove}
      >
    </canvas>
    {/if}
  </div>
</main>

<style>
  .canvas-container {
    position: relative;
    height: 100vh;
    width: 100vw;
  }

  .image {
    filter: brightness(70%)
  }

  .tool-bar {
    position: fixed;
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: center;
    gap: 8px;
    top: 9px;
    left: 0;
    width: 100%;
    height: 100px;
    z-index: 1000;
    box-sizing: border-box;
    background-color: rgba(0, 0, 0, 0.288);
  }

  .full-size {
    position: absolute;
    height: 100vh;
    width: 100vw;
    top: 0px;
    left: 0px;
  }

  canvas.light {
    background-color: lightblue;
  }

  canvas.dark {
    background-color: rgb(42, 50, 53);
  }

</style>