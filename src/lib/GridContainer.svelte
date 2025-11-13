<script>
  import { onMount, onDestroy } from 'svelte';
  import Circle from './Circle.svelte';
  
  export let audioEngine;
  
  let circles = Array.from({ length: 9 }, (_, i) => i);
  let orientation = 'portrait';
  let cleanupInterval;
  
  // C Major scale
  let scale = ['C4', 'D4', 'E4', 'F4', 'G4', 'A4', 'B4', 'C5', 'D5'];
  
  // Track which circles are pressed
  let circleStates = {};
  circles.forEach((_, i) => {
    circleStates[scale[i]] = false;
  });
  
  onMount(() => {
    // Periodic cleanup - check for orphaned oscillators
    cleanupInterval = setInterval(() => {
      if (audioEngine) {
        // First do smart cleanup based on circle states
        smartCleanup();
        // Then do nuclear cleanup of any orphaned oscillators (passing circle states)
        audioEngine.cleanupOrphanedOscillators(circleStates);
      }
    }, 1000); // Check every 1 second
    
    // Add global panic button
    window.addEventListener('keydown', handleKeydown);
    
    // Stop all notes when page loses focus or visibility
    document.addEventListener('visibilitychange', handleVisibilityChange);
    window.addEventListener('blur', handleWindowBlur);
  });
  
  onDestroy(() => {
    if (cleanupInterval) {
      clearInterval(cleanupInterval);
    }
    window.removeEventListener('keydown', handleKeydown);
    document.removeEventListener('visibilitychange', handleVisibilityChange);
    window.removeEventListener('blur', handleWindowBlur);
    
    // Panic on unmount
    if (audioEngine) {
      audioEngine.panic();
    }
  });
  
  function handleVisibilityChange() {
    if (document.hidden && audioEngine) {
      console.log('Page hidden - stopping all notes');
      audioEngine.panic();
      // Reset all circle states
      Object.keys(circleStates).forEach(note => {
        circleStates[note] = false;
      });
    }
  }
  
  function handleWindowBlur() {
    if (audioEngine) {
      console.log('Window blur - stopping all notes');
      audioEngine.panic();
      // Reset all circle states
      Object.keys(circleStates).forEach(note => {
        circleStates[note] = false;
      });
    }
  }
  
  function smartCleanup() {
    // Get all currently playing notes
    const playingNotes = Array.from(audioEngine.activeOscillators.keys());
    
    // Stop any notes that are playing but their circle is not pressed
    playingNotes.forEach(note => {
      if (!circleStates[note]) {
        console.warn('Cleaning up stuck note:', note);
        audioEngine.stopNote(note);
      }
    });
  }
  
  function handleKeydown(e) {
    // Press 'P' key to panic (stop all notes)
    if (e.key === 'p' || e.key === 'P') {
      if (audioEngine) {
        audioEngine.panic();
        // Reset all circle states
        Object.keys(circleStates).forEach(note => {
          circleStates[note] = false;
        });
      }
    }
  }
  
  async function initAudio() {
    // Audio should already be initialized from splash screen
    // But we can resume if suspended
    if (audioEngine && audioEngine.audioContext && audioEngine.audioContext.state === 'suspended') {
      await audioEngine.audioContext.resume();
      console.log('Audio context resumed:', audioEngine.audioContext.state);
    }
  }
  
  function updateOrientation() {
    orientation = window.innerHeight > window.innerWidth ? 'portrait' : 'landscape';
  }
  
  $: {
    if (typeof window !== 'undefined') {
      updateOrientation();
    }
  }
  
  async function handlePress(event) {
    await initAudio();
    circleStates[event.detail.note] = true;
    console.log('Circle pressed:', event.detail.note);
  }
  
  function handleRelease(event) {
    circleStates[event.detail.note] = false;
    console.log('Circle released:', event.detail.note);
  }
</script>

<svelte:window on:resize={updateOrientation} />

<div class="container {orientation}">
  {#each circles as index}
    <Circle 
      {index}
      {orientation}
      {audioEngine}
      note={scale[index]}
      on:press={handlePress}
      on:release={handleRelease}
    />
  {/each}
</div>

<style>
  .container {
    display: grid;
    gap: 2vh;
    padding: 2vh;
    height: 100%;
    width: 100%;
    place-items: center;
    place-content: center;
  }
  
  .container.portrait {
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
  }
  
  .container.landscape {
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
  }
</style>