<template>
  <div id="app">
    <div class="row no-gutters">
      <div class="col-sm-3">
        <div class="toolbox">
          <div class="sticky-top bg-white shadow-sm p-2">
            <div class="form-group d-flex">
              <label for="cityName" class="mr-2 col-form-label text-right"
                >縣市</label
              >
              <div class="flex-fill">
                <select id="cityName" class="form-control" v-model="select.city" @change="removeMarker(),updateMap(),select.area=''"> 
                  <option value="">請選擇</option> 
                  <option :value="city.CityName" v-for="city in CityName" :key="city.CityName">
                    {{ city.CityName }}  
                  </option> 
                 </select>
              </div>
            </div>
            <div class="form-group d-flex">
              <label for="area" class="mr-2 col-form-label text-right"
                >地區</label
              >
              <div class="flex-fill">
                <!-- <select id="area" class="form-control" v-if="select.city.length" v-model="select.area" @change="updateSelect">
                  <option value="">請選擇</option> 
                  <option :value="aa.areaName" v-for="aa in CityName.find((city) => city.CityName === select.city).AreaList" :key="aa.areaName">
                    {{ aa.areaName }}
                  </option>
                </select> -->
                <select id="area" class="form-control" v-if="select.city.length" v-model="select.area" @change="updateSelect()">
                <option value="">請選擇</option> 
                <option :value="area.AreaName" v-for="area in CityName.find((city) => city.CityName === select.city).AreaList"
                  :key="area.AreaName">
                  {{ area.AreaName }}
                </option>
              </select>
              </div>
            </div>
            <p class="mb-0 small text-muted text-right">
              <!-- 請先選擇區域查看（綠色表示還有口罩) -->
            </p>
          </div>
          <ul class="list-group">
            <template v-for="(item,key) in data">
              <a class="list-group-item text-left" :key="key" v-if="item.properties.county === select.city && item.properties.town === select.area" :class="{'highlight':item.properties.mask_adult || item.properties.mask_child}" @click="penTo(item)">
                <h3>{{item.properties.name}}</h3>
                <p class="mb-0">
                成人口罩：{{item.properties.mask_adult}} | 兒童口罩：{{item.properties.mask_child}}
                </p>
                <p class="mb-0">
                  地址：<a
                    :href="`https://www.google.com.tw/maps/place/${item.properties.address}`"
                    target="_blank"
                    title="Google Map"
                  >
                    {{item.properties.address}}</a
                  >
                </p>
              </a>
            </template>
          </ul>
        </div>
      </div>
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import "bootstrap/dist/js/bootstrap";
import L from "leaflet";
import CityName from './assets/cityName.json'

let osmMap = {};


const osm = {
  penTo(x, y, item) {
    osmMap.panTo(new L.LatLng(y, x));
    L.marker([y, x]).addTo(osmMap).bindPopup(`<strong class="pharmacyName">藥局名稱：${item.name}</strong><br>
        <div class="pharmacyDes">電話：${item.phone}<br>位置：${item.address}<br>
        成人口罩：<strong>${item.mask_adult}</strong><br>兒童口罩：<strong>${item.mask_child}</strong><br>最後更新時間：${item.updated}</div>`).openPopup();
  }
}

export default {
  name: "App",
  data: () => ({
    data: [],
    CityName,
    select:{
      city:'臺南市',
      area:'六甲區'
    }
  }),
  components: {
    // HelloWorld
  },
  methods:{
    updateMap(){
      const pharmacies = this.data.filter((pharmacy)=>(pharmacy.properties.county === this.select.city));
      // console.log(pharmacy)
      pharmacies.forEach((pharmacy) => {
        const { properties, geometry } = pharmacy;

        L.marker([geometry.coordinates[1],geometry.coordinates[0]],properties).addTo(osmMap).bindPopup(`<strong class="pharmacyName">藥局名稱：${pharmacy.properties.name}</strong><br>
        <div class="pharmacyDes">電話：${pharmacy.properties.phone}<br>位置：${pharmacy.properties.address}<br>
        成人口罩：<strong>${pharmacy.properties.mask_adult}</strong><br>兒童口罩：<strong>${pharmacy.properties.mask_child}</strong><br>最後更新時間：${pharmacy.properties.updated}</div>`)});
      this.penTo(pharmacies[0]);
      console.log(pharmacies[0])
    },
    removeMarker(){
      osmMap.eachLayer((layer) => {
        if(layer instanceof L.Marker){
          osmMap.removeLayer(layer);
        } 
      })
    },
    penTo(pharmacy) {
      const { properties, geometry } = pharmacy;
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties);
    },
    // updateSelect(){
    //   this.removeMarker();
    //   this.updateMap();
    //   const pharmacy = this.data.find(item => item.properties.towm === this.select.area);
    //   const { properties, geometry } = pharmacy;
    //   osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties);
    // }
    updateSelect() {
      this.removeMarker();
      this.updateMap();
      const pharmacy = this.data.find(item => item.properties.town === this.select.area);
      const { geometry, properties } = pharmacy;
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties);
    },
  },
  mounted() {
    const url =
      "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";

    this.$http.get(url).then(response => {
      console.log(response.data);
      this.data = response.data.features;
      this.updateMap()
    });
    osmMap = L.map("map", {
      center: [25.03, 121.55],
      zoom: 13
    });
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png?{foo}", {
      foo: "bar",
      attribution:
        'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>'
    }).addTo(osmMap);

    L.marker([25.03, 121.55]).addTo(osmMap);
  }
};
</script>

<style lang="scss">
@import "bootstrap/scss/bootstrap";

#map {
  height: 100vh;
}
.highlight {
  background: #e9ffe3;
}
.toolbox {
  height: 100vh;
  overflow-y: auto;
  a {
    cursor: pointer;
  }
}
.pharmacyName{
  font-size: 16px;
}

.pharmacyDes{
  font-size: 14px;
}
</style>
