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
                <select id="cityName" class="form-control" v-model="select.city" @change="updateMap()"> 
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
                <select id="area" class="form-control">
                  <option value="">-- Select One --</option>
                </select>
              </div>
            </div>
            <p class="mb-0 small text-muted text-right">
              請先選擇區域查看（綠色表示還有口罩）
            </p>
          </div>
          <ul class="list-group">
            <template>
              <a class="list-group-item text-left">
                <h3>藥局名稱</h3>
                <p class="mb-0">
                  成人口罩：1 | 兒童口罩：2
                </p>
                <p class="mb-0">
                  地址：<a
                    href="https://www.google.com.tw/maps/place/..."
                    target="_blank"
                    title="Google Map"
                  >
                    地址</a
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

export default {
  name: "App",
  data: () => ({
    data: [],
    CityName,
    select:{
      city:'臺北市'
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
        L.marker([pharmacy.geometry.coordinates[1],pharmacy.geometry.coordinates[0]]).addTo(osmMap).bindPopup(`<strong class="pharmacyName">藥局名稱：${pharmacy.properties.name}</strong><br>
        <div class="pharmacyDes">電話：${pharmacy.properties.phone}<br>位置：${pharmacy.properties.address}<br>
        成人口罩：<strong>${pharmacy.properties.mask_adult}</strong><br>兒童口罩：<strong>${pharmacy.properties.mask_child}</strong><br>最後更新時間：${pharmacy.properties.updated}</div>
        `)
      })
    }
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
