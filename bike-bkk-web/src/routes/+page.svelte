<script lang="ts">
  import {
    MapLibre,
    NavigationControl,
    ScaleControl,
    GlobeControl,
    LineLayer,
    GeoJSONSource,
    CircleLayer,
  } from "svelte-maplibre-gl";
  import RoutePopup from "$lib/components/RoutePopup.svelte";
  import NearbyPlacePopup from "$lib/components/NearbyPlacePopup.svelte";
  import NearbyPlaceCard from "$lib/components/NearbyPlaceCard.svelte";
  import BikeSharingPopup from "$lib/components/BikeSharingPopup.svelte";

  let lnglat: [number, number] = [100.503412, 13.7497695];

  // import data
  const { data } = $props();

  let bike_route = data.bike_route as GeoJSON.FeatureCollection;
  let bike_sharing_loc = data.bike_sharing_loc as GeoJSON.FeatureCollection;
  let bike_route_buffer = data.bike_route_buffer as GeoJSON.FeatureCollection;

  // For route popup
  let routePopupVisible = $state(false);
  let routePopupLngLat = $state<[number, number] | null>(null);
  let selectedRoute = $state<any>(null);
  let highlightedRoute = $state<any>(null);
  let mapInstance: any = null;

  // Overpass API data
  let nearbyPlaces = $state<any[]>([]);
  let isLoading = $state(false);

  // For place popup
  let selectedPlace = $state<any>(null);
  let placePopupVisible = $state(false);
  let placeLngLat = $state<[number, number] | null>(null);

  // state for enable or disable layer
  let markerVisible = $state(true);
  let lineVisible = $state(true);

  let isMarkerHighlighted = $state(false);
  let highlightedStation = $state<any>(null);

  function handleMapLoad(event: any) {
    mapInstance = event.target;
    console.log("Map loaded", mapInstance);
    mapInstance.getCanvas().style.cursor = "pointer";
  }

  function handleMapClick(event: any) {
    if (!mapInstance) return;
    console.log("Map click event:", event);

    // Bike station click handling
    const stationFeatures = mapInstance.queryRenderedFeatures(event.point, {
      layers: ["bike-stations-layer"],
    });

    if (stationFeatures && stationFeatures.length > 0) {
      console.log("Station clicked:", stationFeatures[0]);
      highlightedStation = stationFeatures[0];
      isMarkerHighlighted = true;
    } else {
      // Clear highlight when clicking elsewhere
      // highlightedStation = null;
      isMarkerHighlighted = false;
    }

    // Bike route click handling
    const features = mapInstance.queryRenderedFeatures(event.point, {
      layers: ["bike-route-layer"],
    });
    console.log("Features found:", features);

    if (features && features.length > 0) {
      selectedRoute = features[0];
      highlightedRoute = features[0];
      routePopupLngLat = [event.lngLat.lng, event.lngLat.lat];
      routePopupVisible = true;
    } else {
      // Clear highlight when clicking elsewhere
      highlightedRoute = null;
      routePopupVisible = false;
      // selectedRoute = null;
      routePopupLngLat = null;
    }
  }

  // searchNearbyPlaces function (calling Overpass API)
  async function searchNearbyPlaces() {
    if (!selectedRoute || !bike_route_buffer) {
      console.error("No route selected or buffer data not available");
      return;
    }

    isLoading = true;
    nearbyPlaces = [];

    try {
      // Find the corresponding buffer polygon for the selected route
      const routeId = selectedRoute.properties?.id_bike;
      console.log("Selected route ID:", routeId);
      const bufferFeature = bike_route_buffer.features.find(
        (feature: any) => feature.properties?.id_bike === routeId
      );

      if (!bufferFeature) {
        console.error("No buffer found for this route");
        return;
      }

      // Get buffer bounds to create search area (buffer is set to 500 meters)
      const coordinates = bufferFeature.geometry.coordinates[0];
      console.log("Buffer coordinates:", coordinates);

      const swappedCoordinates = coordinates
        .map((coord: [number, number]) => `${coord[1]} ${coord[0]}`)
        .join(" ");
      console.log("Buffer bounds (lat lon format):", swappedCoordinates);

      const overpassQuery = `
                [out:json][timeout:120];
                (
                    nwr["amenity"~"restaurant|cafe|convenience"](poly:"${swappedCoordinates}");
                    nwr["tourism"~"museum|gallery|attraction|zoo|viewpoint"](poly:"${swappedCoordinates}");
                    nwr["leisure"~"park|nature_reserve"](poly:"${swappedCoordinates}");
                    nwr["shop"~"bakery|coffee|food|ice_cream|pastry|department_store"](poly:"${swappedCoordinates}");
                    nwr["building"~"religious"](poly:"${swappedCoordinates}");
                );
                out center;
            `;

      const response = await fetch("https://overpass-api.de/api/interpreter", {
        method: "POST",
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
        body: `data=${encodeURIComponent(overpassQuery)}`,
      });

      const data = await response.json();

      console.log("Overpass API response:", data);

      // Process the data
      nearbyPlaces = data.elements
        .map((element: any) => ({
          id: element.id,
          type: element.type,
          lat: element.lat || element.center?.lat,
          lon: element.lon || element.center?.lon,
          tags: element.tags || {},
          facility:
            element.tags?.amenity ||
            element.tags?.tourism ||
            element.tags?.leisure ||
            element.tags?.shop ||
            element.tags?.building ||
            "unknown",
          name:
            element.tags?.name ||
            element.tags?.["name:en"] ||
            "Name is not defined",
        }))
        .filter((place: { lat: any; lon: any }) => place.lat && place.lon);

      console.log("Nearby places found:", nearbyPlaces);
    } catch (error) {
      console.error("Error fetching nearby places:", error);
    } finally {
      isLoading = false;
    }
  }

  // Search button click
  function handleSearchNearby() {
    if (routePopupLngLat) {
      searchNearbyPlaces();
    }
  }

  // Handle nearby place card click
  function handleNearbyPlaceClick(place: any) {
    selectedPlace = place;
    placeLngLat = [place.lon, place.lat];
    placePopupVisible = true;

    if (mapInstance) {
      mapInstance.flyTo({ center: placeLngLat, zoom: 15 });
    }
  }
