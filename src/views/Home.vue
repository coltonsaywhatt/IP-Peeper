<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search / Results -->
    <div
      class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32"
    >
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <img class="m-auto max-h-12" src="../assets/logoIP.png" alt="">
        <h1 class="text-white text-center text-5xl">IP PEEPER</h1>
        <h4 class="text-white text-center pb-4">LOCATE ANY IP ADDRESS</h4>
        <div class="flex">
          <input
            v-model="queryIp"
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
            type="text"
            placeholder="Search for any IP address or leave empty to get your ip info"
          />
          <i
            @click="getIpInfo"
            class="cursor-pointer
          bg-black
          text-white
            px-4
            rounded-tr-md
            rounded-br-md
            flex
            items-center
            fas fa-chevron-right"
          ></i>
        </div>
      </div>
      <!-- IP Info -->
      <IPInfo v-if="ipInfo" v-bind:ipInfo="ipInfo" />
    </div>

    <!-- Map -->
    <div id="mapid" class="h-full z-10"></div>
  </div>
</template>

<script>
import IPInfo from "../components/IPInfo.vue";
import leaflet from "leaflet";
import { onMounted, ref } from "vue";
import axios from "axios";
export default {
  name: "HomeView",
  components: { IPInfo },
  setup() {
    // create map variable
    let mymap;

    // data
    const queryIp = ref("");
    const ipInfo = ref(null);

    // mounted lifecycle hook, creates the map
    onMounted(() => {
      mymap = leaflet.map("mapid").setView([28.1471914,-81.966857], 9);

      leaflet
        .tileLayer(
          "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoiY29sdG9ubGVlIiwiYSI6ImNsODZxOGFwaTEyNzEzdm50aWp6OGp0dmMifQ.JzIVK0ztyWeN3bu5rQXGVg",
          {
            attribution:
              'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            accessToken:
              "pk.eyJ1IjoiY29sdG9ubGVlIiwiYSI6ImNsODZxOGFwaTEyNzEzdm50aWp6OGp0dmMifQ.JzIVK0ztyWeN3bu5rQXGVg",
          }
        )
        .addTo(mymap);
    });

    // gets ip information from API
    const getIpInfo = async () => {
      try {
        const data = await axios.get(
          `https://geo.ipify.org/api/v2/country,city?apiKey=at_ALFN1wXfR3hps3bVXRvL3ysnF2ULO&ipAddress=${queryIp.value}`
        );
        const result = data.data;
        console.log(result);
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      } catch (err) {
        alert(err.message);
      }
    };

    return { queryIp, ipInfo, getIpInfo };
  },
};
</script>