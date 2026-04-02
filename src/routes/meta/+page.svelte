<script>
  import { base } from '$app/paths';
  import { onMount, tick } from 'svelte';
  import * as d3 from 'd3';
  import { computePosition, autoPlacement, offset } from '@floating-ui/dom';
  import BarHorizontal from '$lib/BarHorizontal.svelte';

  let locData = [];
  let commits = [];
  let locByLanguage = [];
  let loading = true;
  let error = null;
  let width = 1000;
  let height = 600;
  let margin = { top: 10, right: 10, bottom: 56, left: 72 };
  const minRadius = 5;
  const maxRadius = 30;
  let xAxis;
  let yAxis;
  let yAxisGridlines;
  let commitTooltip;
  let tooltipPosition = { x: 0, y: 0 };
  let hoveredIndex = -1;
  let clickedCommits = [];

  const formatDateTime = d3.timeFormat('%b %-d, %Y, %-I:%M %p');
  const formatTickDay = d3.timeFormat('%a');
  const formatTickDate = d3.timeFormat('%-m/%-d');

  $: usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
    width: width - margin.left - margin.right,
    height: height - margin.top - margin.bottom,
  };
  $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};
  $: allLanguages = d3
    .rollups(
      locData,
      (v) => v.length,
      (d) => String(d.type).toUpperCase()
    )
    .sort((a, b) => b[1] - a[1])
    .map(([label]) => label);
  $: selectedLines = clickedCommits.length > 0 ? clickedCommits.flatMap((c) => c.lines) : locData;
  $: selectedLineCounts = d3.rollup(
    selectedLines,
    (v) => v.length,
    (d) => String(d.type).toUpperCase()
  );
  $: locByLanguage = allLanguages.map((label) => ({
    label,
    value: selectedLineCounts.get(label) ?? 0,
  }));
  $: barChartTitle =
    clickedCommits.length > 0
      ? 'Lines of Code by Language (Selected Commits)'
      : 'Lines of Code by Language (Entire Website)';
  $: lineExtent = d3.extent(commits, (d) => d.totalLines);
  $: minDate = d3.min(commits, (d) => d.datetime);
  $: maxDate = d3.max(commits, (d) => d.datetime);
  $: maxDatePlusDay = maxDate ? d3.timeDay.offset(maxDate, 1) : null;
  $: xScale = d3
    .scaleTime()
    .domain(
      minDate && maxDatePlusDay
        ? [minDate, maxDatePlusDay]
        : [new Date(), d3.timeDay.offset(new Date(), 1)]
    )
    .range([usableArea.left, usableArea.right])
    .nice();
  $: yScale = d3.scaleLinear().domain([0, 24]).range([usableArea.bottom, usableArea.top]);
  $: rScale = d3
    .scaleSqrt()
    .domain(lineExtent[0] != null && lineExtent[1] != null ? lineExtent : [0, 1])
    .range([minRadius, maxRadius]);
  async function dotInteraction(index, evt) {
    let hoveredDot = evt.target;

    if (evt.type === 'mouseenter') {
      hoveredIndex = index;
      await tick();
      if (!commitTooltip) return;

      tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
        strategy: 'fixed',
        middleware: [offset(5), autoPlacement()],
      });
    } else if (evt.type === 'mouseleave') {
      hoveredIndex = -1;
    } else if (evt.type === 'click') {
      let commit = commits[index];
      if (!clickedCommits.includes(commit)) {
        clickedCommits = [...clickedCommits, commit];
      } else {
        clickedCommits = clickedCommits.filter((c) => c !== commit);
      }
      hoveredDot.blur?.();
    }
  }

  $: if (xAxis && yAxis && yAxisGridlines) {
    d3.select(xAxis).call(d3.axisBottom(xScale).ticks(d3.timeSunday.every(1)).tickPadding(6));
    d3.select(xAxis)
      .selectAll('.tick text')
      .each(function (d) {
        const text = d3.select(this);
        text.text(null).attr('y', 0).attr('dy', 0);
        text.append('tspan').attr('x', 0).attr('dy', '1.35em').text(formatTickDay(d));
        text.append('tspan').attr('x', 0).attr('dy', '1.1em').text(formatTickDate(d));
      });
    d3
      .select(yAxis)
      .call(d3.axisLeft(yScale).tickFormat((d) => String(d % 24).padStart(2, '0') + ':00'));
    d3
      .select(yAxisGridlines)
      .call(d3.axisLeft(yScale).tickFormat('').tickSize(-usableArea.width));
  }

  onMount(async () => {
    try {
      locData = await d3.csv(`${base}/loc.csv`, (row) => ({
        ...row,
        line: Number(row.line),
        length: Number(row.length),
        depth: Number(row.depth),
        date: new Date(row.date + 'T00:00' + row.timezone),
        datetime: new Date(row.datetime),
      }));

      commits = d3
        .groups(locData, (d) => d.commit)
        .map(([commit, lines]) => {
          const first = lines[0];
          const datetime = first.datetime;
          return {
            id: commit,
            url: `https://github.com/${first.author}/portfolio-svelte/commit/${commit}`,
            author: first.author,
            datetime,
            date: first.date,
            hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
            totalLines: lines.length,
            lines,
          };
        })
        .sort((a, b) => a.datetime - b.datetime);

      commits = d3.sort(commits, (d) => -d.totalLines);
    } catch (err) {
      error = err;
    }

    loading = false;
  });
