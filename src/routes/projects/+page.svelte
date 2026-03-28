<script>
  import * as d3 from 'd3';
import projects from '$lib/projects.json';
import Project from '$lib/Project.svelte';
import ProjectNarrative from '$lib/ProjectNarrative.svelte';
import Bar from '$lib/Bar.svelte';

  let barData = [];

  let years = projects.map((proj) => proj.year);
  let range = Math.max(...years) - Math.min(...years);

  $: barData = d3.rollups(projects, (v) => v.length, (d) => d.year)
    .map(([year, count]) => ({ label: String(year), value: count }))
    .sort((a, b) => Number(a.label) - Number(b.label));
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

  .outro {
    margin-bottom: 1.5rem;
  }
</style>
