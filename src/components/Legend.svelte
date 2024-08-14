<script>
    import { each } from "svelte/internal";

    export let colorScale;
    export let hoveredContinent;
    console.log(colorScale.domain())
</script>

<div class="legend" on:mouseleave={() => hoveredContinent = null}>
    {#each colorScale.domain() as continent}
        <p 
        on:mouseover={() => hoveredContinent = continent}
        on:focus={() => hoveredContinent = continent}
        class:unhovered={hoveredContinent && hoveredContinent != continent}
        >
            <span style="background-color: {colorScale(continent)};"></span>
            {continent}
        </p>
    {/each}
</div>


<style>
    .legend {
        display: flex;
        flex-direction: row;
        justify-content: center;
        flex-wrap: wrap;
        column-gap: 20px;
        row-gap: 5px;
        margin-bottom: 0.25rem;
    }

    span {
    width: 9px;
    height: 9px;
    display: inline-block; /* Makes the span behave like an inline element */
    border-radius: 50%; /* Makes the span a circle */
    border: 1px solid rgba(0, 0, 0, 0.5); /* Adds a slight border */
}

p {
    font-size: .95rem;
    text-transform: uppercase;
    display: flex;
    align-items: center;
    column-gap: 3px; /* Adds a slight gap between the span and the text */
    transition: opacity 300ms ease;
    cursor: pointer;
}

.unhovered {
    opacity: 0.3;
}

</style>