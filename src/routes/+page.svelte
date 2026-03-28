<script>
  import { onMount } from 'svelte';
  import { base } from '$app/paths';
  import projects from '$lib/projects.json';
  import reading from '$lib/reading.json';
  import Project from '$lib/Project.svelte';
  import ReadingItem from '$lib/ReadingItem.svelte';

  let githubData = null;
  let loading = true;
  let error = null;

  async function retrieveGithubData() {
    try {
      const response = await fetch('https://api.github.com/users/mucowintore');

      if (!response.ok) {
        throw new Error(`GitHub API request failed (${response.status})`);
      }

      githubData = await response.json();
    } catch (err) {
      error = err;
    }

    loading = false;
  }

  onMount(retrieveGithubData);
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

  {#if loading}
    <p>Loading GitHub stats...</p>
  {:else if error}
    <p>Something went wrong loading GitHub stats: {error.message}</p>
  {:else}
    <section class="github-stats">
      <h2>My GitHub Stats</h2>
      <dl>
        <dt>Followers</dt>
        <dd>{githubData.followers}</dd>

        <dt>Following</dt>
        <dd>{githubData.following}</dd>

        <dt>Public Repos</dt>
        <dd>{githubData.public_repos}</dd>

        <dt>Public Gists</dt>
        <dd>{githubData.public_gists}</dd>
      </dl>
    </section>
  {/if}

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

<style>
  .github-stats {
    margin-top: 1.5rem;
    padding: 1rem;
    border: 1px solid oklch(75% 0.06 240 / 60%);
    border-radius: 0.5rem;
    background: color-mix(in oklch, canvas, var(--color-accent) 5%);
  }

  .github-stats h2 {
    margin-top: 0;
  }

  .github-stats dl {
    margin: 0;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(8rem, 1fr));
    gap: 0.35rem 1rem;
  }

  .github-stats dt {
    font-size: 0.8rem;
    text-transform: uppercase;
    letter-spacing: 0.04em;
    opacity: 0.75;
  }

  .github-stats dd {
    margin: 0 0 0.65rem;
    font-size: 1.5rem;
    font-weight: 700;
    line-height: 1.1;
  }
</style>
