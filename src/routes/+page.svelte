<script lang="ts">
    import { browser } from "$app/environment";
  import { goto } from "$app/navigation";
  import { values } from "$lib/asker";
  import Estimate from "$lib/components/Estimate.svelte";
  import Question from "$lib/components/Question.svelte";
  import { questions } from "$lib/questions";
  import { onDestroy, onMount, tick } from "svelte";
  import { fly } from "svelte/transition";
  
  const duration = 500;

  let question = 0;
  let animate = false;
  let prev = -1;

  function changeQuestion(offset: number) {
    // Check condition
    if (offset > 0 && (question + offset) < questions.length && questions[question + offset].condition) {
      let cond = questions[question + offset].condition!;
      if (!cond.check($values[question + offset + cond.offset])) { // Call check function
        offset += 1;
      }
    }

    // Check if within bounds
    if (question + offset >= questions.length) {
      goto("/results");
      return;
    }

    prev = question;
    animate = true;
    tick().then(() => {animate = false;});
    question += offset;
  } 

  onMount(() => {
    if (browser) {
      document.body.style.overflow = "hidden";
    }
  })

  onDestroy(() => {
    if (browser) {
      document.body.style.overflow = "auto";
    }
  })
</script>

<svelte:head>
  <title>Waterly</title>
</svelte:head>

<Estimate></Estimate>

<div class="body">
  {#if animate}
    <div out:fly={{x: ((question - prev) > 0 ? -1 : 1) * window.innerWidth, duration }}>
      <Question id={prev}></Question>
    </div>
  {/if}

  {#if !animate}
    <div in:fly={{x: ((question - prev) > 0 ? 1 : -1) * window.innerWidth, duration }}>
      <Question id={question}></Question>
    </div>
  {/if}
</div>

<div class="switcher input-group input-group-lg">
  <button class="btn btn-outline-primary left-btn" on:click={() => {changeQuestion(-1)}} disabled={question == 0}><i class="bi bi-arrow-left"></i></button>
  <input readonly value={"Question " + (question + 1) + " of " + questions.length} class="form-control input"/>
  <button class="btn btn-outline-primary right-btn" on:click={() => {changeQuestion(1)}}><i class="bi bi-arrow-right"></i></button>
</div>

<style>
  .switcher {
    position: fixed;
    bottom: 0;
    left: 25vw;
    right: 25vw;
  }

  .left-btn {
    border-bottom-left-radius: 0;
  }

  .right-btn {
    border-bottom-right-radius: 0;
  }

  .input {
    flex: 0.5 1 auto;
  }
  
  .body > * {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    overflow: scroll;
    pointer-events: none;
  }

  .body {
    pointer-events: none;
  }

  :global(body) {
    overflow: hidden;
  }
  /* width */
/* set the width of the scrollbar */

::-webkit-scrollbar {
  width: 10px;
}


/* Handle */
/* set the color of the scrollbar and the radius of its corners */

::-webkit-scrollbar-thumb {
  background: rgb(150, 150, 150);
  border-bottom-left-radius: 5px;
  border-top-left-radius: 5px;
  border-bottom-right-radius: 5px;
  border-top-right-radius: 5px;
}


/* Handle on hover */
/* set the color of the scrollbar when hovered over */

::-webkit-scrollbar-thumb:hover {
  background: rgb(131, 131, 131);
}


/* Handle on active */
/* set the color of the scrollbar when clicked */

::-webkit-scrollbar-thumb:active {
  background: rgb(104, 104, 104);
}
</style>