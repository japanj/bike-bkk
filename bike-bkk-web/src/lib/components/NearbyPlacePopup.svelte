<script lang="ts">
    import {Popup, Marker} from 'svelte-maplibre-gl';
    let {selectedPlace, placeLngLat, placePopupVisible} = $props();
</script>

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
