<script>
  export let data = {};
</script>

<article>
  <h2>{data.title}</h2>
  <img src={data.image} alt="" />
  <p class="year">{data.year}</p>
  <p>{data.description}</p>
</article>

<style>
  article {
    display: grid;
    grid-template-rows: subgrid;
    grid-row: span 3;
    gap: 0.5em;
    padding: 0.75em;
    border: 1px solid #ddd;
    border-radius: 0.5em;
  }

  h2 {
    margin: 0;
  }

  .year {
    margin: 0;
    font-size: 0.9rem;
    font-weight: 600;
    color: color-mix(in oklch, currentColor, canvas 35%);
  }

  img {
    max-width: 100%;
    height: auto;
  }
</style>
