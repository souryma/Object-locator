<script lang="ts">
  import "./lib/fulltilt.js";
  import { distance } from "./lib/distance.svelte";
  import { checkLatitude, checkLongitude } from "./lib/utils.svelte";

  // Position of the navigator (mobile)
  let watcher_position: [number, number] = [0, 0];
  // Position of the object (immobile)
  let object_position: [number, number] = [0, 0];
  let isLatitudeValid: boolean = true;
  let isLongitudeValid: boolean = true;
  let isCompassHeadingEnabled: boolean = false;
  let angleToObject: number = 0;
  let direction: string = "";
  let distanceInKM: number = 0;
  let distanceInCM: number = 0;
  let distanceInM: number = 0;
  let angleToNorth: number = 0;
  // Data refresh rate in milliseconds
  const refreshRate: number = 1000;
  let clear;
  let isObjectPositionSet: boolean = false;
  let isWatcherPositionSet: boolean = false;
  let deviceOrientation: number = 0;
  let isDeviceOrientationEnabled: boolean = false;
  let isDeviceOrientationAuthorized: boolean = false;
  let pageTitle: string = "";
  let compassHeading = 0;

  const getWatcherPosition = () => {
    navigator.geolocation.watchPosition((position) => {
      watcher_position[0] = position.coords.latitude;
      watcher_position[1] = position.coords.longitude;
    });
    isWatcherPositionSet = true;

    setOffsetToNorthPole();
  };

  const getObjectPosition = () => {
    navigator.geolocation.getCurrentPosition((position) => {
      object_position[0] = position.coords.latitude;
      object_position[1] = position.coords.longitude;
    });
  };

  const confirmPosition = () => {
    if (checkLatitude(object_position[0]) == true) {
      isLatitudeValid = true;
    } else {
      isLatitudeValid = false;
    }
    if (checkLongitude(object_position[1]) == true) {
      isLongitudeValid = true;
    } else {
      isLongitudeValid = false;
    }
    if (isLongitudeValid && isLatitudeValid) {
      isObjectPositionSet = true;
    }
  };

  function fillDistanceUnits(distance: number) {
    // Split the full distance in distance units (kilometers, meters, centimeters)

    distanceInKM = parseInt(String(distance));

    distanceInM = parseInt(String((distance - distanceInKM) * 1000));

    distanceInCM = parseInt(
      String(((distance - distanceInKM) * 1000 - distanceInM) * 100)
    );
  }

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

  function setOffsetToNorthPole() {
    angleToNorth = compassHeading;
  }

  $: {
    clearInterval(clear);
    clear = setInterval(
      fillDistanceUnits(
        distance(
          watcher_position[0],
          watcher_position[1],
          object_position[0],
          object_position[1]
        )
      ),
      refreshRate
    );
    clear = setInterval(angleFromCoordinate, refreshRate);
  }

  if ("geolocation" in navigator) {
    pageTitle = "Simple GPS";
  } else {
    pageTitle = "Error !";
  }

  const enableCompassHeading = () => {
    if (typeof DeviceOrientationEvent.requestPermission === "function") {
      DeviceOrientationEvent.requestPermission()
        .then((permissionState) => {
          if (permissionState === "granted") {
            window.addEventListener(
              "deviceorientation",
              deviceOrientationHandler
            );
            isDeviceOrientationEnabled = true;
            isDeviceOrientationAuthorized = true;

            // Obtain a new *world-oriented* Full Tilt JS DeviceOrientation Promise
            var promise = FULLTILT.getDeviceOrientation({ type: "world" });

            // Wait for Promise result
            promise
              .then(function (deviceOrientation) {
                // Device Orientation Events are supported

                // Register a callback to run every time a new
                // deviceorientation event is fired by the browser.
                deviceOrientation.listen(function () {
                  // Get the current *screen-adjusted* device orientation angles
                  var currentOrientation =
                    deviceOrientation.getScreenAdjustedEuler();

                  // Calculate the current compass heading that the user is 'looking at' (in degrees)
                  compassHeading = 360 - currentOrientation.alpha;
                  isCompassHeadingEnabled = true;
                });
              })
              .catch(function (errorMessage) {
                // Device Orientation Events are not supported
                console.log(errorMessage);
                isCompassHeadingEnabled = false;
              });
          }
        })
        .catch(console.error);
    } else {
      isDeviceOrientationEnabled = false;
      isDeviceOrientationAuthorized = true;
    }
  };

  function deviceOrientationHandler(eventData) {
    // device orienation angle (counterclockwise)
    if (isDeviceOrientationEnabled) {
      deviceOrientation = eventData.alpha;
      isDeviceOrientationEnabled = true;
      isDeviceOrientationAuthorized = true;
    }
  }
</script>

<main>
  <h1>{pageTitle}</h1>

  {#if "geolocation" in navigator}
    <div class="card">
      {#if isDeviceOrientationAuthorized == false}
      <h2>Authorize your orientation :</h2>
        <button on:click={enableCompassHeading}>Give my device orientation</button>
      {:else if isObjectPositionSet == false}
        <div class="position">
          <h2>Set your destination position :</h2>
          <button on:click={getObjectPosition}>Set your position</button>
          {#if isLatitudeValid == false}
            <p class="error">Latitude must be between -90 and 90 degrees</p>
          {/if}
          <p>
            Latitude : <input
              type="number"
              bind:value={object_position[0]}
              min="-90"
              max="90"
            />
          </p>
          {#if isLongitudeValid == false}
            <p class="error">Longitude must be between -180 and 180 degrees</p>
          {/if}
          <p>
            Longitude : <input
              type="number"
              bind:value={object_position[1]}
              min="-180"
              max="180"
            />
          </p>
          <button on:click={confirmPosition}>Confirm position</button>
        </div>
      {:else if isWatcherPositionSet == false}
        <div class="position">
          <h2>Authorize your position :</h2>
          <button on:click={getWatcherPosition}>Follow my position</button>
        </div>
      {/if}

      {#if isObjectPositionSet && isWatcherPositionSet && isDeviceOrientationAuthorized}
        {#if isDeviceOrientationEnabled == false}
          <p class="error">Can't access device orientation data.</p>
          <p>
            The arrow will show you the right direction if your device is
            heading north.
          </p>
        {/if}

        <div>
          <img
            src="/simpleGPS_arrow.png"
            class="logo"
            alt="Direction to the object"
            style="transform: rotate({deviceOrientation +
              (angleToObject - angleToNorth)}deg)"
          />
        </div>

        <div class="distance">
          <p>
            Distance to your object : {#if distanceInKM != 0}{distanceInKM} Km /
            {/if}{#if distanceInM != 0}{distanceInM} m / {/if}{distanceInCM} cm
          </p>
        </div>
        <div class="orientation">
          <p>Direction : {direction}</p>
        </div>
      {/if}
    </div>
  {:else}
    <p>This navigator doesn't have Geolocation. Try another one !</p>
  {/if}
</main>

<style>
  .error {
    color: rgb(228, 78, 78);
  }

  .card {
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
  }
  .logo {
    height: 13em;
  }
</style>
