<script lang="ts">
    import { Marker, Popup } from "svelte-maplibre-gl";
    let { feature, index, markerVisible } = $props();
</script>

<Marker bind:lnglat={feature.geometry.coordinates} bind:open={markerVisible}>
  <Popup closeButton={true} closeOnClick={false}>
    <div class="p-2">
      <h3 class="font-bold text-lg">
        {feature.properties?.sta_name ?? `Station ${index + 1}`}
      </h3>
      <p class="text-sm text-gray-600">
        Location: {feature.geometry.coordinates[1].toFixed(4)}, {feature.geometry.coordinates[0].toFixed(
          4
        )}
      </p>
    </div>
  </Popup>
</Marker>
