<template>
  <div class="top-menu">
    <div
      class="flex items-center justify-center w-10 h-full mt-auto cursor-pointer"
      @click="() => hideFilterPanel = !hideFilterPanel"
    >
      <img
        src="/src/assets/icons/bars-solid.svg"
        alt="Toogle filter"
        class="w-5 h-5 xl:w-6 xl:h-6 filter-icon"
      />
    </div>
    <div class="font-bold transition-colors w-fit text-breakpoints">
      Meshtastic MQTT Map
    </div>
    <div class="flex flex-wrap ml-auto mr-3 text-blue-500 text-breakpoints">
      <div @click="devicesALL == !devicesALL">
      <div v-if="devicesALL">Показаны ноды за все время</div>
      <div v-else>Показаны ноды за 24 часа</div>
      </div>
    </div>
  </div>
  <div v-if="!devices" class="absolute mt-2 italic left-4">Fetching devices...</div>
  <div class="flex flex-col w-full h-full mt-10 xl:flex-row"  >
    <div v-if="devices" class="overflow-y-scroll border-r xl:w-1/3 h-1/2 xl:h-full" :class="{ hidden: hideFilterPanel }">
      <div class="flex flex-col">
        <div class="px-3 mt-3 mb-1">
          <input v-model="filter" class="w-full border p-1.5 hover:outline-none focus:outline-none" placeholder="Filter by shortName/longName/server/id" />
        </div>
        <div class="p-3 transition-colors hover:bg-neutral-100" v-for="device in filtered" :key="device">
          <div class="cursor-pointer select-none" >
            <div class="text-xl flex gap-1.5 cursor-pointer" @click="devices[device].opened = !devices[device].opened">
              <span :class="(Math.round(Date.now() / 1000) - devices[device].timestamp) > 3600 ? 'text-neutral-500' : 'text-blue-600'">
                <span v-if="devices[device]?.user?.data?.longName">{{ devices[device]?.user?.data?.longName }}</span>
                <span v-else>{{ device }}</span>
              </span>
          </div>
            <div v-if="((Math.round(Date.now() / 1000) - devices[device].timestamp) > 3600)" @click="addToFilter(devices[device].user?.data?.longName)"> Last heard: {{new Date(devices[device].timestamp * 1000).toLocaleString()}} </div>
            <div v-else>{{ timeAgo(new Date(devices[device].timestamp * 1000).getTime()) }}</div>
          </div>
          <div v-if="devices[device].opened">
            <table>
              <tbody>
                <tr>
                  <td>MQTT Server</td>
                  <td><div @click="addToFilter(devices[device].server)" class="p-1 text-sm text-blue-500 cursor-pointer w-fit">{{ devices[device].server }}</div></td>
              </tr>
                  <tr v-if="devices[device]?.user?.data?.shortName">
                  <td>Short name</td>
                  <td>{{ devices[device].user.data.shortName }}</td>
                </tr>
                <tr v-if="devices[device]?.user?.data?.hwModel">
                  <td>Hardware</td>
                  <td>{{ devices[device].user.data.hwModel }}</td>
                </tr>
                <tr v-if="devices[device]?.user?.data?.id">
                  <td>ID</td>
                  <td>{{ devices[device].user.data.id }} 
                    ({{ (devices[device]?.position?.from) ? devices[device]?.position?.from : devices[device]?.telemetry?.from }})</td>
                </tr>
                <tr v-if="devices[device]?.position?.data?.latitudeI">
                  <td>Position </td>
                  <td>{{ Number(devices[device].position.data.latitudeI / 10000000).toFixed(4) }}, {{ Number(devices[device].position.data.longitudeI / 10000000).toFixed(4) }}</td>
                </tr>
                <tr v-if="devices[device]?.position?.data?.altitude">
                  <td>Altitude</td>
                  <td>{{ devices[device].position.data.altitude }} m</td>
                </tr>
                <tr v-if="devices[device]?.position?.data?.satsInView">
                  <td>Sat's In View</td>
                  <td>{{ devices[device].position.data.satsInView }} sat's</td>
                </tr>
                <tr v-if="devices[device]?.user?.hopLimit">
                  <td>User Hop Limit</td>
                  <td> {{ devices[device].user.hopLimit }}</td>
                </tr>
                <tr v-if="devices[device]?.position?.hopLimit">
                  <td>Position Hop Limit</td>
                  <td>{{ devices[device].position.hopLimit }}</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.hopLimit">
                  <td>Telemetry Hop Limit</td>
                  <td>{{ devices[device].telemetry.hopLimit }}</td>
                </tr>
                <tr v-if="devices[device]?.user?.rxSnr">
                  <td>RX SNR</td>
                  <td>{{ devices[device].user.rxSnr }} <br> уровень сигнала с которым пришел пакет nodeinfo</td>
                </tr>
                <tr v-if="devices[device]?.user?.rxRssi">
                  <td>RX RSSI</td>
                  <td>{{ devices[device].user.rxRssi }} <br> уровень сигнала с которым пришел пакет nodeinfo</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.data?.deviceMetrics?.airUtilTx">
                  <td>Air util tx</td>
                  <td>{{ Number(devices[device].telemetry.data.deviceMetrics.airUtilTx).toFixed(1) }} %</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.data?.deviceMetrics?.channelUtilization">
                  <td>Channel utilization</td>
                  <td>{{ Number(devices[device].telemetry.data.deviceMetrics.channelUtilization).toFixed(1) }} %</td>
                </tr>
                <tr v-if="devices[device]?.user?.channel >= 0">
                  <td>Lora channel</td>
                  <td>{{ devices[device].user.channel }}</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.data?.deviceMetrics?.voltage">
                  <td>Battery voltage</td>
                  <td>{{ Number(devices[device]?.telemetry?.data?.deviceMetrics?.voltage).toFixed(1) }} V</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.data?.deviceMetrics?.batteryLevel">
                  <td>Battery level</td>
                  <td>{{ (devices[device].telemetry.data.deviceMetrics.batteryLevel > 100) ? 100 : (Math.round(devices[device].telemetry.data.deviceMetrics.batteryLevel))  }} %</td>
                </tr>
                <tr v-if="devices[device]?.telemetry2?.data?.deviceMetrics?.barometricPressure">
                  <td>Barometric pressure</td>
                  <td>{{ Math.round(devices[device].telemetry.data.environmentMetrics.barometricPressure) }} hPa</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.data?.environmentMetrics?.current">
                  <td>Current</td>
                  <td>{{ devices[device].telemetry.data.environmentMetrics.current }}</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.data?.environmentMetrics?.voltage">
                  <td>Voltage</td>
                  <td>{{ devices[device].telemetry.data.environmentMetrics.voltage }}</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.data?.environmentMetrics?.gasResistance">
                  <td>Gas</td>
                  <td>{{ Number(devices[device].telemetry.data.environmentMetrics.gasResistance).toFixed(0) }} MOhms</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.data?.environmentMetrics?.relativeHumidity">
                  <td>Humidity</td>
                  <td>{{ Number(devices[device].telemetry.data.environmentMetrics.relativeHumidity).toFixed(0) }} %</td>
                </tr>
                <tr v-if="devices[device]?.telemetry?.data?.environmentMetrics?.temperature">
                  <td>Temperature</td>
                  <td>{{ Number(devices[device].telemetry.data.environmentMetrics.temperature).toFixed(1) }} ℃</td>
                </tr>
                <!-- <tr v-if="devices[device]?.mqtt">
                  <td><div class="font-bold">MQTT: </div></td> <td><div class="font-bold">Online</div></td>
                </tr> -->
                <tr v-if="devices[device]?.mqtt">
                  <td>Server: </td> <td>{{devices[device]?.server}}</td>
                </tr>
                <tr v-if="devices[device]?.text?.data || devices[device]?.message?.data">
                  <td>Last public message: </td> <td>{{devices[device]?.message?.data }} </td>
                </tr>
                </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
    <div id="map" class="w-full h-1/2 xl:h-full"></div>
  </div>
