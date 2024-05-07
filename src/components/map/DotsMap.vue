<template>
  <div id="map" class="w-full h-full"></div>
</template>

<script setup>
import {
  ref,
  unref,
  shallowRef,
  computed,
  onMounted,
  toRefs,
  watch,
} from "vue";

import { debounce } from "lodash-es";

const emit = defineEmits(["infoOpen", "tableOpen"]);

const props = defineProps({
  devices: {
    required: true,
    type: Object,
  },
});

const { devices } = toRefs(props);

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

const timeAgo = (date) => {
  const seconds = Math.floor((new Date() - date) / 1000);
  const interval = Math.floor(seconds / 60);
  if (interval >= 1) {
    return interval + " min ago";
  }
  if (seconds < 10) return "just now";
  return Math.floor(seconds) + " sec ago";
};

// const server = useServer()

onMounted(async () => {
  let geolocationmsk = [55.76, 37.64]; // moskovv center off world
  let map;

  const renderSelfBallon = () => {
    let geolocation = ymaps.geolocation;

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

    geolocation
      .get({
        provider: "browser",
        zoom: 8,
        mapStateAutoApply: false,
      })
      .then(function (result) {
        result.geoObjects.options.set("preset", "islands#redCircleIcon");
        result.geoObjects
          .get(0)
          .properties.set({ balloonContentBody: "Скорее всего, я тут" });
        map.geoObjects.add(result.geoObjects);
      });
  };

  const renderBallons = (devices) => {
    renderSelfBallon();

    for (const index in devices) {
      const device = devices[index];

      const [latitude, longitude] = [
        device?.position?.data?.latitudeI / 10000000,
        device?.position?.data?.longitudeI / 10000000,
      ];
      const name =
        device?.user?.data?.shortName ||
        device?.user?.data?.longName ||
        device?.user?.data?.id;

      const [[leftBottomLat, leftBottomLong], [rightTopLat, rightTopLong]] =
        map.getBounds();

      if (!latitude || !longitude) {
        continue;
      }

      if (latitude < leftBottomLat || latitude > rightTopLat) {
        continue;
      }

      if (longitude < leftBottomLong || longitude > rightTopLong) {
        continue;
      }

      let presetcolor =
        device?.user?.rxSnr === 0 && device?.user?.rxRssi === 0
          ? "islands#darkBlueStretchyIcon"
          : "islands#blueStretchyIcon";
      presetcolor =
        Math.round(Date.now() / 1000) - device.timestamp > 3600
          ? "islands#greyStretchyIcon"
          : presetcolor;
      const timestampfooter =
        Math.round(Date.now() / 1000) - device.timestamp > 3600
          ? new Date(device.timestamp * 1000).toLocaleString()
          : timeAgo(new Date(device.timestamp * 1000).getTime());

      let balloonContents = "";
      if (device?.position?.data?.altitude) {
        balloonContents += `<div>Altitude: ${Number(
          device?.position?.data?.altitude
        ).toFixed(0)} m</div>`;
      }
      if (device?.position?.data?.satsInView) {
        balloonContents += `<div>Sat's in view: ${device?.position?.data?.satsInView} Sat's</div>`;
      }

      const { environmentMetrics } = device?.environmentMetrics?.data || {};
      if (environmentMetrics?.temperature) {
        balloonContents += `<div>Temperature: ${Number(
          device?.environmentMetrics?.data?.environmentMetrics?.temperature
        ).toFixed(1)} C</div>`;
      }
      if (environmentMetrics?.relativeHumidity) {
        balloonContents += `<div>Humidity: ${Number(
          device?.environmentMetrics?.data?.environmentMetrics?.relativeHumidity
        ).toFixed(0)} %</div>`;
      }
      if (environmentMetrics?.barometricPressure) {
        balloonContents += `<div>Pressure: ${Math.round(
          device?.environmentMetrics?.data?.environmentMetrics
            ?.barometricPressure
        )} hPa</div>`;
      }
      if (environmentMetrics?.gasResistance) {
        balloonContents += `<div>Gas Resistance (AQI): ${Number(
          device?.environmentMetrics?.data?.environmentMetrics?.gasResistance
        ).toFixed(0)} MOhms</div>`;
      }

      const { deviceMetrics } = device?.deviceMetrics?.data || {};
      if (deviceMetrics?.batteryLevel) {
        balloonContents += `<div>Battery: ${
          Number(
            device?.deviceMetrics?.data?.deviceMetrics?.batteryLevel
          ).toFixed(0) > 100
            ? 100
            : Number(
                device?.deviceMetrics?.data?.deviceMetrics?.batteryLevel
              ).toFixed(0)
        } % (${Number(
          device?.deviceMetrics?.data?.deviceMetrics?.voltage
        ).toFixed(2)} V)</div>`;
      }
      if (deviceMetrics?.airUtilTx) {
        balloonContents += `<div>Air util TX: ${Number(
          device?.deviceMetrics?.data?.deviceMetrics?.airUtilTx
        ).toFixed(1)} %</div>`;
      }

      if (
        device?.user?.rxRssi !== undefined &&
        device?.user?.rxSnr !== undefined &&
        device?.user?.rxRssi !== 0
      ) {
        balloonContents += `<div>NodeInfo RX RSSI: ${Math.round(
          device?.user?.rxRssi
        ).toFixed(0)},  SNR: ${Math.round(device?.user?.rxSnr).toFixed(
          0
        )}</div>`;
      }
      if (
        device?.position?.rxRssi !== undefined &&
        device?.position?.rxSnr !== undefined &&
        device?.position?.rxRssi !== 0
      ) {
        balloonContents += `<div>Position RX RSSI: ${Math.round(
          device?.position?.rxRssi
        ).toFixed(0)},  SNR: ${Math.round(device?.position?.rxSnr).toFixed(
          0
        )}</div>`;
      }
      if (
        device?.deviceMetrics?.rxRssi !== undefined &&
        device?.deviceMetrics?.rxSnr !== undefined &&
        device?.deviceMetrics?.rxRssi !== 0
      ) {
        balloonContents += `<div>Device Metrics RX RSSI: ${Math.round(
          device?.deviceMetrics?.rxRssi
        ).toFixed(0)},  SNR: ${Math.round(device?.deviceMetrics?.rxSnr).toFixed(
          0
        )}</div>`;
      }

      if (
        (device?.user?.hopLimit ||
          device?.position?.hopLimit ||
          device?.deviceMetrics?.hopLimit) &&
        device?.user?.rxRssi !== 0
      ) {
        balloonContents += `<div>Hop's:`;
        if (device?.user?.hopLimit) {
          balloonContents += ` User: ${Number(device?.user?.hopLimit)}`;
        }
        if (device?.position?.hopLimit) {
          balloonContents += ` Position: ${Number(device?.position?.hopLimit)}`;
        }
        if (device?.deviceMetrics?.hopLimit) {
          balloonContents += ` Telemetry: ${Number(
            device?.deviceMetrics?.hopLimit
          )} `;
        }
        balloonContents += `</div>`;
      }
      if (device?.user?.rxSnr === 0 && device?.user?.rxRssi === 0) {
        balloonContents += `<div class="font-bold">MQTT: YES </div>`;
        balloonContents += `<div>Server: ${device?.server}</div>`;
      }
      if (device?.message?.data !== undefined) {
        balloonContents += `<div>Last public message: ${device.message.data} </div>`;
      }

      map.geoObjects.add(
        new window.ymaps.Placemark(
          [latitude, longitude],
          {
            iconContent: name,
            balloonContentHeader: `
              <div>Short Name: ${device?.user?.data?.shortName}</div>
              <div>Long Name: ${device?.user?.data?.longName}</div>`,
            balloonContentBody: ` 
              <div>Node ID: ${device?.user?.data?.id} (${
              device?.position?.from
                ? device?.position?.from
                : device?.deviceMetrics?.from
            })</div>
              <div>Hardware: ${device?.user?.data?.hwModel}</div>
              <div>Role: ${device?.user?.data?.role}</div>

              <div>Position: <a href="yandexmaps://maps.yandex.ru/?ll=${latitude},${longitude}&z=12"> ${latitude}, ${longitude}</a></div>

              <div> ${balloonContents}</div>`,
            balloonContentFooter: `Updated: ${timestampfooter}`,
          },
          { preset: `${presetcolor}` }
        )
      );
    }
  };

  const initYMap = () => {
    map = new ymaps.Map("map", {
      center: geolocationmsk,
      zoom: 9,
    });
    map.controls.remove("fullscreenControl"); // костыль против двух кнопок
    map.controls.remove("searchControl"); // удаляем кнопку перехода в полноэкранный режим

    // var fullscreenControl = new ymaps.control.FullscreenControl();
    // map.controls.add(fullscreenControl)
    // fullscreenControl.enterFullscreen() // переходив в enterFullscreen

    let infoButton = new ymaps.control.Button("INFO");
    map.controls.add(infoButton, {
      selectOnClick: false,
      float: "left",
      floatIndex: 0,
    });
    infoButton.events.add("click", function () {
      emit("infoOpen");
    });

    let tabletButton = new ymaps.control.Button("NODES");
    map.controls.add(tabletButton, {
      selectOnClick: false,
      float: "left",
      floatIndex: 1,
    });
    tabletButton.events.add("click", function () {
      emit("tableOpen");
    });

    const onBoundsChange = (e) => { 
      map.geoObjects.removeAll();
      renderBallons(devices?.value);
    };
    map.events.add("boundschange", debounce(onBoundsChange, 1000));

    map.events.add("contextmenu", function (e) {
      console.log(e.get("coords"));
      alert(e.get("coords"));
    });
  };

  const init = () => {
    // console.log("!!! init", devices.value);

    initYMap();
    // renderBallons();

    renderBallons(devices?.value);

    watch(devices, (newDevices) => { // следит за обновлениями данных
      map.geoObjects?.removeAll();
      renderBallons(newDevices);
    });
  };

  ymaps.ready(init);

  // setInterval(() => {
  //   //fetchDevices()
  //   map.geoObjects.removeAll();
  //   renderBallons();
  // }, 20000);
});

