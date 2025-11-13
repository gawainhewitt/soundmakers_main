<script>
  import { onMount } from 'svelte';
  import ResponsiveContainer from './lib/ResponsiveContainer.svelte';
  import GridContainer from './lib/GridContainer.svelte';
  import SplashScreen from './lib/SplashScreen.svelte';
  import { AudioEngine } from './lib/AudioEngine.js';

  let showSplash = true;
  let audioEngine = null;
  let audioInitialized = false;

  onMount(() => {
    // Create audio engine immediately (but don't initialize yet)
    audioEngine = new AudioEngine();
  });

  async function handleSplashClick() {
    document.body.style.setProperty('background-color', '#FBAC2E', 'important');
    // Initialize audio context on user interaction (required for iOS)
    if (audioEngine && !audioInitialized) {
      await audioEngine.init();
      audioInitialized = true;
      console.log('Audio initialized from splash screen');
    }
    
    // Hide splash screen
    showSplash = false;
  }
</script>

{#if showSplash}
  <SplashScreen 
    title="Circles"
    instructions="To play: touch or click screen or use ZXCVBNM,. keys on a keyboard"
    footerNote="On Apple devices, turn off silent mode"
    on:click={handleSplashClick}
  />
{/if}

<main>
  <ResponsiveContainer>
    <GridContainer {audioEngine} {audioInitialized} />
  </ResponsiveContainer>
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    width: 100%;
    max-width: 100%;
    box-sizing: border-box;
    margin: 0;
  }


  @media (orientation: landscape) and (max-height: 500px) {
    main {
        padding: 0.5em;
      }
    }
</style>
