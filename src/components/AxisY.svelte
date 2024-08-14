<script>
    import { each } from "svelte/internal";

    export let yScale;

    let ticks = yScale.domain();
</script>

<g class="axis-y">
    {#each ticks as tick}
        <g class="tick" transform="translate(0,{yScale(tick)})">
            <text>{tick}</text>    
        </g>
    {/each}
</g>