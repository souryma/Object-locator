<script lang="ts">
  import svelteLogo from "./assets/svelte.svg";

  let watcher_position = [0, 0];
  let object_position = [0, 0];

  const watchActualPosition = () => {
    navigator.geolocation.watchPosition((position) => {
      watcher_position[0] = position.coords.latitude;
      watcher_position[1] = position.coords.longitude;
    });
  };

  const getObjectPosition = () => {
    navigator.geolocation.getCurrentPosition((position) => {
      object_position[0] = position.coords.latitude;
      object_position[1] = position.coords.longitude;
    });
  };

  let distanceBetweenPositions: number = 0;

  const distance = () => {
    // The math module contains a function
    // named toRadians which converts from
    // degrees to radians.
    let lon1 = (watcher_position[1] * Math.PI) / 180;
    let lon2 = (object_position[1] * Math.PI) / 180;
    let lat1 = (watcher_position[0] * Math.PI) / 180;
    let lat2 = (object_position[0] * Math.PI) / 180;

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

  let ms = 1000;
  let clear;
  $: {
    clearInterval(clear);
    clear = setInterval(distance, ms);
  }
</script>

<main>
  <div>
    <a href="https://vitejs.dev" target="_blank" rel="noreferrer">
      <img src="/vite.svg" class="logo" alt="Vite Logo" />
    </a>
    <a href="https://svelte.dev" target="_blank" rel="noreferrer">
      <img src={svelteLogo} class="logo svelte" alt="Svelte Logo" />
    </a>
  </div>
  <h1>Vite + svelte</h1>

  <!-- DEBUG : Distance update speed -->
  Distance update speed :
  <input type="number" bind:value={ms} />

  {#if "geolocation" in navigator}
    <div class="card">
      <div class="position">
        <button on:click={watchActualPosition}>Authorize your position</button>
        <div class="coords">
          <p>Your latitude : {watcher_position[0]}</p>
          <p>Your longitude : {watcher_position[1]}</p>
        </div>
      </div>

      <div class="position">
        <button on:click={getObjectPosition}> Set your object position </button>
        <div class="coords">
          <p>Object latitude : {object_position[0]}</p>
          <p>Object longitude : {object_position[1]}</p>
        </div>
      </div>

      <div class="coords">
        <p>Distance to your object : {distanceBetweenPositions} M</p>
      </div>
    </div>
  {:else}
    <p>The navigator doesn't have Geolocation</p>
  {/if}

  <p>
    Check out <a
      href="https://github.com/sveltejs/kit#readme"
      target="_blank"
      rel="noreferrer">SvelteKit</a
    >, the official Svelte app framework powered by Vite!
  </p>

  <p class="read-the-docs">Click on the Vite and Svelte logos to learn more</p>
</main>

<style>
  .position {
    display: flex;
    justify-content: space-around;
  }

  .coords {
    display: flex;
    flex-direction: column;
  }

  .card {
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
  }
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