</template>

<script setup>
import { ref, unref, shallowRef, computed, onMounted } from 'vue'
import { useServer } from './servers.js'

/*
  import { useBreakpoints } from '@vueuse/core' — реактивный оверинжиниринг, котоырй ещё и npm-мить
  Breakpoints:
    xs: 0,
    sm: 576,
    md: 768,
    lg: 992,
    xl: 1200,
    xxl: 1400
*/
let hideFilterPanel = ref(true) // ref(window.innerWidth < 992)

const devices = ref()
const devicesALL = ref()
devicesALL == false
const devicesPT = ref()
const senderList = ref()

const timeAgo = (date) => {
  const seconds = Math.floor((new Date() - date) / 1000)
  const interval = Math.floor(seconds / 60)
  if (interval >= 1) {
    return interval + ' min ago'
  }
  if (seconds < 10) return 'just now'
  return Math.floor(seconds) + ' sec ago'
}

const server = useServer()

const fetchDevices = () =>
  fetch(server.theOnlyOne)
    .then((response) => {
      if (!response.ok) {
        throw new Error('Network response was not ok')
      }
      return response.json()
    })
    .then((json) => {
      const names = Object.keys(json);
      names.forEach(name => {
        json[name].timestamp = new Date(json[name].timestamp).getTime() / 1000;
      })
      devices.value = json
    })
    .catch((any) => { })

