<script lang="ts">
  import svelteLogo from "./assets/svelte.svg";

  // Position of the navigator (mobile)
  let watcher_position = [0, 0];
  // Position of the object (immobile)
  let object_position = [0, 0];
  let angleToObject: number = 0;
  let direction: string = "";
  let distanceBetweenPositions: number = 0;
  let distanceInKM: number = 0;
  let distanceInCM: number = 0;
  let distanceInM: number = 0;
  // Data refresh rate in milliseconds
  let refreshRate: number = 1000;
  let clear;
  let isObjectPositionSet: boolean = false;
  let isWatcherPositionSet: boolean = false;
  let deviceOrientation: number = 0;
  let isDeviceOrientationEnabled: boolean = false;
  let pageTitle: string = "";

  const watchActualPosition = () => {
    navigator.geolocation.watchPosition((position) => {
      watcher_position[0] = position.coords.latitude;
      watcher_position[1] = position.coords.longitude;
    });
    isWatcherPositionSet = true;
  };

  const getObjectPosition = () => {
    navigator.geolocation.getCurrentPosition((position) => {
      object_position[0] = position.coords.latitude;
      object_position[1] = position.coords.longitude;
    });
    isObjectPositionSet = true;
  };

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

  const angleFromCoordinate = () => {
    // Returns angle in degrees based on coordinates
    angleToObject =
      (Math.atan2(
        object_position[1] - watcher_position[1],
        object_position[0] - watcher_position[0]
      ) *
        180) /
      Math.PI;
    if (angleToObject < 0) {
      angleToObject += 360;
    }

    if (angleToObject < 225 && angleToObject >= 135) {
      direction = "South";
    } else if (angleToObject < 135 && angleToObject >= 45) {
      direction = "East";
    } else if (angleToObject < 315 && angleToObject >= 225) {
      direction = "West";
    } else if (angleToObject < 45 || angleToObject >= 315) {
      direction = "North";
    }
  };

  $: {
    clearInterval(clear);
    clear = setInterval(distance, refreshRate);
    clear = setInterval(angleFromCoordinate, refreshRate);
  }

  if ("geolocation"! in navigator) {
    pageTitle = "Locate an object";
  } else {
    pageTitle = "Error !";
  }

  // Request device orientation permission
  const enableDeviceOrientation = () => {
    if (typeof DeviceOrientationEvent.requestPermission === "function") {
      DeviceOrientationEvent.requestPermission()
        .then((permissionState) => {
          if (permissionState === "granted") {
            window.addEventListener("deviceorientation", () => {});
            isDeviceOrientationEnabled = true;
          }
        })
        .catch(console.error);
    } else {
      isDeviceOrientationEnabled = false;
    }
  };

  function deviceOrientationHandler(eventData) {
    // device orienation angle (counterclockwise)
    if (isDeviceOrientationEnabled) {
      deviceOrientation = eventData.alpha;
      isDeviceOrientationEnabled = true;
    }
  }
</script>

<main>
  <h1>{pageTitle}</h1>

  {#if "geolocation" in navigator}
    <div class="card">
      {#if isObjectPositionSet == false}
        <div class="position">
          <button on:click={getObjectPosition}>Set your object position</button>
        </div>
      {:else if isWatcherPositionSet == false}
        <div class="position">
          <button on:click={watchActualPosition}>Authorize your position</button
          >
        </div>
      {/if}

        {#if isDeviceOrientationEnabled == true}
          <div class="position">
            <button on:click={enableDeviceOrientation}
              >Authorize orientation</button
            >
          </div>
          <p>Device orientation : {deviceOrientation}</p>
        {:else}
          <p>Can't acces device orientation data.</p>
        {/if}

      {#if isObjectPositionSet && isWatcherPositionSet}
        <div>
          <a href="https://vitejs.dev" target="_blank" rel="noreferrer">
            <img
              src="/Dark_Green_Arrow_Up.png"
              class="logo"
              alt="Direction to the object"
              style="transform: rotate({angleToObject + deviceOrientation}deg)"
            />
          </a>
        </div>

        <p>Image rotation : {angleToObject + deviceOrientation} deg</p>

        <div class="coordinates">
          <div class="coords">
            <p>Object latitude : {object_position[0]}</p>
            <p>Object longitude : {object_position[1]}</p>
          </div>
          <div class="coords">
            <p>Your latitude : {watcher_position[0]}</p>
            <p>Your longitude : {watcher_position[1]}</p>
          </div>
        </div>
        <div class="distance">
          <p>
            Distance to your object : {#if distanceInKM != 0}{distanceInKM} Km /
            {/if}{#if distanceInM != 0}{distanceInM} m / {/if}{distanceInCM} cm
          </p>
        </div>
        <div class="orientation">
          <p>Angle to your object : {angleToObject}°</p>
          <p>Direction : {direction}</p>
        </div>
      {/if}
    </div>
  {:else}
    <p>The navigator doesn't have Geolocation. Try another one !</p>
  {/if}
</main>

<style>
  .position {
    display: flex;
    justify-content: space-around;
  }

  .coordinates {
    display: flex;
    justify-content: space-between;
  }

  .coords {
    display: flex;
    flex-direction: column;
    justify-content: end;
  }

  .card {
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
  }
  .logo {
    height: 6em;
    padding: 1.5em;
    /* transition: transform 0.3s; */
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
</style>
