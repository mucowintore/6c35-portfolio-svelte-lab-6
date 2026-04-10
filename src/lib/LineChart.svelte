<script>
  import * as d3 from 'd3';

  export let data = [];

  let width = 1000;
  let height = 300;
  let margin = { top: 16, right: 20, bottom: 36, left: 56 };
  let xAxis;
  let yAxis;
  let hoveredDay = null;

  $: usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
    width: width - margin.left - margin.right,
    height: height - margin.top - margin.bottom,
  };

  $: dateExtent = d3.extent(data, (d) => d.date);
  $: xScale = d3
    .scaleTime()
    .domain(
      dateExtent[0] && dateExtent[1]
        ? [dateExtent[0], dateExtent[1]]
        : [new Date(), d3.timeDay.offset(new Date(), 1)]
    )
    .range([usableArea.left, usableArea.right]);

  $: maxCount = d3.max(data, (d) => d.count) ?? 0;
  $: yScale = d3
    .scaleLinear()
    .domain([0, maxCount > 0 ? maxCount : 1])
    .nice()
    .range([usableArea.bottom, usableArea.top]);

  $: line = d3
    .line()
    .x((d) => xScale(d.date))
    .y((d) => yScale(d.count))
    .curve(d3.curveBumpX);
  $: dayRegions = (() => {
    if (data.length === 0) return [];

    return data.map((d, i, arr) => {
      const prev = arr[i - 1];
      const next = arr[i + 1];
      const left = prev ? new Date((d.date.getTime() + prev.date.getTime()) / 2) : d.date;
      const right = next ? new Date((d.date.getTime() + next.date.getTime()) / 2) : d.date;

      return {
        date: d.date,
        weekday: d.date.toLocaleString('en', { weekday: 'long' }),
        x: xScale(left),
        width: xScale(right) - xScale(left),
      };
    });
  })();

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }
</script>

<section class="line-chart">
  <h3>{hoveredDay ? `Lines Edited on ${hoveredDay}s` : 'Lines Edited by Day'}</h3>
  <div class="chart-wrap">
    <svg
      viewBox={`0 0 ${width} ${height}`}
      role="img"
      aria-label="Line chart of lines edited by day"
      on:mouseleave={() => (hoveredDay = null)}
    >
      <g transform={`translate(0, ${usableArea.bottom})`} bind:this={xAxis} />
      <g transform={`translate(${usableArea.left}, 0)`} bind:this={yAxis} />

      <g class="highlight-bands">
        {#if hoveredDay}
          {#each dayRegions as region}
            {#if region.weekday === hoveredDay}
              <rect
                x={region.x}
                y={usableArea.top}
                width={region.width}
                height={usableArea.bottom - usableArea.top}
                fill="var(--color-accent)"
                opacity="0.2"
              />
            {/if}
          {/each}
        {/if}
      </g>

      <path d={line(data)} fill="none" stroke="steelblue" stroke-width="2" />

      {#each data as d}
        {@const isHighlighted = d.date.toLocaleString('en', { weekday: 'long' }) === hoveredDay}
        <circle
          cx={xScale(d.date)}
          cy={yScale(d.count)}
          r={isHighlighted ? 5 : 3}
          fill={isHighlighted ? 'var(--color-accent)' : 'steelblue'}
        />
        {#if isHighlighted}
          <text
            x={xScale(d.date)}
            y={usableArea.top + 15}
            text-anchor="middle"
            font-size="12"
            fill="var(--color-accent)"
            class="day-annotation"
          >
            {Math.round(d.count)}
          </text>
        {/if}
      {/each}

      <text
        x={usableArea.left + (usableArea.right - usableArea.left) / 2}
        y={height - 5}
        text-anchor="middle"
        class="axis-label"
      >
        Date
      </text>

      <text
        x={-(usableArea.top + (usableArea.bottom - usableArea.top) / 2)}
        y={16}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label"
      >
        Number of Lines Edited
      </text>

      <g class="interaction-regions">
        {#each dayRegions as region}
          <rect
            x={region.x}
            y={usableArea.top}
            width={region.width}
            height={usableArea.bottom - usableArea.top}
            fill="transparent"
            role="presentation"
            on:mouseenter={() => (hoveredDay = region.weekday)}
          />
        {/each}
      </g>
    </svg>
  </div>
</section>

<style>
  .line-chart {
    margin-top: 2rem;
  }

  h3 {
    margin: 0 0 0.5rem;
    font-size: 1.15rem;
  }

  .chart-wrap {
    overflow-x: auto;
  }

  svg {
    width: 100%;
    height: auto;
    display: block;
  }

  .axis-label {
    font-size: 0.8em;
    fill: currentColor;
  }

  .day-annotation {
    font-weight: 600;
  }
</style>