</script>

<div class="flex items-center gap-x-4 text-base">
  <b>Layer:</b>
  <label>
    <input type="checkbox" bind:checked={markerVisible} /> Bike Sharing Location (จุดจอดจักรยานสาธารณะ)
  </label>
  <label>
    <input type="checkbox" bind:checked={lineVisible} /> Bike Route (เส้นทางจักรยาน)
  </label>
</div>

<br />

<MapLibre
  class="h-[55vh] min-h-[300px]"
  style="https://basemaps.cartocdn.com/gl/voyager-gl-style/style.json"
  zoom={12}
  center={lnglat}
  onclick={handleMapClick}
  onload={handleMapLoad}
>
  <NavigationControl />
  <ScaleControl />
  <GlobeControl />

  <!-- Bike route layer -->
  <GeoJSONSource data={bike_route} id="bike-route-source">
    <LineLayer
      id="bike-route-layer"
      paint={{
        "line-color": "#ff0000",
        "line-width": 3,
      }}
      layout={{
        visibility: lineVisible ? "visible" : "none",
      }}
    ></LineLayer>
  </GeoJSONSource>

  <!-- Highlighted route layer -->
  {#if highlightedRoute && lineVisible}
    <GeoJSONSource
      data={{
        type: "FeatureCollection",
        features: [highlightedRoute],
      }}
      id="highlighted-route-source"
    >
      <LineLayer
        id="highlighted-route-layer"
        paint={{
          "line-color": "#00ff00",
          "line-width": 6,
          "line-opacity": 0.8,
        }}
      />
    </GeoJSONSource>
  {/if}

  <!-- Bike sharing locations layer -->
  <GeoJSONSource data={bike_sharing_loc} id="bike-stations-source">
    <!-- Regular bike stations -->
    <CircleLayer
      id="bike-stations-layer"
      paint={{
        "circle-radius": 8,
        "circle-color": "#3b82f6",
        "circle-stroke-width": 2,
        "circle-stroke-color": "#ffffff",
      }}
      layout={{
        visibility: markerVisible ? "visible" : "none",
      }}
    />

    <!-- Highlighted station (orange) -->
    {#if isMarkerHighlighted && markerVisible}
      <CircleLayer
        id="bike-stations-highlighted-layer"
        paint={{
          "circle-radius": 12,
          "circle-color": "#f97316",
          "circle-stroke-width": 3,
          "circle-stroke-color": "#ffffff",
          "circle-opacity": 0.8,
        }}
        filter={[
          "==",
          ["get", "Name"],
          highlightedStation.properties?.Name || "",
        ]}
      />
    {/if}
  </GeoJSONSource>

  <!-- Bike sharing popup -->
  {#if isMarkerHighlighted && markerVisible}
    <BikeSharingPopup
      {highlightedStation}
      {isMarkerHighlighted}
    />
  {/if}

  <!-- Route popup -->
  {#if routePopupVisible && routePopupLngLat && selectedRoute}
    <RoutePopup
      {routePopupLngLat}
      {selectedRoute}
      {isLoading}
      onClose={() => {
        routePopupVisible = false;
        // selectedRoute = null;
        routePopupLngLat = null;
      }}
      onSearch={handleSearchNearby}
    />
  {/if}

  <!-- Special marker for selected place -->
  {#if selectedPlace && placeLngLat}
    <NearbyPlacePopup {selectedPlace} bind:placeLngLat bind:placePopupVisible />
  {/if}
</MapLibre>

<!-- Results panel below the map -->
{#if nearbyPlaces.length > 0}
  <div class="mt-4 p-4 bg-gray-50 rounded-lg">
    <h3 class="font-bold text-lg mb-3">
      📍 Nearby Places of {selectedRoute.properties?.name} ({nearbyPlaces.length})
    </h3>
    <div
      class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3 max-h-60 overflow-y-auto"
    >
      {#each nearbyPlaces as place}
        <NearbyPlaceCard {place} {handleNearbyPlaceClick} />
      {/each}
    </div>
  </div>
{/if}

<!-- <p>Longitude: {lnglat[0]}</p>
<p>Latitude: {lnglat[1]}</p> -->

<!-- {#each bike_sharing_loc.features as feature}
        <p>Bike Station: {(feature.properties?.sta_name ?? 'Unnamed')} at {feature.geometry.coordinates}</p>
{/each} -->
<!-- <p>Data: {JSON.stringify(data)}</p> -->
