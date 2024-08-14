<script>
    export let data;
    export let margin;
    export let colorScale;

    import { fly, fade } from "svelte/transition";

    let xNudge = 20;
    let yNudge = 10;
</script>

<div
    class="tooltip"
    in:fly={{ y: 10, duration: 200, delay: 200 }}
    out:fade
    style="position: absolute;
    top: {data.y + margin.top + xNudge}px;
    left: {data.x + margin.left + yNudge}px;
    "
>
    <!-- Country name -->
    <h1>
        {data.country}
    </h1>
    <!-- Additional info under the country name -->
    <div class="info">
        <span class="score">
            {data.happiness}/10
        </span>
        <span
            class="continent"
            style="background: {colorScale(data.continent)}"
        >
            {data.continent}
        </span>
    </div>
    <span class="bar background" />
    <span
        class="bar foreground"
        style="width: {data.happiness * 10}%; 
    background: {colorScale(data.continent)}"
    />
</div>

<style>
    .tooltip {
        position: absolute;
        background: white;
        box-shadow: rgba(0, 0, 0, 0.15) 2px 3px 8px; /* Gives a nice 3d effect */
        padding: 8px 6px; /* Adds space around content within tooltip */
        border-radius: 3px; /* Rounds corners */
        pointer-events: none; /* Prevents tooltip from blocking mouse events */

 /*        transition:
            top 50ms ease,
            left 50ms ease; */
    }

    .info {
        display: flex;
        justify-content: space-between;
        column-gap: 8px;
    }

    .score {
        font-size: 0.8rem;
    }

    .continent {
        font-size: 0.65rem;
        padding: 3px 4px 2px 4px;
        border-radius: 3px;
        text-transform: uppercase;
        white-space: nowrap; /* Prevents line breaks */
    }

    h1 {
        margin: 0;
        font-size: 1rem;
        font-weight: 500;
        margin-bottom: 3px;
    }

    .bar {
        position: absolute;
        bottom: 0; /* Forces to bottom of tooltip */
        left: 0; /* Forces to left of tooltip */
        height: 3px;
        width: 100%;
    }

    .bar.background {
        background: #eee;
    }
</style>
