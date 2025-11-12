
<script>
  import ResponsiveContainer from './lib/ResponsiveContainer.svelte';
  import GridContainer from './lib/GridContainer.svelte';

</script>

<main>
  <ResponsiveContainer>
    <GridContainer />
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


