<script>
  import { onMount } from 'svelte';
  import mapboxgl from 'mapbox-gl';

  mapboxgl.accessToken = 'pk.eyJ1IjoibmF0ZG9zYW4iLCJhIjoiY2xza3huODg4MDh1ZzJpcDVoOTJ1eWFqayJ9.hvQnPf9rwTCV4aok1j7xJA';

  let mapTokyo;
  let isRailwayVisible = false;

  onMount(() => {
    mapTokyo = new mapboxgl.Map({
      container: 'map-tokyo',
      style: 'mapbox://styles/mapbox/navigation-night-v1',
      center: [139.6917, 35.6895],
      zoom: 12
    });

    const popup = new mapboxgl.Popup({
      closeButton: false,
      closeOnClick: false
    });

    mapTokyo.on('load', () => {
      mapTokyo.addSource('tokyo-railways', {
        type: 'geojson',
        data: '/N02-19_RailroadSection.geojson'
      });

      mapTokyo.addLayer({
        id: 'tokyo-railways-layer',
        type: 'line',
        source: 'tokyo-railways',
        layout: {},
        paint: {
          'line-color': '#ADD8E6',
          'line-width': 2,
          'line-opacity': isRailwayVisible ? 1 : 0
        }
      });

      mapTokyo.on('mousemove', 'tokyo-railways-layer', (e) => {
        if (e.features.length > 0) {
          const feature = e.features[0];
          popup.setLngLat(e.lngLat)
               .setText(feature.properties.路線名)
               .addTo(mapTokyo);
        }
      });

      mapTokyo.on('mouseleave', 'tokyo-railways-layer', () => {
        popup.remove();
      });
    });

    new mapboxgl.Marker()
      .setLngLat([139.7528, 35.6852])
      .setPopup(new mapboxgl.Popup().setText('Tokyo'))
      .addTo(mapTokyo);
  });

  function toggleRailwayVisibility() {
    isRailwayVisible = !isRailwayVisible;
    mapTokyo.setPaintProperty('tokyo-railways-layer', 'line-opacity', isRailwayVisible ? 1 : 0);
  }
</script>

<style>
  .maps-container {
    display: flex;
    justify-content: center;
    align-items: stretch;
  }

  #map-tokyo {
    flex: 1;
    height: 100vh;
  }
</style>

<button on:click={toggleRailwayVisibility}>
  {isRailwayVisible ? 'Hide Railway Lines' : 'Show Railway Lines'}
</button>

<div class="maps-container">
  <div id="map-tokyo"></div>
</div>
