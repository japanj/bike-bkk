<script lang="ts">
    let { place, handleNearbyPlaceClick } = $props();
</script>

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
