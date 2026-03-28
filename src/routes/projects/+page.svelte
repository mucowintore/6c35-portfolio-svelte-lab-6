<script>
  import projects from '$lib/projects.json';
  import Project from '$lib/Project.svelte';
  import ProjectNarrative from '$lib/ProjectNarrative.svelte';

  let years = projects.map((proj) => proj.year);
  let range = Math.max(...years) - Math.min(...years);
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<main>
  <h1>{projects.length} Projects over {range} Years</h1>
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
  .outro {
    margin-bottom: 1.5rem;
  }
</style>
