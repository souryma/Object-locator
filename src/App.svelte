<script lang="ts">
  import "./lib/fulltilt.js";

  // GPS coordinates of the north pole
  const north_pole: [number, number] = [90, 0];
  const earth_radius = 6371;
  // Position of the navigator (mobile)
  let watcher_position: [number, number] = [0, 0];
  // Position of the object (immobile)
  let object_position: [number, number] = [0, 0];
  let isLatitudeValid: boolean = true;
  let isLongitudeValid: boolean = true;
  let angleToObject: number = 0;
  let direction: string = "";
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
  let isDeviceOrientationAuthorized: boolean = false;
  let pageTitle: string = "";

  let compassHeading = 0;

  // Obtain a new *world-oriented* Full Tilt JS DeviceOrientation Promise
  var promise = FULLTILT.getDeviceOrientation({ 'type': 'world' });

  // Wait for Promise result
  promise.then(function(deviceOrientation) { // Device Orientation Events are supported

    // Register a callback to run every time a new 
    // deviceorientation event is fired by the browser.
    deviceOrientation.listen(function() {

      // Get the current *screen-adjusted* device orientation angles
      var currentOrientation = deviceOrientation.getScreenAdjustedEuler();

      // Calculate the current compass heading that the user is 'looking at' (in degrees)
      compassHeading = 360 - currentOrientation.alpha;
    });

  }).catch(function(errorMessage) { // Device Orientation Events are not supported

    console.log(errorMessage);

    // Implement some fallback controls here...

  });

  const getWatcherPosition = () => {
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
  };

  const confirmPosition = () => {
    if (checkLatitude(object_position[0]) == true) {
      isLatitudeValid = true;
      if (checkLongitude(object_position[1]) == true) {
        isObjectPositionSet = true;
        isLatitudeValid = true;
        isLongitudeValid = true;
      } else {
        isLongitudeValid = false;
      }
    } else {
      isLatitudeValid = false;
    }
  };

  function checkLatitude(latitude: number) {
    // Returns true if the given number is between -90 and 90
    if (latitude <= 90 && -90 <= latitude) {
      return true;
    } else {
      return false;
    }
  }

  function checkLongitude(longitude: number) {
    // Returns true if the given number is between -180 and 180
    if (longitude <= 180 && -180 <= longitude) {
      return true;
    } else {
      return false;
    }
  }

  function distance(
    latitude1: number,
    longitude1: number,
    latitude2: number,
    longitude2: number
  ) {
    // Returns the distance between two GPS points

    let lon1 = (longitude1 * Math.PI) / 180;
    let lon2 = (longitude2 * Math.PI) / 180;
    let lat1 = (latitude1 * Math.PI) / 180;
    let lat2 = (latitude2 * Math.PI) / 180;

    // Haversine formula (https://fr.wikipedia.org/wiki/Formule_de_haversine)
    let distanceLongitude = lon2 - lon1;
    let distanceLatitude = lat2 - lat1;
    let a =
      Math.pow(Math.sin(distanceLatitude / 2), 2) +
      Math.cos(lat1) *
        Math.cos(lat2) *
        Math.pow(Math.sin(distanceLongitude / 2), 2);

    let c = 2 * Math.asin(Math.sqrt(a));

    // Radius of earth in kilometers. Use 3956
    // for miles

    return c * earth_radius;
  }

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

  let angleToNorth: number = 0;

  function angleToNorthPole(
    userPosition: [number, number],
    destination: [number, number]
  ) {
    // Returns the angle between three points in degree : user position(A), north pole(B), destination position(C)
    // Ref: https://math.stackexchange.com/questions/361412/finding-the-angle-between-three-points

    // If the distance between the two points is 0, angle is 0.
    if (
      distance(
        userPosition[0],
        userPosition[1],
        destination[0],
        destination[1]
      ) != 0
    ) {
      let vectorAB: [number, number] = [
        north_pole[0] - userPosition[0],
        north_pole[1] - userPosition[1],
      ];
      let vectorAC: [number, number] = [
        destination[0] - userPosition[0],
        destination[1] - userPosition[1],
      ];

      let normAB = Math.sqrt(
        Math.pow(vectorAB[0], 2) + Math.pow(vectorAB[1], 2)
      );
      let normAC = Math.sqrt(
        Math.pow(vectorAC[0], 2) + Math.pow(vectorAC[1], 2)
      );

      let a = vectorAB[0] * vectorAC[0] + vectorAB[1] * vectorAC[1];
      let b = normAB * normAC;

      let angleInRadian = Math.acos(a / b);

      return angleInRadian * (180 / Math.PI);
    } else {
      return 0;
    }
  }

  const getAngleToNorthPole = () => {
    angleToNorth = angleToNorthPole(watcher_position, object_position);
  };

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
            window.addEventListener(
              "deviceorientation",
              deviceOrientationHandler
            );
            isDeviceOrientationEnabled = true;
            isDeviceOrientationAuthorized = true;
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
      {#if isObjectPositionSet == false}
        <div class="position">
          <h2>Set your destination position :</h2>
          <button on:click={getObjectPosition}>Get your position</button>
          {#if isLatitudeValid == false}
            <p class="error">Latitude must be between -90 and 90 degrees</p>
          {/if}
          <p>
            Latitude : <input
              type="tel"
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
              type="tel"
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
      {:else}
        <div class="position">
          <button on:click={getAngleToNorthPole}>Get angle to north pole</button
          >
          <p>Angle to north pole : {angleToNorth}</p>
        </div>

        <div class="position">
          <button on:click={enableDeviceOrientation}
            >Authorize orientation</button
          >
        </div>
      {/if}

      <p>Compass heading : {compassHeading}</p>

      {#if isObjectPositionSet && isWatcherPositionSet && isDeviceOrientationAuthorized}
        {#if isDeviceOrientationEnabled == true}
          <p>Device orientation : {deviceOrientation}</p>
        {:else}
          <p>Can't access device orientation data.</p>
        {/if}

        <div>
          <a href="https://vitejs.dev" target="_blank" rel="noreferrer">
            <img
              src="/Dark_Green_Arrow_Up.png"
              class="logo"
              alt="Direction to the object"
              style="transform: rotate({(compassHeading - deviceOrientation) + angleToObject}deg)"
            />

          </a>
        </div>

        <p>Image rotation : {(compassHeading - deviceOrientation) + angleToObject} deg</p>

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
    <p>This navigator doesn't have Geolocation. Try another one !</p>
  {/if}
</main>

<style>
  .position {
    justify-content: space-around;
  }

  .error {
    color: rgb(228, 78, 78);
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