</script>

<svelte:head>
  <title>Meta Analysis</title>
</svelte:head>

<main>
  <h1>Meta Analysis</h1>
  <p>
    This page tracks statistics about the codebase itself, including lines of code by language.
  </p>

  {#if loading}
    <p>Loading code metrics...</p>
  {:else if error}
    <p>Could not load code metrics: {error.message}</p>
  {:else}
    <section class="scatterplot-section">
      <h2>Commit Activity by Time and Date</h2>
      <div class="chart-container">
        <svg viewBox={`0 0 ${width} ${height}`}>
          <g
            class="gridlines"
            transform={`translate(${usableArea.left}, 0)`}
            bind:this={yAxisGridlines}
          />
          <g
            class="x-axis"
            transform={`translate(0, ${usableArea.bottom})`}
            bind:this={xAxis}
          />
          <g
            class="y-axis"
            transform={`translate(${usableArea.left}, 0)`}
            bind:this={yAxis}
          />
          <g class="dots">
            {#each commits as commit, index}
              <circle
                class:selected={clickedCommits.includes(commit)}
                cx={xScale(commit.datetime)}
                cy={yScale(commit.hourFrac)}
                r={rScale(commit.totalLines)}
                fill="#8aa0b2"
                fill-opacity="0.7"
                data-index={index}
                role="button"
                aria-label={`Commit ${commit.id}`}
                aria-pressed={clickedCommits.includes(commit)}
                tabindex="0"
                on:mouseenter={(evt) => dotInteraction(index, evt)}
                on:mouseleave={(evt) => dotInteraction(index, evt)}
                on:click={(evt) => dotInteraction(index, evt)}
                on:keydown={(evt) => {
                  if (evt.key === 'Enter' || evt.key === ' ') {
                    evt.preventDefault();
                    dotInteraction(index, { type: 'click', target: evt.currentTarget });
                  }
                }}
              />
            {/each}
          </g>
        </svg>
      </div>
      {#if hoveredIndex >= 0 && hoveredCommit.id}
        <section
          class="tooltip"
          bind:this={commitTooltip}
          style={`left: ${tooltipPosition.x}px; top: ${tooltipPosition.y}px;`}
        >
          <p><strong>Commit:</strong> {hoveredCommit.id}</p>
          <p><strong>Author:</strong> {hoveredCommit.author}</p>
          <p><strong>Date:</strong> {formatDateTime(hoveredCommit.datetime)}</p>
          <p><strong>Lines edited:</strong> {hoveredCommit.totalLines}</p>
        </section>
      {/if}
    </section>

    <section class="bar-chart-section">
      <BarHorizontal data={locByLanguage} title={barChartTitle} />
    </section>
  {/if}
</main>

<style>
  .scatterplot-section {
    margin: 1.8rem 0 2rem;
  }

  .scatterplot-section h2 {
    margin: 0 0 0.75rem;
    font-size: 1.3rem;
    font-weight: 600;
    line-height: 1.2;
  }

  .bar-chart-section {
    margin-top: 2.4rem;
  }

  .chart-container {
    overflow-x: auto;
    overflow-y: visible;
  }

  svg {
    width: 100%;
    height: auto;
  }

  circle {
    transition: 200ms;
    cursor: pointer;
  }

  circle:hover:not(.selected) {
    fill: #5f8f6e;
    fill-opacity: 0.9;
  }

  circle:focus,
  circle:focus-visible {
    outline: none;
  }

  .selected {
    fill: #2f8f4e;
    fill-opacity: 1;
    stroke: rgba(255, 255, 255, 0.85);
    stroke-width: 1.25;
  }

  .tooltip {
    position: fixed;
    z-index: 1000;
    pointer-events: none;
    width: max-content;
    max-width: min(32rem, 100%);
    padding: 0.65rem 0.8rem;
    border: 1px solid #d1d5db;
    border-radius: 0.5rem;
    background: #ffffff;
    box-shadow: 0 6px 18px rgba(0, 0, 0, 0.12);
    color: #111827;
    font-size: 0.9rem;
    line-height: 1.35;
  }

  .tooltip p {
    margin: 0.15rem 0;
  }

  :global(.gridlines .tick line) {
    stroke: #6b7280;
    stroke-opacity: 0.25;
  }

  :global(.gridlines .domain) {
    display: none;
  }

  :global(.x-axis text),
  :global(.y-axis text) {
    font-size: 0.8rem;
  }
</style>
