<template>
  <div class="top-menu">
    <div class="flex items-center justify-center w-10 h-full mt-auto cursor-pointer"  >
      <img
        src="/src/assets/icons/bars-solid.svg"
        alt="Toogle filter"
        class="w-5 h-5 xl:w-6 xl:h-6 filter-icon"
      />
    </div>
    <div class="font-bold transition-colors w-fit text-breakpoints">
      Meshtastic
    </div>
    <div class="flex flex-wrap ml-auto mr-3 text-blue-500 text-breakpoints">
    </div>
  </div>
  <div class="flex flex-col w-full h-full mt-10 xl:flex-row"  >
    <div id="charts" class="w-full h-1/2 xl:h-full">
      Chart
    </div>
  </div>
</template>

<script setup>
import { ref, unref, shallowRef, computed, onMounted } from 'vue'
import { useServer } from './servers.js'


const devices = ref()
const server = useServer()

const id = 500021556


const fetchDevices = () =>
  fetch(server.theOnlyOne + '/gps:' +  id )
    .then((response) => {
      if (!response.ok) {
        throw new Error('Network response was not ok')
      }
      // console.log(response.json())
      return response.json()
    })
    .then((json) => {
      // const names = Object.keys(json);
      // names.forEach(name => {
      //   json[name].timestamp = new Date(json[name].timestamp).getTime() / 1000;
      // })

      const gps = Object.keys(json.data);
      gps.forEach((element) => console.log(element));

      devices.value = json

   
      
    })
    .catch((any) => { })

onMounted(async () => {
  await fetchDevices()

  setInterval(fetchDevices, 3 * 1000)
})

</script>


<style lang="scss">
/* Pls install PostCSS Language Support extension */

/* 
 * Color converter here
 * https://isotropic.co/tool/hex-color-to-css-filter/
 */
.filter-icon {
  filter: invert(73%) sepia(0%) saturate(0%) hue-rotate(187deg) brightness(90%) contrast(86%);
}

.top-menu {
  @apply fixed top-0 z-50;
  @apply flex flex-row items-center;
  @apply w-full h-10;
  @apply gap-1 md:gap-2 lg:gap-4;
  @apply pl-3;
  @apply border-b;
  @apply bg-neutral-100;
}

.text-breakpoints {
  @media (max-width: 500px) {
    @apply text-[12px];
  }
  @media (min-width: 500px) and (max-width: 600px) {
    @apply text-sm;
  }
}
</style>
