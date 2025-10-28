<script lang="ts">
  import { Popup } from "svelte-maplibre-gl";
  let {
    routePopupLngLat,
    selectedRoute,
    isLoading,
    onClose,
    onSearch,
  } = $props();
</script>

<Popup lnglat={routePopupLngLat} closeButton={true} onclose={onClose}>
  <div class="p-3 max-h-[200px] overflow-y-auto">
    <h3 class="font-bold text-lg mb-2 text-blue-600">🚴 Bike Route</h3>

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

    {#if selectedRoute.properties?.distance}
      <p class="text-sm mb-1">
        <strong>Distance:</strong>
        {selectedRoute.properties.distance} km
      </p>
    {/if}

    <p class="text-xs text-gray-500 mt-2">
      Click location: {routePopupLngLat[1].toFixed(4)}, {routePopupLngLat[0].toFixed(
        4
      )}
    </p>
  </div>

  <!-- Search button -->
  <div class="border-t pt-2">
    <button
      class="w-full bg-blue-500 text-white py-2 px-4 rounded hover:bg-blue-600 transition-colors disabled:bg-gray-400"
      onclick={onSearch}
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

<!-- <style>
    /* Higher specificity - targets button inside this component */
    :global(.maplibregl-popup-content) .search-button {
        background-color: #3b82f6 !important;
        border: none !important;
    }
    
    :global(.maplibregl-popup-content) .search-button:hover:not(:disabled) {
        background-color: #2563eb !important;
    }
    
    :global(.maplibregl-popup-content) .search-button:disabled {
        background-color: #9ca3af !important;
    }
</style> -->
