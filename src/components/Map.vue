<script setup>
import * as L from "leaflet";
import "proj4leaflet";
import { ref, onMounted, onUnmounted, watch } from "vue";
import "leaflet/dist/leaflet.css";

const crs = new L.Proj.CRS(
  "EPSG:3301",
  "+proj=lcc +lat_1=59.33333333333334 +lat_2=58 +lat_0=57.51755393055556 +lon_0=24 +x_0=500000 +y_0=6375000 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs",
  {
    resolutions: [2048, 1024, 512, 256, 128, 64, 32, 16, 8, 4, 2, 1, 0.5],
  }
);

const proxy = "https://api.allorigins.win/raw?url=";

const files = [
  //"https://strateegia.tallinn.ee/static/81ca1c90e2e6d0c9a1b39c80c05712db/S06-tanavatuubid-local.geojson",
  "https://strateegia.tallinn.ee/static/c3751fb1e2295b2ecc43d9792874194b/S06-tanavatuubid-town_square.geojson",
  "https://strateegia.tallinn.ee/static/a4c3419c61e3e8203ce3bc5e872e4562/S06-tanavatuubid-city_place.geojson",

  "https://strateegia.tallinn.ee/static/a865b991e4fd4736cf3b1f82d8529ded/S06-tanavatuubid-connector.geojson",
  "https://strateegia.tallinn.ee/static/faa9b32f38a8db601ae28466fae7c77c/S06-tanavatuubid-high_street.geojson",
  "https://strateegia.tallinn.ee/static/059a0ab6b99510d2a0eb83c1cc663010/S06-tanavatuubid-city_street.geojson",

  "https://strateegia.tallinn.ee/static/5bd7fc2889030fd799ada3be48d16e6c/S06-tanavatuubid-core_road.geojson",
  "https://strateegia.tallinn.ee/static/17b7b7a33f2d735f9d517ce9b8f3f1e1/S06-tanavatuubid-high_road.geojson",
  "https://strateegia.tallinn.ee/static/aa88f9af8b79eb42ed37b4c0e9ef6c01/S06-tanavatuubid-city_hub.geojson",

  "/kuressaare_roads.json",
].map((url) =>
  fetch(url.startsWith("http") ? proxy + url : url).then((res) => res.json())
);

const mapRef = ref(null);

const colors = {
  //greenLight: "#ADE6B0",
  greenMiddle: "#3AAA64",
  greenDark: "#0D7430",
  purpleLight: "#C9AFEF",
  purpleMiddle: "#B083F6",
  purpleDark: "#6B34C1",
  redLight: "#F2CF4D",
  redMiddle: "#EF8247",
  redDark: "#D2303D",
  gray: "#777",
};

const views = [
  { title: "Tallinn", data: [[59.42350215252274, 24.76599521934987], 13] },
  {
    title: "Kuressaare",
    data: [[58.252932513634654, 22.485026672948155], 15],
  },
];

const activeView = ref(0);

const map = ref(null);

const onClick = (i) => {
  activeView.value = i;
  map.value.setView(...views[activeView.value].data);
};

onMounted(() => {
  // Estonia
  //const view = [[58.77, 25.04], 8]

  map.value = L.map(mapRef.value, { zoom: 19 }).setView(
    ...views[activeView.value].data
  );

  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    maxZoom: 14,
  }).addTo(map.value);

  map.value.on("moveend", (e) =>
    console.log(map.value.getCenter(), map.value.getZoom())
  );

  const wms2 = L.tileLayer
    .wms("https://kaart.maaamet.ee/wms/alus?", {
      layers: "of10000",
      crs,
      minZoom: 14,
      maxZoom: 25,
    })
    .addTo(map.value);

  const wms = L.tileLayer
    .wms(
      "https://gis.saaremaavald.ee:443/arcgis/services/hr/Saaremaa_WMS/MapServer/WMSServer?",
      {
        layers: "16,17,17,19,20,21",
        crs,
        maxZoom: 25,
        opacity: 0.5,
      }
    )
    .addTo(map.value);

  const wms3 = L.tileLayer
    .wms("http://gis.tallinn.ee/aluskaardid?", {
      layers: "detailne_aluskaart",
      crs,
      maxZoom: 25,
      opacity: 0.5,
    })
    .addTo(map.value);

  Promise.all(files).then((files) =>
    files.forEach((file, i) => {
      L.geoJSON(file.features, {
        color: Object.values(colors)[i],
        weight: 2.5,
        opacity: 0.8,
      }).addTo(map.value);
    })
  );
});

// https://gis.saaremaavald.ee/arcgis/services/Rasteralused/VANAD_KAARDID_WMS/MapServer/WMSServer
// 0,1,2,3
</script>

<template>
  <div
    id="map"
    ref="mapRef"
    style="height: 100vh; mix-blend-mode: multiply"
  ></div>
  <div
    style="
      position: fixed;
      top: 10px;
      right: 10px;
      width: 200px;
      background: rgba(255, 255, 255, 0.7);
      z-index: 1000;
      border: 2px solid rgba(0, 0, 0, 0.2);
      background-clip: padding-box;
      border-radius: 4px;
      padding: 12px;
    "
  >
    <div
      v-for="(view, i) in views"
      :key="i"
      @click="onClick(i)"
      :style="{
        cursor: 'pointer',
        color: i === activeView ? 'black' : '#aaa',
        padding: '4px',
        fontWeight: 'bold',
      }"
    >
      {{ view.title }}
    </div>
  </div>
</template>
