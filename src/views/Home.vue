<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- search / results -->
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">

      <!-- search input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">Ip Address Tracker</h1>
        <div class="flex">
          <input 
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
            placeholder="Search for any IP address or leave empty to get your ip info"
            type="text"
            v-model="queryIp"/>
            <i 
              @click="getIpInfo"
              class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fas fa-chevron-right"></i>
        </div>
      </div>

      <!-- IpInfoComponent -->
      <IpInfoComponents v-if="ipInfo" :ipInfo="ipInfo" />
    </div>
    <!-- Map -->
    <div id="mapid" class="h-full z-10"></div>
  </div>
</template>

<script>
// @ is an alias to /src
import IpInfoComponents from '../components/IpInfoComponent.vue';
import leaflet from 'leaflet';
import { onMounted, ref } from 'vue';
import axios from 'axios';
export default {
  name: 'Home',
  components: {
    IpInfoComponents,
  },
  setup(){

    let mymap;

    const queryIp = ref("");
    const ipInfo = ref(null);
    onMounted(()=>{
      mymap = leaflet.map('mapid').setView([0, 0], 3);
      leaflet.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoiZW1tYW51ZWxwYW5kb3JhIiwiYSI6ImNrcndja2IwMTBmOTcydW83ZmRnMGlydWQifQ.2Y6XFiT7-nrz2uO6w_GEfg', {
          attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: 'mapbox/streets-v11',
          tileSize: 512,
          zoomOffset: -1,
          accessToken: 'pk.eyJ1IjoiZW1tYW51ZWxwYW5kb3JhIiwiYSI6ImNrcndja2IwMTBmOTcydW83ZmRnMGlydWQifQ.2Y6XFiT7-nrz2uO6w_GEfg'
      }).addTo(mymap);
    });

    const getIpInfo = async ()=>{
      try{
        const data = await axios.get(`
          https://geo.ipify.org/api/v1?apiKey=at_nAb6mU6lFdyQyePR5s0q1FO7tHaB0&ipAddress=${queryIp.value}`
        );
        const result = data.data;
        ipInfo.value = {
          address : result.ip,
          state : result.location.region,
          timezone : result.location.timezone,
          isp :result.isp,
          lat: result.location.lat,
          lng: result.location.lng
        };
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng],13);
      }catch(err){
        alert(err.message);
      }
    };

    return {queryIp, ipInfo , getIpInfo}

  }
}
</script>
