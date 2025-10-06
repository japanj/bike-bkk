<script lang="ts">
    import {
        MapLibre,
        Marker,
        Popup,
        NavigationControl,
        ScaleControl,
        GlobeControl,
        LineLayer,
        GeoJSONSource
    } from 'svelte-maplibre-gl';

    let lnglat: [number, number] = [100.503412, 13.7497695];
    
    // import bike route data
    export let data;
    let bike_route = data.bike_route as GeoJSON.FeatureCollection;
    let bike_sharing_loc = data.bike_sharing_loc as GeoJSON.FeatureCollection;

    // For route popup
    let routePopupVisible = false;
    let routePopupLngLat: [number, number] | null = null;
    let selectedRoute: any = null;
    let mapInstance: any = null;

    // Overpass API data
    let nearbyPlaces: any[] = [];
    let isLoading = false;
    let searchRadius = 500; // meters
    
    function handleMapLoad(event: any) {
        mapInstance = event.target;
        console.log('Map loaded', mapInstance);
        mapInstance.getCanvas().style.cursor = 'pointer';
    }

    function handleMapClick(event: any) {
        if (!mapInstance) return;
        console.log('Map click event:', event);

        const features = mapInstance.queryRenderedFeatures(event.point, {
            layers: ['bike-route-layer']
        });
        console.log('Features found:', features);
        
        if (features && features.length > 0) {
            selectedRoute = features[0];
            routePopupLngLat = [event.lngLat.lng, event.lngLat.lat];
            routePopupVisible = true;
        }
    }

    // Overpass API function
    async function searchNearbyPlaces(lat: number, lon: number) {
        isLoading = true;
        nearbyPlaces = [];
        
        try {
            const overpassQuery = `
                [out:json][timeout:50];
                (
                    nwr["amenity"~"restaurant|cafe|convenience"](around:${searchRadius},${lat},${lon});
                    nwr["tourism"~"museum|gallery|attraction|zoo"](around:${searchRadius},${lat},${lon});
                    nwr["leisure"~"park"](around:${searchRadius},${lat},${lon});
                );
                out center meta;
            `;

            const response = await fetch('https://overpass-api.de/api/interpreter', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded'
                },
                body: `data=${encodeURIComponent(overpassQuery)}`
            });

            const data = await response.json();

            console.log('Overpass API response:', data);
            
            // Process the data
            nearbyPlaces = data.elements.map((element: any) => ({
                id: element.id,
                type: element.type,
                lat: element.lat || element.center?.lat,
                lon: element.lon || element.center?.lon,
                tags: element.tags || {},
                facility: element.tags?.amenity || element.tags?.tourism || element.tags?.leisure || 'unknown',
                name: element.tags?.name || 'Name is not defined',
            })).filter((place: { lat: any; lon: any; }) => place.lat && place.lon);

            console.log('Nearby places found:', nearbyPlaces);
            
        } catch (error) {
            console.error('Error fetching nearby places:', error);
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
</script>

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
                'line-color': '#ff0000',
                'line-width': 3
            }}
        >
        </LineLayer>
    </GeoJSONSource>

    {#each bike_sharing_loc.features as feature}
        <Marker bind:lnglat={feature.geometry.coordinates}>
            <Popup closeButton={true} closeOnClick={false}>
                <div class="p-2">
                    <h3 class="font-bold text-lg">
                        {feature.properties?.sta_name ?? `Station ${index + 1}`}
                    </h3>
                    <p class="text-sm text-gray-600">
                        Location: {feature.geometry.coordinates[1].toFixed(4)}, {feature.geometry.coordinates[0].toFixed(4)}
                    </p>
                </div>
            </Popup>
        </Marker>
    {/each}

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
            <div class="p-3 min-w-[200px]">
                <h3 class="font-bold text-lg mb-2 text-red-600">🚴 Bike Route</h3>
                
                {#if selectedRoute.properties?.name}
                    <p class="text-sm mb-1">
                        <strong>Name:</strong> {selectedRoute.properties.name}
                    </p>
                {/if}
                
                {#if selectedRoute.properties?.start}
                    <p class="text-sm mb-1">
                        <strong>Starting point:</strong> {selectedRoute.properties.start}
                    </p>
                {/if}
                
                {#if selectedRoute.properties?.end}
                    <p class="text-sm mb-1">
                        <strong>Ending point:</strong> {selectedRoute.properties.end}
                    </p>
                {/if}
                
                {#if selectedRoute.properties?.route}
                    <p class="text-sm mb-1">
                        <strong>Route type:</strong> {selectedRoute.properties.route}
                    </p>
                {/if}

                <p class="text-xs text-gray-500 mt-2">
                    Click location: {routePopupLngLat[1].toFixed(4)}, {routePopupLngLat[0].toFixed(4)}
                </p>
            </div>

           <!-- Search button -->
            <div class="border-t pt-2">
                <button 
                    class="w-full bg-blue-500 text-white py-2 px-4 rounded hover:bg-blue-600 transition-colors disabled:bg-gray-400"
                    on:click={handleSearchNearby}
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
</MapLibre>

<!-- Results panel below the map -->
{#if nearbyPlaces.length > 0}
    <div class="mt-4 p-4 bg-gray-50 rounded-lg">
        <h3 class="font-bold text-lg mb-3">📍 Nearby Places ({nearbyPlaces.length})</h3>
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3 max-h-60 overflow-y-auto">
            {#each nearbyPlaces as place}
                <div class="bg-white p-3 rounded border border-gray-200">
                    <h4 class="font-semibold text-blue-600 mb-1">{place.name}</h4>
                    <p class="text-sm text-gray-600 capitalize mb-1">{place.facility}</p>
                    <!-- <p class="text-xs text-gray-500">{Math.round(place.distance)}m away</p> -->
                    {#if place.tags.phone}
                        <p class="text-xs mt-1">📞 {place.tags.phone}</p>
                    {/if}
                </div>
            {/each}
        </div>
    </div>
{/if}

<p>Longitude: {lnglat[0]}</p>
<p>Latitude: {lnglat[1]}</p>

<!-- {#each bike_sharing_loc.features as feature}
        <p>Bike Station: {(feature.properties?.sta_name ?? 'Unnamed')} at {feature.geometry.coordinates}</p>
{/each} -->
<!-- <p>Data: {JSON.stringify(data)}</p> -->