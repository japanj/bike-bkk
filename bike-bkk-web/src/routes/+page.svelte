<script lang="ts">
    import {
        MapLibre,
        Marker,
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

</script>

<MapLibre
    class="h-[55vh] min-h-[300px]"
    style="https://basemaps.cartocdn.com/gl/voyager-gl-style/style.json"
    zoom={12}
    center={{ lng: 100.503412, lat: 13.7497695 }}
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
        />
    </GeoJSONSource>

    {#each bike_sharing_loc.features as feature}
        <Marker bind:lnglat={feature.geometry.coordinates}>
        </Marker>
    {/each}
</MapLibre>

<p>
    Longitude:
    <input type="number" bind:value={lnglat[0]} />
</p>
<p>
    Latitude:
    <input type="number" bind:value={lnglat[1]} />
</p>
{#each bike_sharing_loc.features as feature}
        <p>Bike Station: {feature.properties.sta_name || 'Unnamed'} at {feature.geometry.coordinates}</p>
{/each}
<!-- <p>Data: {JSON.stringify(data)}</p> -->