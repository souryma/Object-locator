<script lang="ts">
    export let display: boolean = false
    let latitude: number = 0
    let longitude: number = 0
    let watchID: number = 0

    const getValues = (lat: number, lon: number) => {
        latitude = lat
        longitude = lon
    }
    
    const getPosition = () => {
        watchID = navigator.geolocation.watchPosition((position) => {
            getValues(position.coords.latitude, position.coords.longitude);
        }); 
    }  
</script>

{#if display}
<div class="position">
    {#if 'geolocation' in navigator}
        <button on:click={getPosition}>
            Watch position
        </button>
        <p>Your latitude : {latitude}</p>
        <p>Your longitude : {longitude}</p>
        <p>watchID : {watchID}</p>
    {:else}
        <p>This navigator is not supporting geolocalisation.</p>
    {/if}
</div>
{/if}

<style>
    .position {
        display: flex;
        flex-direction: column;
    }
</style>