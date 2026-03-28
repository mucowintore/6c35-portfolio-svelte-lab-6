<script>
  import * as d3 from 'd3';

  export let data = [];

  let width = 700;
  let height = 360;
  let margin = { top: 40, right: 170, bottom: 70, left: 110 };
  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  let xAxis;
  let yAxis;

  $: xScale = d3.scaleLinear()
    .domain([0, d3.max(data, (d) => d.value) || 1])
    .range([0, innerWidth]);

  $: yScale = d3.scaleBand()
    .domain(data.map((d) => d.label))
    .range([0, innerHeight])
    .padding(0.2);

  $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    .domain(data.map((d) => d.label));

  $: maxBar = d3.greatest(data, (d) => d.value);

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }
</script>

<div class="container">
  <svg viewBox="0 0 {width} {height}">
    <text
      x={margin.left + innerWidth / 2}
      y={margin.top / 2}
      text-anchor="middle"
      class="chart-title"
    >
      Lines of Code by Language
    </text>

    <g transform="translate({margin.left}, {margin.top + innerHeight})" bind:this={xAxis} />
    <g transform="translate({margin.left}, {margin.top})" bind:this={yAxis} />

    <g transform="translate({margin.left}, {margin.top})">
      {#each data as d}
        <rect
          x={0}
          y={yScale(d.label)}
          width={xScale(d.value)}
          height={yScale.bandwidth()}
          fill={colorScale(d.label)}
        />
      {/each}

      {#if maxBar}
        <rect
          x={0}
          y={yScale(maxBar.label)}
          width={xScale(maxBar.value)}
          height={yScale.bandwidth()}
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        />

        <line
          x1={xScale(maxBar.value)}
          y1={yScale(maxBar.label) + yScale.bandwidth() / 2}
          x2={xScale(maxBar.value) + 30}
          y2={yScale(maxBar.label) + yScale.bandwidth() / 2}
          stroke="currentColor"
          stroke-width="1"
        />

        <text
          x={xScale(maxBar.value) + 35}
          y={yScale(maxBar.label) + yScale.bandwidth() / 2}
          dominant-baseline="middle"
          class="annotation"
        >
          Most LOC
        </text>
      {/if}

      <text
        x={innerWidth / 2}
        y={innerHeight + margin.bottom - 18}
        text-anchor="middle"
        class="axis-label"
      >
        Lines of Code
      </text>

      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 16}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label"
      >
        Language
      </text>
    </g>
  </svg>

  <ul class="legend">
    {#each data as d}
      <li style="--color: {colorScale(d.label)}">
        <span class="swatch" />
        {d.label} <em>({d.value})</em>
      </li>
    {/each}
  </ul>
</div>

<style>
  .container {
    display: flex;
    gap: 1rem;
    align-items: flex-start;
  }

  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
    flex: 1.5;
  }

  .legend {
    flex: 1;
    list-style: none;
    margin: 0;
    padding: 0;
    display: grid;
    gap: 0.5rem;
  }

  .legend li {
    display: flex;
    align-items: center;
    gap: 0.35rem;
  }

  .swatch {
    width: 0.9rem;
    height: 0.9rem;
    background-color: var(--color);
    border-radius: 0.15rem;
    border: 1px solid oklch(60% 0.02 240 / 45%);
  }

  .chart-title {
    font-size: 1em;
    font-weight: bold;
    fill: currentColor;
  }

  .axis-label {
    font-size: 0.8em;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.7em;
    fill: currentColor;
    font-style: italic;
  }
</style>
