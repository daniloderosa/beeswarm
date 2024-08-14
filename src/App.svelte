<script>
  import data from "$data/data.js";
  import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
  import { scaleLinear, scaleBand, scaleOrdinal, scaleSqrt } from "d3-scale";
  import { mean, rollups } from "d3-array";
  import AxisX from "$components/AxisX.svelte";
  import AxisY from "$components/AxisY.svelte";

  let width = 10000,
    height = 600;
  const margin = { top: 0, right: 250, left: 250, bottom: 25 };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  const RADIUS = 5;

  // Generate the average for each continent, so that we can sort according to that
  const continents = rollups(
    data,
    (v) => mean(v, (d) => d.happiness),
    (d) => d.continent,
  ) // Group data by continent and return the group-wide mean
    .sort((a, b) => a[1] - b[1]) // Sort according to value
    .map((d) => d[0]); // Grab the continent name

  $: xScale = scaleLinear()
    .domain([1, 9]) // Alternatively, we could pass .domain(extent(data, d => d.happiness))
    .range([0, innerWidth]);

  let yScale = scaleBand()
    .domain(continents)
    .range([innerHeight, 0])
    .paddingOuter(0.5);

  const colorRange = [
    "#dda0dd",
    "#fe7f2d",
    "#fcca46",
    "#a1c181",
    "#619b8a",
    "#eae2b7",
  ];
  let colorScale = scaleOrdinal().domain(continents).range(colorRange);

  let radiusScale = scaleSqrt()
    .domain([1, 9])
    .range(width < 568 ? [1, 6] : [5, 11]);

  let simulation = forceSimulation(data);

  $: {
    simulation
      .force(
        "x",
        forceX()
          .x((d) => xScale(d.happiness))
          .strength(0.8),
      )
      .force(
        "y",
        forceY()
          .y((d) => yScale(d.continent))
          .strength(0.2),
      )
      .force(
        "collide",
        forceCollide().radius((d) => radiusScale(d.happiness)),
      )
      .alpha(0.3) // [0, 1] The rate at which the simulation finishes. You should increase this if you want a faster simulation, or decrease it if you want more "movement" in the simulation.
      .alphaDecay(0.0005) // [0, 1] The rate at which the simulation alpha approaches 0. you should decrease this if your bubbles are not completing their transitions between simulation states.
      .restart(); // Restart the simulation
  }

  let nodes = [];
  simulation.on("tick", () => {
    nodes = simulation.nodes(); // Update the nodes array
  });
</script>

<h1>The happiest countries in the world</h1>
<div class="chart-container" bind:clientWidth={width}>
  <svg {width} {height}>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
      <AxisX {xScale} height={innerHeight} width={innerWidth} />
      <AxisY {yScale} />
      {#each nodes as node}
        <circle
          cx={node.x}
          cy={node.y}
          r={radiusScale(node.happiness)}
          fill={colorScale(node.continent)}
          stroke="black"
        />
      {/each}
    </g>
  </svg>
</div>

<style>
  :global(.tick text, .axis-title) {
    font-size: 14px; /* How big our text is */
    font-weight: 400; /* How bold our text is */
    fill: hsla(212, 10%, 53%, 1); /* The color of our text */
    user-select: none; /* Prevents text from being selected */
  }
  h1 {
    font-size: 1.35rem; /* How big our text is */
    margin: 0 0 0.5rem 0; /* Adds a bottom margin */
    font-weight: 600; /* How bold our text is */
    text-align: center;
  }
</style>
