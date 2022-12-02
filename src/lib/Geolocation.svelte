<script lang="ts">
    export let positionNumber: number = 0
    let latitude: number = 0
    let longitude: number = 0

    const getValues = (lat: number, lon: number) => {
        latitude = lat
        longitude = lon
    }
    
    const getPosition = () => {
        navigator.geolocation.getCurrentPosition((position) => {
            getValues(position.coords.latitude, position.coords.longitude);
        });
    }    
</script>

<div class="position">
    {#if 'geolocation' in navigator}
        <button on:click={getPosition}>
            Get position {positionNumber}
        </button>
        <p>Your latitude : {latitude}</p>
        <p>Your longitude : {longitude}</p>
    {:else}
        <p>This navigator is not supporting geolocalisation.</p>
    {/if}
</div>

<style>
    .position {
        display: flex;
        flex-direction: column;
    }
</style>