onMounted(async () => {
  await fetchDevices()
  // setInterval(fetchDevices, 30 * 1000)

  let geolocationmsk = [55.76, 37.64] // moskowa

  const init = () => {
    var map = new ymaps.Map('map', {
      center: geolocationmsk,
      zoom: 9,
      controls: ['smallMapDefaultSet']
    }),
    firstButton = new ymaps.control.Button("СПИСОК")

    map.controls.add(firstButton, { float: 'left' })

    firstButton.events.add('click', function () {
      alert('О, кнопка!')
    })

    var geolocation = ymaps.geolocation

    // Сравним положение, вычисленное по ip пользователя и положение, вычисленное средствами браузера.
    // geolocation.get({
    //     provider: 'yandex',
    //     mapStateAutoApply: true
    // }).then(function (result) {
    //     result.geoObjects.options.set('preset', 'islands#redCircleIcon');
    //     result.geoObjects.get(0).properties.set({
    //         balloonContentBody: 'Мое местоположение по IP'
    //     })
    //     map.geoObjects.add(result.geoObjects);
    // })

    geolocation.
    get({
        provider: 'browser',
        zoom: 8,
        mapStateAutoApply: false 
      })
    .then(function (result) {
      result.geoObjects.options.set('preset', 'islands#redCircleIcon')
      result.geoObjects.get(0).properties.set({ 
        balloonContentBody: 'Скорее всего, я тут'
        })
      map.geoObjects.add(result.geoObjects)
    })

    for (const index in devices.value) {
      const device = devices.value[index]

      const [latitude, longitude] = [device?.position?.data?.latitudeI / 10000000, device?.position?.data?.longitudeI / 10000000]
      const name = device?.user?.data?.shortName || device?.user?.data?.longName || device?.user?.data?.id

      if (latitude && longitude && (Math.round(Date.now() / 1000) - device.timestamp < 86400)) { // проверяем что ноды (пейджеры) были в сети за сутки
        let presetcolor = ((device?.user?.rxSnr === 0)&& (device?.user?.rxRssi === 0))  ? 'islands#darkBlueStretchyIcon' : 'islands#blueStretchyIcon'
        presetcolor = (Math.round(Date.now() / 1000) - device.timestamp > 3600) ? 'islands#greyStretchyIcon' : presetcolor
        const timestampfooter = (Math.round(Date.now() / 1000) - device.timestamp > 3600) ? (new Date(device.timestamp * 1000).toLocaleString()) : (timeAgo(new Date(device.timestamp * 1000).getTime()))

        let balloonContents = ''
        if (device?.position?.data?.altitude) { balloonContents += `<div>Altitude: ${Number(device?.position?.data?.altitude).toFixed(0)} m</div>` }
        if (device?.position?.data?.satsInView) { balloonContents += `<div>Sat's in view: ${device?.position?.data?.satsInView} Sat's</div>`}
      
        if (device?.telemetryEnvironment?.data?.environmentMetrics?.temperature) { balloonContents += `<div>Temperature: ${Number(device?.telemetryEnvironment?.data?.environmentMetrics?.temperature).toFixed(1)} C</div>` }
        if (device?.telemetryEnvironment?.data?.environmentMetrics?.relativeHumidity) { balloonContents += `<div>Humidity: ${Number(device?.telemetryEnvironment?.data?.environmentMetrics?.relativeHumidity).toFixed(0)} %</div>` }
        if (device?.telemetryEnvironment?.data?.environmentMetrics?.barometricPressure) { balloonContents += `<div>Pressure: ${Math.round(device?.telemetryEnvironment?.data?.environmentMetrics?.barometricPressure)} hPa</div>` }
        if (device?.telemetryEnvironment?.data?.environmentMetrics?.gasResistance) { balloonContents += `<div>Gas Resistance (AQI): ${Number(device?.telemetryEnvironment?.data?.environmentMetrics?.gasResistance).toFixed(0)} MOhms</div>` }
        if (device?.telemetryEnvironment?.data?.environmentMetrics?.voltage) { balloonContents += `<div>Battery voltage: ${Number(device?.telemetryEnvironment?.data?.environmentMetrics?.voltage).toFixed(1)} V</div>` }
        if (device?.telemetryEnvironment?.data?.environmentMetrics?.current) { balloonContents += `<div>Current: ${Number(device?.telemetryEnvironment?.data?.environmentMetrics?.current).toFixed(1)} %</div>` }

        if (device?.telemetry?.data?.deviceMetrics?.batteryLevel) { balloonContents += `<div>Battery level: ${Number(device?.telemetry?.data?.deviceMetrics?.batteryLevel).toFixed(0) > 100 ? 100 : Number(device?.telemetry?.data?.deviceMetrics?.batteryLevel).toFixed(0)} %</div>` }
        if (device?.telemetry?.data?.deviceMetrics?.voltage) { balloonContents += `<div>Battery voltage: ${Number(device?.telemetry?.data?.deviceMetrics?.voltage).toFixed(2)} V</div>` }
        if (device?.telemetry?.data?.deviceMetrics?.channelUtilization) { balloonContents += `<div>Channel utilization: ${Number(device?.telemetry?.data?.deviceMetrics?.channelUtilization).toFixed(1)} %</div>` }
        if (device?.telemetry?.data?.deviceMetrics?.airUtilTx) { balloonContents += `<div>Air util TX: ${Number(device?.telemetry?.data?.deviceMetrics?.airUtilTx).toFixed(1)} %</div>` }

        if (device?.user?.rxRssi!== undefined) { balloonContents += `RX RSSI: ${Math.round(device?.user?.rxRssi).toFixed(0)},  ` }
        if (device?.user?.rxSnr!== undefined) { balloonContents += `RX SNR: ${Math.round(device?.user?.rxSnr).toFixed(0)}` }
        if ((device?.user?.hopLimit)||(device?.position?.hopLimit)||(device?.telemetry?.hopLimit)){
          balloonContents += `<div>Hop's:` 
          if (device?.user?.hopLimit) { balloonContents += ` User: ${Number(device?.user?.hopLimit)}` }
          if (device?.position?.hopLimit) { balloonContents += ` Position: ${Number(device?.position?.hopLimit)}` }
          if (device?.telemetry?.hopLimit) { balloonContents += ` Telemetry: ${Number(device?.telemetry?.hopLimit)} ` }
          balloonContents += `</div>` 
        }
        if ((device?.user?.rxSnr === 0)&& (device?.user?.rxRssi === 0)) {
          balloonContents += `<div class="font-bold">MQTT: YES </div>`
          balloonContents += `<div>Server: ${device?.server}</div>`
        }
        if (device?.message?.data !== undefined) { balloonContents += `<div>Last public message: ${device.message.data} </div>` }

        map.geoObjects
          .add(new window.ymaps.Placemark([latitude, longitude], {
            iconContent: name,
            balloonContentHeader: `
              <div>Short Name: ${device?.user?.data?.shortName}</div>
              <div>Long Name: ${device?.user?.data?.longName}</div>`,
            balloonContentBody: ` 
              <div>Node ID: ${device?.user?.data?.id} (${(device?.position?.from) ? device?.position?.from : device?.telemetry?.from})</div>
              <div>Hardware: ${device?.user?.data?.hwModel}</div>
              <div>Role: ${device?.user?.data?.role}</div>

              <div>Position: <a href="yandexmaps://maps.yandex.ru/?ll=${latitude},${longitude}&z=12"> ${latitude}, ${longitude}</a></div>

              <div> ${balloonContents}</div>`,
            balloonContentFooter: `Updated: ${timestampfooter}`
          }, { preset: `${presetcolor}` }))       
      }
    } 
  }
  
  ymaps.ready(init)  
})

