<script lang="ts">
  import svelteLogo from "./assets/svelte.svg";

  let watcher_position = [0, 0];
  let object_position = [0, 0];

  const watchActualPosition = () => {
    console.log("Watcher position")
    navigator.geolocation.watchPosition((position) => {
      watcher_position[0] = position.coords.latitude;
      watcher_position[1] = position.coords.longitude;
    });
  };

  const getObjectPosition = () => {
    console.log("Object position")
    navigator.geolocation.getCurrentPosition((position) => {
      object_position[0] = position.coords.latitude;
      object_position[1] = position.coords.longitude;
    });
  };

  let distanceBetweenPositions: number = 0;
  let distanceInKM: number = 0;
  let distanceInCM: number = 0;
  let distanceInM: number = 0;

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

    distanceInKM = parseInt(String(distanceBetweenPositions));
    distanceInM = parseInt(
      String((distanceBetweenPositions - distanceInKM) * 1000)
    );
    distanceInCM = parseInt(
      String(
        ((distanceBetweenPositions - distanceInKM) * 1000 - distanceInM) * 100
      )
    );
  };

  let angle: number = 0;

  const angleFromCoordinate = () => {
    // Returns angle in degrees based on coordinates
    angle = (Math.atan2(object_position[1] - watcher_position[1], object_position[0] - watcher_position[0]) * 180) / Math.PI;
    if (angle < 0){
      angle += 360;
    }
  };

  let ms = 1000;
  let clear;
  $: {
    clearInterval(clear);
    clear = setInterval(distance, ms);
    clear = setInterval(angleFromCoordinate, ms);
  }
</script>

<main>
  <div>
    <a href="https://vitejs.dev" target="_blank" rel="noreferrer">
      <img src="/vite.svg" class="logo" alt="Vite Logo" style="transform: rotate({angle}deg)"/>
    </a>
  </div>
  <h1>Locate an object</h1>

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
        <button on:click={getObjectPosition}>Set your object position</button>
        <div class="coords">
          <p>Object latitude : {object_position[0]}</p>
          <p>Object longitude : {object_position[1]}</p>
        </div>
      </div>

      <div class="coords">
        <p>
          Distance to your object : {#if distanceInKM != 0}{distanceInKM} Km /
          {/if}{#if distanceInM != 0}{distanceInM} m / {/if}{distanceInCM} cm
        </p>
      </div>
      <div class="coords">
        <p>Angle to your object : {angle}°</p>
        {#if angle < 225 && angle >= 135}
          <p>Direction : South</p>
        {/if}
        {#if angle < 135 && angle >= 45}
          <p>Direction : East</p>
        {/if}
        {#if angle < 315 && angle >= 225}
          <p>Direction : West</p>
        {/if}
        {#if angle < 45  || angle >= 315}
          <p>Direction : North</p>
        {/if}
      </div>
    </div>
  {:else}
    <p>The navigator doesn't have Geolocation</p>
  {/if}
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
    transition: transform 0.3s;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
</style>
