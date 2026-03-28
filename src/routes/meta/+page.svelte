<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import BarHorizontal from '$lib/BarHorizontal.svelte';

  let locData = [];
  let locByLanguage = [];
  let loading = true;
  let error = null;

  onMount(async () => {
    try {
      locData = await d3.csv(`${base}/loc.csv`, (row) => ({
        ...row,
        line: Number(row.line),
        length: Number(row.length),
        depth: Number(row.depth),
      }));

      locByLanguage = d3.rollups(
        locData,
        (v) => v.length,
        (d) => d.type
      )
        .map(([label, value]) => ({ label: String(label).toUpperCase(), value }))
        .sort((a, b) => b.value - a.value);
    } catch (err) {
      error = err;
    }

    loading = false;
  });
</script>

<svelte:head>
  <title>Meta Analysis</title>
</svelte:head>

<main>
  <h1>Meta Analysis</h1>
  <p>
    This page tracks statistics about the codebase itself, including lines of code by language.
  </p>

  {#if loading}
    <p>Loading code metrics...</p>
  {:else if error}
    <p>Could not load code metrics: {error.message}</p>
  {:else}
    <BarHorizontal data={locByLanguage} />
  {/if}
</main>