const filter = shallowRef('')

// this is actually a setFilter
const addToFilter = (item) => {
  console.debug('addToFilter item', item)
  filter.value = item
}

const filtered = computed(() => {
  if (!filter.value) {
    return Object.keys(devices.value)
  }
  else {
    const candidates = {}
    // devices is an object почему-то...
    const needle = filter.value.toLowerCase()
    for (const candidate in devices.value) {
      // console.log(
      //   devices.value[candidate].server,
      //   devices.value[candidate]?.user?.data?.shortName,
      //   devices.value[candidate]?.user?.data?.longName,
      //   devices.value[candidate]?.user?.data?.id
      // )
      // сюда
      if (devices.value[candidate].server.match(needle)) {
        candidates[candidate] = devices.value[candidate]
      }
      else if (devices.value[candidate]?.user?.data?.shortName.toLowerCase().match(needle)) {
        candidates[candidate] = devices.value[candidate]
      }
      else if (devices.value[candidate]?.user?.data?.longName.toLowerCase().match(needle)) {
        candidates[candidate] = devices.value[candidate]
      }
      else if (devices.value[candidate]?.user?.data?.id.toLowerCase().match(needle)) {
        candidates[candidate] = devices.value[candidate]
      }
    }
    return Object.keys(candidates)
  }
})

const servers = computed(() => {
  if (devices.value === undefined) {
    return []
  }
  const candidates = new Set()
  // все еше Object
  for (const candidate in devices.value) {
    candidates.add(devices.value[candidate].server)
  }
  // console.log('C', candidates)
  return Array.from(candidates)
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
