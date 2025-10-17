<script lang="ts">
    import {
        MapLibre,
        Marker,
        Popup,
        NavigationControl,
        ScaleControl,
        GlobeControl,
        LineLayer,
        GeoJSONSource,
    } from "svelte-maplibre-gl";

    let lnglat: [number, number] = [100.503412, 13.7497695];

    // import bike route data
    const { data } = $props();

    let bike_route = data.bike_route as GeoJSON.FeatureCollection;
    let bike_sharing_loc = data.bike_sharing_loc as GeoJSON.FeatureCollection;
    let bike_route_buffer = data.bike_route_buffer as GeoJSON.FeatureCollection;

    // For route popup
    let routePopupVisible = $state(false);
    let routePopupLngLat = $state<[number, number] | null>(null);
    let selectedRoute = $state<any>(null);
    let mapInstance: any = null;

    // Overpass API data
    let nearbyPlaces = $state<any[]>([]);
    let isLoading = $state(false);
    let searchRadius = $state(500); // meters

    // For place popup
    let selectedPlace = $state<any>(null);
    let placePopupVisible = $state(false);
    let placeLngLat = $state<[number, number] | null>(null);

    // state for enable or disable layer
    let markerVisible = $state(true);
    let lineVisible = $state(true);

    function handleMapLoad(event: any) {
        mapInstance = event.target;
        console.log("Map loaded", mapInstance);
        mapInstance.getCanvas().style.cursor = "pointer";
    }

    function handleMapClick(event: any) {
        if (!mapInstance) return;
        console.log("Map click event:", event);

        const features = mapInstance.queryRenderedFeatures(event.point, {
            layers: ["bike-route-layer"],
        });
        console.log("Features found:", features);

        if (features && features.length > 0) {
            selectedRoute = features[0];
            routePopupLngLat = [event.lngLat.lng, event.lngLat.lat];
            routePopupVisible = true;
        }
    }

    // Overpass API function
    async function searchNearbyPlaces(lat: number, lon: number) {
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
                (feature: any) => feature.properties?.id_bike === routeId,
            );

            if (!bufferFeature) {
                console.error("No buffer found for this route");
                return;
            }

            // Get buffer bounds to create search area
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
                    nwr["tourism"~"museum|gallery|attraction|zoo"](poly:"${swappedCoordinates}");
                    nwr["leisure"~"park"](poly:"${swappedCoordinates}");
                );
                out center;
            `;

            const response = await fetch(
                "https://overpass-api.de/api/interpreter",
                {
                    method: "POST",
                    headers: {
                        "Content-Type": "application/x-www-form-urlencoded",
                    },
                    body: `data=${encodeURIComponent(overpassQuery)}`,
                },
            );

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
                        "unknown",
                    name: element.tags?.name || "Name is not defined",
                }))
                .filter(
                    (place: { lat: any; lon: any }) => place.lat && place.lon,
                );

            console.log("Nearby places found:", nearbyPlaces);
        } catch (error) {
            console.error("Error fetching nearby places:", error);
        } finally {
            isLoading = false;
        }
    }

    // Handle search button click
    function handleSearchNearby() {
        if (routePopupLngLat) {
            searchNearbyPlaces(routePopupLngLat[1], routePopupLngLat[0]);
        }
    }

    // Handle nearby card click
    function handleNearbyPlaceClick(place: any) {
        selectedPlace = place;
        placeLngLat = [place.lon, place.lat];
        placePopupVisible = true;

        if (mapInstance) {
            mapInstance.flyTo({ center: placeLngLat, zoom: 15 });
        }
    }
</script>

<div class="flex items-center gap-x-4 text-sm" s>
    <b>Layer:</b>
    <label>
        <input type="checkbox" bind:checked={markerVisible} /> Bike sharing location
    </label>
    <label>
        <input type="checkbox" bind:checked={lineVisible} /> Bike route
    </label>
</div>

<MapLibre
    class="h-[55vh] min-h-[300px]"
    style="https://basemaps.cartocdn.com/gl/voyager-gl-style/style.json"
    zoom={12}
    center={{ lng: 100.503412, lat: 13.7497695 }}
    onclick={handleMapClick}
    onload={handleMapLoad}
>
    <NavigationControl />
    <ScaleControl />
    <GlobeControl />

    <!-- <Marker bind:lnglat draggable /> -->

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

    {#if markerVisible}
        {#each bike_sharing_loc.features as feature}
            <Marker
                bind:lnglat={feature.geometry.coordinates}
                bind:open={markerVisible}
            >
                <Popup closeButton={true} closeOnClick={false}>
                    <div class="p-2">
                        <h3 class="font-bold text-lg">
                            {feature.properties?.sta_name ??
                                `Station ${index + 1}`}
                        </h3>
                        <p class="text-sm text-gray-600">
                            Location: {feature.geometry.coordinates[1].toFixed(
                                4,
                            )}, {feature.geometry.coordinates[0].toFixed(4)}
                        </p>
                    </div>
                </Popup>
            </Marker>
        {/each}
    {/if}

    <!-- Route popup -->
    {#if routePopupVisible && routePopupLngLat && selectedRoute}
        <Popup
            lnglat={routePopupLngLat}
            closeButton={true}
            onclose={() => {
                routePopupVisible = false;
                selectedRoute = null;
                routePopupLngLat = null;
            }}
        >
            <div class="p-3 max-h-[200px] overflow-y-auto">
                <h3 class="font-bold text-lg mb-2 text-red-600">
                    🚴 Bike Route
                </h3>

                {#if selectedRoute.properties?.name}
                    <p class="text-sm mb-1">
                        <strong>Name:</strong>
                        {selectedRoute.properties.name}
                    </p>
                {/if}

                {#if selectedRoute.properties?.start}
                    <p class="text-sm mb-1">
                        <strong>Starting point:</strong>
                        {selectedRoute.properties.start}
                    </p>
                {/if}

                {#if selectedRoute.properties?.end}
                    <p class="text-sm mb-1">
                        <strong>Ending point:</strong>
                        {selectedRoute.properties.end}
                    </p>
                {/if}

                {#if selectedRoute.properties?.route}
                    <p class="text-sm mb-1">
                        <strong>Route type:</strong>
                        {selectedRoute.properties.route}
                    </p>
                {/if}

                <p class="text-xs text-gray-500 mt-2">
                    Click location: {routePopupLngLat[1].toFixed(4)}, {routePopupLngLat[0].toFixed(
                        4,
                    )}
                </p>
            </div>

            <!-- Search button -->
            <div class="border-t pt-2">
                <button
                    class="w-full bg-blue-500 text-white py-2 px-4 rounded hover:bg-blue-600 transition-colors disabled:bg-gray-400"
                    onclick={handleSearchNearby}
                    disabled={isLoading}
                >
                    {#if isLoading}
                        🔄 Searching...
                    {:else}
                        🔍 Search things nearby
                    {/if}
                </button>
            </div>
        </Popup>
    {/if}

    <!-- Special marker for selected place -->
    {#if selectedPlace && placeLngLat}
        <Marker lnglat={placeLngLat} color="red">
            <!--<div class="bg-yellow-500 text-white rounded-full w-8 h-8 flex items-center justify-center text-sm font-bold border-2 border-yellow-600 shadow-lg animate-pulse">
                ⭐
            </div>-->
            <Popup closeButton={true} closeOnClick={false}>
                <div class="p-3 max-h-[200px] overflow-y-auto">
                    <h3 class="font-bold text-lg mb-2 text-yellow-600">
                        ⭐ Selected Place
                    </h3>
                    <h4 class="font-bold text-blue-600 mb-2">
                        {selectedPlace.name}
                    </h4>
                    <p class="text-sm text-gray-600 capitalize mb-2">
                        {selectedPlace.facility}
                    </p>

                    {#if selectedPlace.tags.phone}
                        <p class="text-sm mb-2">
                            📞 {selectedPlace.tags.phone}
                        </p>
                    {/if}

                    {#if selectedPlace.tags.opening_hours}
                        <p class="text-sm mb-2">
                            🕒 {selectedPlace.tags.opening_hours}
                        </p>
                    {/if}

                    {#if selectedPlace.tags.website}
                        <p class="text-sm mb-2">
                            🌐 <a
                                href={selectedPlace.tags.website}
                                target="_blank"
                                class="text-blue-500 hover:underline"
                                >Visit Website</a
                            >
                        </p>
                    {/if}

                    {#if selectedPlace.tags.cuisine}
                        <p class="text-sm mb-2">
                            🍽️ {selectedPlace.tags.cuisine}
                        </p>
                    {/if}

                    <p class="text-xs text-gray-500 mt-3">
                        📍 {selectedPlace.lat.toFixed(4)}, {selectedPlace.lon.toFixed(
                            4,
                        )}
                    </p>

                    <div class="border-t pt-2 mt-3">
                        <button
                            class="w-full bg-red-500 text-white py-2 px-4 rounded hover:bg-red-600 transition-colors"
                            onclick={() => {
                                selectedPlace = null;
                                placeLngLat = null;
                                placePopupVisible = false;
                            }}
                        >
                            Clear Selection
                        </button>
                    </div>
                </div>
            </Popup>
        </Marker>
    {/if}
</MapLibre>

<!-- Results panel below the map -->
{#if nearbyPlaces.length > 0}
    <div class="mt-4 p-4 bg-gray-50 rounded-lg">
        <h3 class="font-bold text-lg mb-3">
            📍 Nearby Places ({nearbyPlaces.length})
        </h3>
        <div
            class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3 max-h-60 overflow-y-auto"
        >
            {#each nearbyPlaces as place}
                <button
                    class="bg-white p-3 rounded border border-gray-200 hover:border-blue-300 hover:bg-blue-50 transition-colors text-left w-full"
                    onclick={() => handleNearbyPlaceClick(place)}
                >
                    <!-- <div class="bg-white p-3 rounded border border-gray-200"> -->
                    <h4 class="font-semibold text-blue-600 mb-1">
                        {place.name}
                    </h4>
                    <p class="text-sm text-gray-600 capitalize mb-1">
                        {place.facility}
                    </p>
                    <!-- <p class="text-xs text-gray-500">{Math.round(place.distance)}m away</p> -->
                    {#if place.tags.phone}
                        <p class="text-xs mt-1">📞 {place.tags.phone}</p>
                    {/if}
                </button>
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
