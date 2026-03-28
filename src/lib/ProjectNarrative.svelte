<script>
  import Scrolly from 'svelte-scrolly';
  import projects from '$lib/projects.json';

  let scrollyProgress = 0;
  let sorted_projects = [...projects].sort((a, b) => a.year - b.year);
  let progressPerProject = 100 / sorted_projects.length;
  $: activeProjectIdx = Math.min(
    sorted_projects.length - 1,
    Math.floor(scrollyProgress / progressPerProject)
  );
  $: activeProject = sorted_projects[activeProjectIdx];
</script>

<div class="scrolly-wrapper">
  <Scrolly bind:progress={scrollyProgress}>
    {#each sorted_projects as project}
      <section class="step">
        <div class="step-content">
          <h3>{project.year} · {project.title}</h3>
          <p>{project.story}</p>
        </div>
      </section>
    {/each}

    <svelte:fragment slot="viz">
      <div class="project-detail">
        <h3>{activeProject.year} · {activeProject.title}</h3>
        <img
          src={activeProject.image}
          alt={`Preview image for ${activeProject.title}`}
        />
        <p>{activeProject.description}</p>
      </div>
    </svelte:fragment>
  </Scrolly>
</div>

<style>
  .scrolly-wrapper {
    margin: 1.5rem 0 2rem;
    width: min(1000ch, 60vw);
    position: relative;
    left: 50%;
    transform: translateX(-50%);
  }

  .step {
    min-height: 80vh;
    padding: 2rem;
    display: grid;
    align-items: center;
  }

  .step-content {
    padding: 1.5rem 2rem;
    border-left: 4px solid var(--color-accent);
    background: color-mix(in oklch, canvas, var(--color-accent) 4%);
    border-radius: 0 0.35rem 0.35rem 0;
  }

  .step-content h3 {
    margin: 0 0 0.4rem;
  }

  .step-content p {
    margin: 0;
  }

  .project-detail {
    padding: 2rem;
    width: 100%;
    border: 1px solid oklch(70% 0.06 240 / 50%);
    border-radius: 0.5rem;
    background: color-mix(in oklch, canvas, var(--color-accent) 8%);
  }

  .project-detail img {
    width: 100%;
    height: auto;
    border-radius: 0.35rem;
  }

  @media (max-width: 900px) {
    .scrolly-wrapper {
      width: 92vw;
      left: 0;
      transform: none;
    }
  }
</style>
