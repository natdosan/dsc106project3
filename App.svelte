<script>
  import { onMount } from 'svelte';
  import mapboxgl from 'mapbox-gl';

  // Your Mapbox access token
  mapboxgl.accessToken = 'pk.eyJ1IjoibmF0ZG9zYW4iLCJhIjoiY2xza3huODg4MDh1ZzJpcDVoOTJ1eWFqayJ9.hvQnPf9rwTCV4aok1j7xJA';

  onMount(() => {
    const shanghai = {
      lng: 121.4737,
      lat: 31.2304,
      ridershipToPopulationRatio: '0.416' // Calculated previously
    };

    const newYork = {
      lng: -74.0060,
      lat: 40.7128,
      ridershipToPopulationRatio: '0.238' // Calculated previously
    };

    const mapShanghai = new mapboxgl.Map({
      container: 'map-shanghai',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [shanghai.lng, shanghai.lat],
      zoom: 12
    });

    new mapboxgl.Marker()
      .setLngLat([shanghai.lng, shanghai.lat])
      .setPopup(new mapboxgl.Popup().setText(`Ridership to Population Ratio: ${shanghai.ridershipToPopulationRatio}`))
      .addTo(mapShanghai);

    const mapNewYork = new mapboxgl.Map({
      container: 'map-new-york',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [newYork.lng, newYork.lat],
      zoom: 12
    });

    new mapboxgl.Marker()
      .setLngLat([newYork.lng, newYork.lat])
      .setPopup(new mapboxgl.Popup().setText(`Ridership to Population Ratio: ${newYork.ridershipToPopulationRatio}`))
      .addTo(mapNewYork);
  });
</script>

<style>
  .maps-container {
    display: flex;
    justify-content: center;
    align-items: stretch; /* This ensures that the maps fill the container vertically */
  }

  #map-shanghai, #map-new-york {
    flex: 1; /* This allows each map to take up equal space */
    height: 100vh; /* Adjust the height as needed */
  }
</style>

<div class="maps-container">
  <div id="map-shanghai"></div>
  <div id="map-new-york"></div>
</div>
