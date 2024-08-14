<script>
    export let xScale;
    export let height;
    export let width;

    $: ticks = xScale.ticks(4);
</script>

<g class="axis-x" transform="translate(0,{height})">
    {#each ticks as tick}
        <g class="tick" transform="translate({xScale(tick)},0)">
            <text x="3">{tick}</text>
            <line
                x1="0"
                x2="0"
                y1="0"
                y2="-{height}"
                stroke="hsla(215, 15%, 91%, 1)"
            ></line>
        </g>
    {/each}
    <text class="axis-title" x={width} text-anchor="end" dominant-baseline="hanging" dy="4">
        Happiness, out of 10 &rarr
    </text>
</g>
