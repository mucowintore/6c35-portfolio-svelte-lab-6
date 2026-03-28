<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
import projects from '$lib/projects.json';
import Project from '$lib/Project.svelte';
import ProjectNarrative from '$lib/ProjectNarrative.svelte';
import Bar from '$lib/Bar.svelte';

  let rawData = [];
  let wrangled = [];
  let barData = [];

  let years = projects.map((proj) => proj.year);
  let range = Math.max(...years) - Math.min(...years);

  $: barData = d3.rollups(projects, (v) => v.length, (d) => d.year)
    .map(([year, count]) => ({ label: String(year), value: count }))
    .sort((a, b) => Number(a.label) - Number(b.label));

  onMount(async () => {
    rawData = await d3.json(`${base}/lab6_example.json`);
    wrangled = d3.rollups(
      rawData,
      (v) => d3.sum(v, (d) => d.lines),
      (d) => d.language
    );
  });
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<main>
  <h1>{projects.length} Projects over {range} Years</h1>
  <section class="demo-chart">
    <h2>Projects by Year</h2>
    <Bar data={barData} />
  </section>
  <p>
    Scroll down to see a timeline of my projects and how each one shaped the way I build software.
  </p>
  <section class="wrangling-output">
    <h2>Data Wrangling Result</h2>
    <pre>{JSON.stringify(wrangled, null, 2)}</pre>
  </section>
  <ProjectNarrative />
  <p class="outro">
    Thanks for reading through the story. You can keep exploring all projects in the gallery below.
  </p>
  <div class="projects">
    {#each projects as p}
      <Project data={p} />
    {/each}
  </div>
</main>

<style>
  .demo-chart {
    margin: 1rem 0 2rem;
  }

  .wrangling-output {
    margin: 1rem 0 2rem;
  }

  .wrangling-output pre {
    padding: 0.8rem 1rem;
    border: 1px solid oklch(72% 0.04 230 / 60%);
    border-radius: 0.5rem;
    background: color-mix(in oklch, canvas, var(--color-accent) 4%);
    overflow-x: auto;
  }

  .outro {
    margin-bottom: 1.5rem;
  }
</style>
