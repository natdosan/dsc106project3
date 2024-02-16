<script>
  import { onMount } from 'svelte';
  import mapboxgl from 'mapbox-gl';
  //import mapboxgl.css;


  mapboxgl.accessToken = 'pk.eyJ1IjoibmF0ZG9zYW4iLCJhIjoiY2xza3huODg4MDh1ZzJpcDVoOTJ1eWFqayJ9.hvQnPf9rwTCV4aok1j7xJA';

  onMount(() => {
    // Initialize the map for Tokyo with an increased zoom level
    const mapTokyo = new mapboxgl.Map({
      container: 'map-tokyo', // Make sure to update the container ID
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [139.6917, 35.6895], // Tokyo coordinates
      zoom: 12 // Adjust the zoom level as needed
    });

    const popup = new mapboxgl.Popup({
      closeButton: false,
      closeOnClick: false
    });

    mapTokyo.on('load', () => {
      mapTokyo.addSource('tokyo-railways', {
        type: 'geojson',
        // Assuming Tokyo.geojson is in the public folder
        data: '/N02-19_RailroadSection.geojson'
      });

      mapTokyo.addLayer({
        id: 'tokyo-railways-layer',
        type: 'line',
        source: 'tokyo-railways',
        layout: {},
        paint: {
          'line-color': '#FF0000', // Example: Red lines for railways
          'line-width': 2
        }
      });

      // When the user moves their mouse over the layer, update the
      // feature state for the feature under the mouse
      mapTokyo.on('mousemove', 'tokyo-railways-layer', (e) => {
        console.log("Hovering over:", e.features[0]); // Debugging line
        if (e.features.length > 0) {
          const feature = e.features[0];
          // Set the popup's content based on the feature's properties
          popup.setLngLat(e.lngLat)
               .setText(feature.properties.路線名) // Assuming '路線名' is the field name in your GeoJSON
               .addTo(mapTokyo);
        }
      });

      // Reset the cursor style and remove the popup when the mouse leaves the layer
      mapTokyo.on('mouseleave', 'tokyo-railways-layer', () => {
        popup.remove();
      });
    });

    // Add a marker for Tokyo (optional)
    new mapboxgl.Marker()
      .setLngLat([139.7528, 35.6852])
      .setPopup(new mapboxgl.Popup().setText('Tokyo'))
      .addTo(mapTokyo);

    // Initialize the map for New York with an increased zoom level
    const mapNewYork = new mapboxgl.Map({
      container: 'map-new-york',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [-74.0060, 40.7128], // New York coordinates
      zoom: 12 // Adjust the zoom level as needed
    });

    mapNewYork.on('load', () => {
      mapNewYork.addSource('nyc-subways', {
        type: 'geojson',
        data: '/nyc.geojson' 
      });

      mapNewYork.addLayer({
        id: 'nyc-subway-lines',
        type: 'line',
        source: 'nyc-subways',
        paint: {
          'line-width': 2,
          'line-color': '#0000FF' // Example: Blue lines for NYC subways
        }
      });
    });

    // Add a marker for New York (optional)
    new mapboxgl.Marker()
      .setLngLat([-74.0060, 40.7128])
      .setPopup(new mapboxgl.Popup().setText('New York'))
      .addTo(mapNewYork);
  });
</script>

<style>
  .maps-container {
    display: flex;
    justify-content: center;
    align-items: stretch;
  }

  #map-tokyo, #map-new-york {
    flex: 1;
    height: 100vh; /* Adjust the height as needed */
  }
</style>

<div class="maps-container">
  <div id="map-tokyo"></div> <!-- Update the ID here -->
  <div id="map-new-york"></div>
</div>
