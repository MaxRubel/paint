<script>
  import { onMount } from 'svelte';
  import {event_state_store} from '../../stores/eventState'
  import { createEventDispatcher } from 'svelte';
  import {deleteTextBox} from '../../stores/migmaStore'
  export let data;
  export let updateTextBox;

  let { x, y, id } = data;
  let textareaElement;
  let isDragging = false;
  let dragStartX, dragStartY;
  let isEditting = true

  const dispatch = createEventDispatcher();

  onMount(()=>{
    event_state_store.set("typing")
  })

  function handleInput(e) {
    updateTextBox(id, { ...data, text: e.target.value });
    event_state_store.set("typing")
  }

  function handleKeydown(e) {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      textareaElement.blur();
    }
  }

  function handleDragStart(e){
    !isEditting ? isDragging = true : isDragging = false
    dragStartX = e.clientX;
    dragStartY = e.clientY;
  }

  function handleDragEnd(event) {
  if (isDragging) {
    const rect = textareaElement.getBoundingClientRect();
    x = event.clientX - (rect.width / 2) ;
    y = event.clientY - (rect.height / 2) + 14;
    isDragging = false;
  }
}

  function handleBlur(){
    isEditting = false;       
    event_state_store.set("arrow")
    dispatch('select', null);
    if (textareaElement.value === ""){
      deleteTextBox(id)
    }
  }
  
  function focusAndSelectAll(node) {
    node.focus();
    node.setSelectionRange(0, 0);
  }

  function handleFocus(){
    dispatch('select', `textbox&${id}`);
  }

  let cursorStyle = 'grab';
  $:cursorStyle = isEditting ? 'text' : 'grab';

  </script>
  
  <!-- svelte-ignore a11y-mouse-events-have-key-events -->
  <textarea
    bind:this={textareaElement}
    class="text-box"
    class:non-selectable={!isEditting}
    id="textbox-{id}"
    style="top: {y-27}px; left: {x}px; cursor: {cursorStyle}"
    on:input={handleInput}
    on:keydown={handleKeydown}
    on:mousedown={handleDragStart}
    on:dragend={handleDragEnd}
    on:dblclick={()=>{isEditting = true}}
    on:blur={handleBlur}
    on:focus={handleFocus}
  
    use:focusAndSelectAll
    readonly={!isEditting}
    draggable={isEditting ? "false" : "true"}
  >{data.text}</textarea>
  
  <style>
    .text-box {
      position: absolute;
      background-color: transparent;
      border: none;
      padding: 5px;
      min-width: 100px;
      min-height: 20px;
      color: lightgray;
      font-size: 18pt;
      z-index: 200;
      border-radius: 12px;
      transition: box-shadow 0.3s ease;
    }

  .non-selectable {
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }
  
  </style>