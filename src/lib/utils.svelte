<script context="module" lang="ts">
  export function checkLatitude(latitude: number) {
    // Returns true if the given number is between -90 and 90
    if (latitude <= 90 && -90 <= latitude) {
      return true;
    } else {
      return false;
    }
  }

  export function checkLongitude(longitude: number) {
    // Returns true if the given number is between -180 and 180
    if (longitude <= 180 && -180 <= longitude) {
      return true;
    } else {
      return false;
    }
  }
</script>
