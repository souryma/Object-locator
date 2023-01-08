<script context="module" lang="ts">
  const earth_radius = 6371;

  export function distance(
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
</script>
