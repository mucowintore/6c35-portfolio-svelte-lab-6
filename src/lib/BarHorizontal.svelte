<script>
  import * as d3 from 'd3';

  export let data = [];
  export let title = 'Lines of Code by Language';

  let width = 620;
  let height = 260;
  let margin = { top: 8, right: 170, bottom: 56, left: 82 };
  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  let xAxis;
  let yAxis;

  $: xScale = d3.scaleLinear()
    .domain([0, d3.max(data, (d) => d.value) || 1])
    .range([0, innerWidth]);
  $: maxXValue = d3.max(data, (d) => d.value) || 1;

  $: yScale = d3.scaleBand()
    .domain(data.map((d) => d.label))
    .range([0, innerHeight])
    .padding(0.2);

  $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    .domain(data.map((d) => d.label));

  $: maxBar = d3.greatest(data, (d) => d.value);

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(
      d3.axisBottom(xScale).ticks(Math.min(maxXValue, 10)).tickFormat(d3.format('d'))
    );
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }
</script>

<div class="container">
  <h2 class="chart-title">{title}</h2>
  <svg viewBox="0 0 {width} {height}">
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

        <text
          x={xScale(maxBar.value) + 10}
          y={yScale(maxBar.label) + yScale.bandwidth() / 2 + 1}
          text-anchor="start"
          dominant-baseline="central"
          class="annotation"
        >
          Most LOC
        </text>
      {/if}

      <text
        x={innerWidth / 2}
        y={innerHeight + margin.bottom - 12}
        text-anchor="middle"
        class="axis-label"
      >
        Lines of Code
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
    flex-direction: column;
    gap: 0.7rem;
    align-items: flex-start;
  }

  svg {
    max-width: 100%;
    width: 100%;
    height: auto;
    overflow: visible;
    flex: unset;
  }

  .legend {
    width: 100%;
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    flex-wrap: wrap;
    gap: 0.35rem 0.75rem;
    font-size: 0.85rem;
  }

  .legend li {
    display: flex;
    align-items: center;
    gap: 0.35rem;
    flex: 1 1 8.5rem;
    min-width: 8.5rem;
    white-space: nowrap;
  }

  .swatch {
    width: 0.8rem;
    height: 0.8rem;
    background-color: var(--color);
    border-radius: 0.15rem;
    border: 1px solid oklch(60% 0.02 240 / 45%);
  }

  .chart-title {
    font-size: 1.3rem;
    font-weight: 600;
    line-height: 1.2;
    margin: 0 0 0.05rem;
    color: currentColor;
  }

  .axis-label {
    font-size: 0.7em;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.56em;
    fill: currentColor;
    font-style: italic;
    font-weight: 600;
  }

  @media (max-width: 700px) {
    .legend {
      font-size: 0.78rem;
      gap: 0.3rem 0.55rem;
    }

    .legend li {
      flex: 1 1 7rem;
      min-width: 7rem;
    }

    .swatch {
      width: 0.72rem;
      height: 0.72rem;
    }
  }

  @media (max-width: 500px) {
    .legend li {
      flex: 1 1 100%;
      min-width: 0;
    }
  }
</style>