const filter = shallowRef("");

// this is actually a setFilter
const addToFilter = (item) => {
  // console.debug('addToFilter item', item)
  filter.value = item;
};

const filtered = computed(() => {
  if (!filter.value) {
    return Object.keys(devices.value);
  } else {
    const candidates = {};
    const needle = filter.value.toLowerCase();
    for (const candidate in devices.value) {
      // console.log(
      // devices.value[candidate].server,
      // devices.value[candidate]?.user?.data?.shortName,
      // devices.value[candidate]?.user?.data?.longName,
      // devices.value[candidate]?.user?.data?.id
      // )
      if (devices.value[candidate].server.match(needle)) {
        candidates[candidate] = devices.value[candidate];
      } else if (
        devices.value[candidate]?.user?.data?.shortName
          .toLowerCase()
          .match(needle)
      ) {
        candidates[candidate] = devices.value[candidate];
      } else if (
        devices.value[candidate]?.user?.data?.longName
          .toLowerCase()
          .match(needle)
      ) {
        candidates[candidate] = devices.value[candidate];
      } else if (
        devices.value[candidate]?.user?.data?.id.toLowerCase().match(needle)
      ) {
        candidates[candidate] = devices.value[candidate];
      }
    }
    return Object.keys(candidates);
  }
});

const servers = computed(() => {
  if (devices.value === undefined) {
    return [];
  }
  const candidates = new Set();
  for (const candidate in devices.value) {
    candidates.add(devices.value[candidate].server);
  }
  // console.log('C', candidates)
  return Array.from(candidates);
});
</script>

<style lang="scss">
/* Pls install PostCSS Language Support extension */

/* 
 * Color converter here
 * https://isotropic.co/tool/hex-color-to-css-filter/
 */
.filter-icon {
  filter: invert(73%) sepia(0%) saturate(0%) hue-rotate(187deg) brightness(90%)
    contrast(86%);
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