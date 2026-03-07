<script>
  import { base } from '$app/paths';
  import projects from '$lib/projects.json';
  import reading from '$lib/reading.json';
  import Project from '$lib/Project.svelte';
  import ReadingItem from '$lib/ReadingItem.svelte';
</script>

<svelte:head>
  <title>Muco: Personal site and portfolio</title>
</svelte:head>

<main class="home-page">
  <h1>Muco</h1>
  <section>
    <p>I am Muco and this is my portfolio website. Stay tuned for awesome content coming up!</p>
    <img src="{base}/images/profile.jpg" alt="Portrait of Muco" />
  </section>

  <section>
    <h2>Latest Projects</h2>
    <div class="projects highlights">
      {#each projects.slice(0, 3) as p}
        <Project data={p} />
      {/each}
    </div>
  </section>

  <section>
    <h2>What I’m Reading</h2>
    <div class="projects reading-list">
      {#each reading as item}
        <ReadingItem data={item} />
      {/each}
    </div>
  </section>
</main>