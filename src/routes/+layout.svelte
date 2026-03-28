<script>
  import { base } from '$app/paths';
  import { page } from '$app/stores';

  let pages = [
    { url: '/', title: 'Home' },
    { url: '/projects', title: 'Projects' },
    { url: '/meta', title: 'Meta' },
    { url: '/contact', title: 'Contact' },
    { url: '/resume', title: 'Resume' },
    { url: 'https://github.com/mucowintore', title: 'GitHub' },
  ];

  let root = globalThis.document?.documentElement;
  let localStorage = globalThis.localStorage ?? {};
  let colorScheme = 'light dark';

  if (localStorage.colorScheme) {
    colorScheme = localStorage.colorScheme;
  }

  $: root?.style.setProperty('color-scheme', colorScheme);
  $: localStorage.colorScheme = colorScheme;
</script>

<label class="color-scheme-switch">
  Theme:
  <select bind:value={colorScheme}>
    <option value="light dark">Automatic</option>
    <option value="light">Light</option>
    <option value="dark">Dark</option>
  </select>
</label>

<nav aria-label="Primary">
  {#each pages as p}
    <a
      href={p.url.startsWith('http') ? p.url : base + p.url}
      class:current={p.url === '/'
        ? $page.url.pathname === base || $page.url.pathname === (base + '/')
        : $page.url.pathname.startsWith(base + p.url)}
      target={p.url.startsWith('http') ? '_blank' : null}
      rel={p.url.startsWith('http') ? 'noopener noreferrer' : null}
    >
      {p.title}
    </a>
  {/each}
</nav>

<slot />

<style>
  nav {
    display: flex;
    margin-bottom: 1rem;
    --color-nav-border: oklch(50% 10% 200 / 40%);
    border-bottom: 2px solid var(--color-nav-border);
  }

  nav a {
    flex: 1;
    text-align: center;
    padding: 0.5em;
    text-decoration: none;
    color: inherit;
    border-bottom: 2px solid transparent;
  }

  nav a.current {
    color: var(--color-accent);
    border-bottom-width: 4px;
    border-bottom-color: var(--color-nav-border);
  }

  nav a:hover {
    color: var(--color-accent);
    background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
    border-bottom-color: var(--color-accent);
  }

  .color-scheme-switch {
    position: absolute;
    top: 1rem;
    right: 1rem;
    display: inline-flex;
    align-items: center;
    gap: 0.25rem;
    font-size: 80%;
  }
</style>
