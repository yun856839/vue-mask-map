<template>
  <div id="app">
    <div class="row no-gutters m-5">
      <!-- 所在縣市地區 -->
      <div class="toolbox col-md-3 p-2 bg-white">
        <div class="mb-3 form-group d-flex">
          <label for="cityName" class="col-sm-2 col-form-label text-right"
            >縣市：</label
          >
          <div class="flex-fill">
            <select id="cityName" class="form-control" v-model="select.city">
              <option value="" selected disabled>請選擇縣市</option>
              <option
                :value="city.CityName"
                v-for="city in cityName"
                :key="city.CityName"
              >
                {{ city.CityName }}
              </option>
              pt
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="area" class="col-sm-2 col-form-label text-right"
            >地區：</label
          >
          <div class="flex-fill">
            <select id="area" class="form-control" v-model="select.area">
              <option value="" selected disabled>請選擇地區</option>
              <option
                :value="area.AreaName"
                v-for="area in cityName.find(
                  (city) => select.city === city.CityName
                ).AreaList"
                :key="area.AreaName"
              >
                {{ area.AreaName }}
              </option>
            </select>
          </div>
        </div>
      </div>

      <!-- 藥局位置 -->
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
    <router-view />
  </div>
</template>

<script>
import L from "leaflet";
import "bootstrap";
import "bootstrap/dist/css/bootstrap.min.css";
import cityName from "./assets/cityName.json";

let openStreetMap = {};
var redIcon = L.icon({
  iconUrl:
    "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-red.png",
  shadowUrl:
    "https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png",
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
});

export default {
  name: "App",
  data() {
    return {
      data: [],
      cityName,
      select: {
        city: "臺北市",
        area: "中正區",
      },
    };
  },
  computed: {
    pharmaciesMark() {
      return this.data.filter((pharmacy) => {
        if (!this.select.area) {
          return pharmacy.properties.country === this.select.city;
        }
        return pharmacy.properties.town === this.select.area;
      });
    },
  },
  watch: {
    /* eslint-disable */
    pharmaciesMark(newValue) {
      this.updateMap();
    },
  },
  methods: {
    updateMap() {
      // 清除 Mark 避免圖層多於疊加
      openStreetMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          openStreetMap.removeLayer(layer);
        }
      });

      this.pharmaciesMark.forEach((pharmacy) => {
        L.marker(
          [pharmacy.geometry.coordinates[1], pharmacy.geometry.coordinates[0]],
          { icon: redIcon }
        )
          .addTo(openStreetMap)
          .bindPopup(
            `<p style="margin: 0"><strong style="font-size: 24px;">
            ${pharmacy.properties.name}</strong></p>
            <hr style="margin: 8px; border: 1px solid red"/>
            <strong style="font-size: 20px; color: blue;">口罩剩餘：成人 - ${
              pharmacy.properties.mask_adult
                ? `${pharmacy.properties.mask_adult} 個`
                : "未取得資料"
            } / 兒童 - ${
              pharmacy.properties.mask_child
                ? `${pharmacy.properties.mask_child} 個`
                : "未取得資料"
            }</strong><br>
            地址: ${pharmacy.properties.address}<br>
            電話: ${pharmacy.properties.phone}<br>
            <small style="color: red"><strong>最後更新時間: ${
              pharmacy.properties.updated
            }</strong></small>`
          );
      });
    },
  },
  async mounted() {
    const API =
      "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";
    const res = await this.$http.get(API);
    this.data = res.data.features;

    openStreetMap = L.map("map", {
      center: [23.982028, 120.6847551],
      zoom: 16,
    });
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution:
        '&copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a>',
    }).addTo(openStreetMap);
  },
};
</script>

<style scoped>
#map {
  width: 800px;
  height: 600px;
}
</style>


