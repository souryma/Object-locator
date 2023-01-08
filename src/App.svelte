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
  let arrowColor: string = "logoGrey";
  let isOnMobile: boolean = false;

  window.onload = function () {
    checkDeviceType();
  };

  // Check if the device is a mobile
  // From https://stackoverflow.com/questions/11381673/detecting-a-mobile-browser
  function checkDeviceType() {
    (function (a) {
      if (
        /(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino/i.test(
          a
        ) ||
        /1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(
          a.substr(0, 4)
        )
      )
        isOnMobile = true;
    })(navigator.userAgent || navigator.vendor || window.opera);
  }

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

  const updateArrowColor = () => {
    let imageRotation: number =
      deviceOrientation + (angleToObject - angleToNorth);
    if (
      (imageRotation < 5 && imageRotation > -5) ||
      (imageRotation < 365 && imageRotation > 355)
    ) {
      arrowColor = "logoGreen";
    } else {
      arrowColor = "logoGrey";
    }
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
    clear = setInterval(updateArrowColor, 250);
  }

  if ("geolocation" in navigator || isOnMobile) {
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
  {#if isOnMobile}
    {#if "geolocation" in navigator && "DeviceOrientationEvent" in window}
      <div class="card">
        {#if isDeviceOrientationAuthorized == false}
          <h2>Authorize access to your orientation :</h2>
          <button on:click={enableCompassHeading}
            >Give my orientation</button
          >
        {:else if isObjectPositionSet == false}
          <div class="position">
            <h2>Set your destination position :</h2>
            <button on:click={getObjectPosition}>My destination is here</button>
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
              <p class="error">
                Longitude must be between -180 and 180 degrees
              </p>
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
            <p class="error">Can't access device orientation data. The arrow will show you the right direction if your device is
              heading north.</p>
          {/if}

          <div>
            <img
              src="/simpleGPS_arrow.svg"
              class={arrowColor}
              alt="Direction to the object"
              style="transform: rotate({deviceOrientation +
                (angleToObject - angleToNorth)}deg)"
            />
          </div>

          <div class="distance">
            <p>
              Distance to your destination : {#if distanceInKM != 0}{distanceInKM} Km
                /
              {/if}{distanceInM} m
            </p>
          </div>
          <div class="orientation">
            <p>Direction : {direction}</p>
          </div>
        {/if}
      </div>
    {:else}
      <p class="error">
        This navigator doesn't have Geolocation or device orientation. Try
        another one !
      </p>
    {/if}
  {:else}
    <p class="error">This app only works on mobile !</p>
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
  .logoGrey {
    height: 13em;
    filter: invert(65%) saturate(500%) hue-rotate(86deg) brightness(118%)
      contrast(119%);
  }
  .logoGreen {
    height: 13em;
    filter: invert(48%) sepia(79%) saturate(2476%) hue-rotate(86deg)
      brightness(118%) contrast(119%);
  }
</style>
