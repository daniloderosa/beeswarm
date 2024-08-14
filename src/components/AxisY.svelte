<script>
    import { each } from "svelte/internal";
    import { fade } from "svelte/transition";

    export let yScale;
    export let groupByContinent;

    let ticks = yScale.domain();
</script>

<g class="axis-y">
    {#each ticks as tick, index}
        {#if groupByContinent}
            <g class="tick" transform="translate(0,{yScale(tick)})" in:fade={{delay: index * 100}} out:fade>
                <text>{tick}</text>
            </g>
        {/if}
    {/each}
</g>
