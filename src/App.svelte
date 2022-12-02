<script lang="ts">
  import svelteLogo from "./assets/svelte.svg";
  import Distance from "./lib/Distance.svelte";

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

  {#if "geolocation" in navigator}
    <div class="card">
      <div class="position">
        <button on:click={watchActualPosition}> Get your position </button>
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

      <Distance
        lat1={watcher_position[0]}
        lon1={watcher_position[1]}
        lat2={object_position[0]}
        lon2={object_position[1]}
      />
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
