<script lang="ts">
  import { Popup } from "svelte-maplibre-gl";
  let {
    highlightedStation = $bindable(),
    isMarkerHighlighted = $bindable(),
  } = $props();
</script>

<Popup
  lnglat={highlightedStation.geometry.coordinates}
  closeButton={true}
  bind:open={isMarkerHighlighted}
  onclose={() => {
    isMarkerHighlighted = false;
    // highlightedStation = null;
  }}
>
  <div class="p-2">
    <h3 class="font-bold text-lg">
      {highlightedStation.properties?.Name} - {highlightedStation.properties?.location}
    </h3>
    <p class="text-base">
      จำนวนจักรยาน: {highlightedStation.properties?.num_bike} คัน
    </p>
    <p class="text-sm text-gray-600">
      Location: {highlightedStation.geometry.coordinates[1].toFixed(4)}, {highlightedStation.geometry.coordinates[0].toFixed(
        4
      )}
    </p>
  </div>
</Popup>
