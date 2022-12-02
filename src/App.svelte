<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import Counter from './lib/Counter.svelte'
  import Distance from './lib/Distance.svelte';
  import GeolocationWatch from './lib/GeolocationWatch.svelte';

  let first_position = [0, 0]
  let second_position = [0, 0]

  const getFirstPosition = () => {
      navigator.geolocation.getCurrentPosition((position) => {
          first_position[0] = position.coords.latitude
          first_position[1] = position.coords.longitude
      });
  }

  const getSecondPosition = () => {
    navigator.geolocation.getCurrentPosition((position) => {
        second_position[0] = position.coords.latitude
        second_position[1] = position.coords.longitude
    });
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

  <div class="card">
    <div class="position">
      <button on:click={getFirstPosition}>
        Get position
      </button>
      <p>Lat : {first_position[0]}</p>
      <p>Lon : {first_position[1]}</p>
    </div>
    
    <div class="position">
      <button on:click={getSecondPosition}>
        Get position
      </button>
      <p>Lat : {second_position[0]}</p>
      <p>Lon : {second_position[1]}</p>
    </div>

    <Distance lat1={first_position[0]} lon1={first_position[1]} lat2={second_position[0]} lon2={second_position[1]}/>  
  </div>

  <p>
    Check out <a href="https://github.com/sveltejs/kit#readme" target="_blank" rel="noreferrer">SvelteKit</a>, the official Svelte app framework powered by Vite!
  </p>

  <p class="read-the-docs">
    Click on the Vite and Svelte logos to learn more
  </p>
</main>

<style>
  .position {
    display: flex;
    justify-content: space-around;
  }

  .card {
    height: 200px;
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