<script>
  import { onMount } from 'svelte';
  import mapboxgl from 'mapbox-gl';

  let mapTokyo, mapNewYork;
  let showMetroLines = true;
  mapboxgl.accessToken = 'pk.eyJ1IjoibmF0ZG9zYW4iLCJhIjoiY2xza3huODg4MDh1ZzJpcDVoOTJ1eWFqayJ9.hvQnPf9rwTCV4aok1j7xJA';

  onMount(() => {
    // set metro lines to not show by default
    const initialVisibility = showMetroLines ? 'visible' : 'none'; // Determine initial visibility

    // Initialize the map for Tokyo with an increased zoom level
    const mapTokyo = new mapboxgl.Map({
      container: 'map-tokyo', // Make sure to update the container ID
      style: 'mapbox://styles/mapbox/navigation-night-v1',
      center: [139.6917, 35.6895], // Tokyo coordinates
      zoom: 12 // Adjust the zoom level as needed
    });

    // Initialize the map for New York with an increased zoom level
    const mapNewYork = new mapboxgl.Map({
      container: 'map-new-york',
      style: 'mapbox://styles/mapbox/navigation-night-v1',
      center: [-74.0060, 40.7128], // New York coordinates
      zoom: 12 // Adjust the zoom level as needed
    });

    const popup = new mapboxgl.Popup({
      closeButton: false,
      closeOnClick: false
    });

    // Add Tokyo Metro Lines
    mapTokyo.on('load', () => {
      mapTokyo.addSource('tokyo-railways', {
        type: 'geojson',
        data: '/N02-19_RailroadSection.geojson'
      });

      mapTokyo.addLayer({
        id: 'tokyo-railways-layer',
        type: 'line',
        source: 'tokyo-railways',
        layout: {
          'visibility': initialVisibility // Set initial visibility based on showMetroLines
        },
        paint: {
          'line-color': '#0000FF', // Example: Red lines for railways
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
    // Add NYC Metro Lines
    mapNewYork.on('load', () => {
      mapNewYork.addSource('nyc-subways', {
        type: 'geojson',
        data: '/nyc.geojson' 
      });

      mapNewYork.addLayer({
        id: 'nyc-subway-lines',
        type: 'line',
        source: 'nyc-subways',
        layout: {
          'visibility': initialVisibility // Set initial visibility based on showMetroLines
        },
        paint: {
          'line-width': 2,
          'line-color': '#0000FF' // Example: Blue lines for NYC subways
        }
      });
    });

    // Add a marker for Tokyo (optional)
    new mapboxgl.Marker()
      .setLngLat([139.7528, 35.6852])
      .setPopup(new mapboxgl.Popup().setText('Tokyo'))
      .addTo(mapTokyo);

    // Add a marker for Shibuya (optional)
    new mapboxgl.Marker()
      .setLngLat([139.70199, 35.65803])
      .setPopup(new mapboxgl.Popup().setText('Daily Ridership: 1,090,000'))
      .addTo(mapTokyo);

    // Add a marker for New York (optional)
    new mapboxgl.Marker()
      .setLngLat([-74.0060, 40.7128])
      .setPopup(new mapboxgl.Popup().setText('New York'))
      .addTo(mapNewYork);

    const stations = [
      { city: 'New York City', name: 'Grand Central Station', coordinates: [-73.977229, 40.752726], ridership: '750,000' },
      { city: 'New York City', name: 'WTC Station', coordinates: [-74.009531, 40.712646], ridership: '300,000' },
      { city: 'Tokyo', name: 'Shibuya Station', coordinates: [139.70199, 35.658033], ridership: '1,090,000' },
      { city: 'Tokyo', name: 'Shinjuku Station', coordinates: [139.700464, 35.689607], ridership: '3,500,000' },
      { city: 'Tokyo', name: 'Tokyo Station', coordinates: [139.767125, 35.681236], ridership: '800,000' },
    ];

    // Markers for selected Stations
    stations.forEach(station => {
      const el = document.createElement('div');
      el.className = 'marker';
      // Optionally set a marker icon through el.innerHTML or className

      const popup = new mapboxgl.Popup({ offset: 25 })
        .setHTML(`<h3>${station.name}</h3><p>Daily Ridership: ${station.ridership}</p>`);

      // Determine which map to add the marker to based on the station's city
      const targetMap = station.city === 'Tokyo' ? mapTokyo : mapNewYork;

      new mapboxgl.Marker(el)
        .setLngLat(station.coordinates)
        .setPopup(popup) // Add this popup to the marker
        .addTo(targetMap); // Use targetMap here instead of map
    });

    // Reactive statement to toggle metro line visibility
    $: {
      //console.log('Toggling metro lines:', showMetroLines);
      if (mapTokyo && mapNewYork) { // Ensure maps are initialized
        const visibility = showMetroLines ? 'visible' : 'none';
        // Check if the layers exist before trying to set their visibility
        if (mapTokyo.getLayer('tokyo-railways-layer')) {
          mapTokyo.setLayoutProperty('tokyo-railways-layer', 'visibility', visibility);
        }
        if (mapNewYork.getLayer('nyc-subway-lines')) {
          mapNewYork.setLayoutProperty('nyc-subway-lines', 'visibility', visibility);
        }
      }
    }
  });
</script>

<label>
  <input type="checkbox" bind:checked={showMetroLines}>
  Show Metro Lines
</label>

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
