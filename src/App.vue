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
import "bootstrap";
import "bootstrap/dist/css/bootstrap.min.css";
import cityName from "./assets/cityName.json";

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
  async mounted() {
    const API =
      "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";
    const res = await this.$http.get(API);
    this.data = res.data.features;
  },
};
</script>


