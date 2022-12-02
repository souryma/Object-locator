<script lang="ts">
  export let lat1: number = 0;
  export let lon1: number = 0;
  export let lat2: number = 0;
  export let lon2: number = 0;

  let distanceBetweenPositions: number = -1;

  const distance = () => {
    // The math module contains a function
    // named toRadians which converts from
    // degrees to radians.
    lon1 = (lon1 * Math.PI) / 180;
    lon2 = (lon2 * Math.PI) / 180;
    lat1 = (lat1 * Math.PI) / 180;
    lat2 = (lat2 * Math.PI) / 180;

    // Haversine formula
    let dlon = lon2 - lon1;
    let dlat = lat2 - lat1;
    let a =
      Math.pow(Math.sin(dlat / 2), 2) +
      Math.cos(lat1) * Math.cos(lat2) * Math.pow(Math.sin(dlon / 2), 2);

    let c = 2 * Math.asin(Math.sqrt(a));

    // Radius of earth in kilometers. Use 3956
    // for miles
    let r = 6371;

    // calculate the result
    distanceBetweenPositions = c * r;
  };
</script>

<div class="distance">
  <button on:click={distance}> Get distance between points </button>
  {#if distanceBetweenPositions >= 0}
    <p>Distance : {distanceBetweenPositions} km</p>
  {/if}
</div>
