<script>
  import data from "$data/data.js";
  import { onMount, onDestroy } from "svelte";
  import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
  import { scaleLinear, scaleBand, scaleOrdinal, scaleSqrt } from "d3-scale";
  import { group, mean, rollups } from "d3-array";
  import AxisX from "$components/AxisX.svelte";
  import AxisY from "$components/AxisY.svelte";
  import Legend from "$components/Legend.svelte";
  import Tooltip from "$components/Tooltip.svelte";
  import { fade } from "svelte/transition";

  let clientWidth = window.innerWidth;
  let height = 600;
  let margin = { top: 0, right: 50, left: 50, bottom: 25 }; // Default values

  $: margin = {
    top: 0,
    right: clientWidth > 768 ? 250 : 50,
    left: clientWidth > 768 ? 250 : 50,
    bottom: 25,
  };

  $: innerWidth = clientWidth - margin.left - margin.right;
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
    .range(clientWidth < 768 ? [1, 6] : [5, 11]);

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
          .y((d) => groupByContinent ? yScale(d.continent) : innerHeight / 2)
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

  // Set up and remove the resize event listener properly
  onMount(() => {
    const handleResize = () => {
      clientWidth = window.innerWidth; // This triggers reactivity
    };
    window.addEventListener("resize", handleResize);
    return () => {
      window.removeEventListener("resize", handleResize);
    };
  });
  $: console.log("Width:", clientWidth, "Inner Width:", innerWidth);

  let hovered;
  let hoveredContinent;

  let groupByContinent = false;
</script>

<h1 style="margin-bottom: 10px">The Happiest Countries in the World</h1>
<Legend {colorScale} bind:hoveredContinent></Legend>
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="chart-container" style="position: relative;" on:click={(e) => {groupByContinent = !groupByContinent; hovered = null}}>
  <svg width={clientWidth} {height}>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
      <AxisX {xScale} height={innerHeight} width={innerWidth} />
      <AxisY {yScale} {groupByContinent}/>
      {#if hovered}
        <line
        in:fade={{ delay: 100, duration: 100 }} 
          x1={hovered.x}
          x2={hovered.x}
          y1={height - margin.bottom}
          y2={hovered.y + margin.top + radiusScale(hovered.happiness)}
          stroke={colorScale(hovered.continent)}
          stroke-width="2"
        />
      {/if}
      {#each nodes as node, index }
        <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
        <circle
          in:fade={{delay : 400 + index * 10 }}
          cx={node.x}
          cy={node.y}
          r={radiusScale(node.happiness)}
          stroke={hovered || hoveredContinent
            ? hovered === node || hoveredContinent === node.continent
              ? "black"
              : "transparent"
            : "#00000090"}
          fill={colorScale(node.continent)}
          title={node.country}
          opacity={hovered || hoveredContinent ? (hovered === node || hoveredContinent === node.continent ? 1 : 0.3) : 1}
          on:mouseover={() => (hovered = node)}
          on:focus={() => (hovered = node)}
          on:mouseleave={() => (hovered = null)}
          on:click={(e) => {e.stopPropagation()}}
          tabindex="0"
        />
      {/each}
    </g>
  </svg>
  {#if hovered}
    <Tooltip data={hovered} {margin} {colorScale}></Tooltip>
  {/if}
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
  circle {
    transition:
      stroke 300ms ease,
      opacity 300ms ease;
    cursor: pointer;
  }
</style>
