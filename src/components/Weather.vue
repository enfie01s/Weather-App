<script>
import DayThumb from './DayThumb.vue'
import WeatherDetail from './WeatherDetail.vue'

export default {
  name: "WeatherApp",
  components: {
    DayThumb,
    WeatherDetail,
  },
  data() {
    return {
      apiKey: import.meta.env.VITE_WEATHER_API_KEY,
      location: "auto:ip",
      weather: undefined,
      alerts: undefined,
      search: "",
      newLocation: "",
      locations: [],
      detailDialog: false,
      detail: {},
      weatherConditions: {
        'grey': ["drizzle", "rain", "overcast"],
        'lightblue': ["mist"],
        'lightgrey': ["snow", "blizzard"],
        'darkgrey': ["thunder", "lightning", "storm"],
      },
    }
  },
  methods: {
    getLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(this.updatePosition);
      }
    },
    updatePosition(position) {
      if (position.coords) {
        this.location = position.coords.latitude + "," + position.coords.longitude;
      }
    },
    load() {
      let url = "http://api.weatherapi.com/v1/forecast.json?key=" + this.apiKey
        + "&q=" + this.location + "&aqi=no&days=4";

      fetch(url)
        .then((response) => {
          return response.json();
        })
        .then((data) => {
          this.updateWeather(data)
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    searchLocation() {
      let url = "http://api.weatherapi.com/v1/search.json?key=" + this.apiKey
        + "&q=" + this.search;

      fetch(url)
        .then((response) => {
          return response.json();
        })
        .then((data) => {
          this.updateLocations(data)
        })
        .catch(function(error) {
          console.log(error);
        });

    },
    updateWeather(data) {
      this.weather = data;
    },
    updateLocations(data) {
      this.locations = data.map(item => ({
        'title': item.name + ", " + item.region + ", " + item.country,
        'location': item.name
      }));
    },
    showDetail(data) {
      this.detail = data;
      this.detailDialog = true;
    },
  },
  created() {
    this.load();  // Initial load before we ask permission to get location from browser
    this.getLocation();
  },
  watch: {
    location() {
      this.load();
    },
    newLocation() {
        this.location = this.newLocation;
    }
  },
  computed: {
    forecastDays() {
      if (this.weather != undefined) {
        return this.weather.forecast.forecastday;
      }
      return [];
    },
    updatedTime() {
      let dateObj = new Date(this.weather.current.last_updated_epoch * 1000);
      return dateObj.toLocaleTimeString().slice(0, -3);
    },
    bodyWeather() {
      if (this.weather != undefined) {
        let condition = this.weather.current.condition.text.replace(" ", "-").toLowerCase();
        for (let index in this.weatherConditions) {
          let conditions = this.weatherConditions[index];
          if (conditions.includes(condition)) {
            return index;
          }
        }
      }
      return "sunny";
    },
    bodySeason() {
      let today = new Date();
      let month = today.getMonth();
      if ([11,0,1].includes(month)) return "winter";
      if ([2,3,4].includes(month)) return "spring";
      if ([5,6,7].includes(month)) return "summer";
      if ([8,9,10].includes(month)) return "autumn";
      return "summer";
    },
  },
}
</script>

<template>
  <div v-if="weather != undefined" class="page" :class="[bodyWeather, bodySeason]">
    <div class="header">
      <div class="clouds_one"></div>
      <div class="clouds_two"></div>
      <div class="clouds_three"></div>
      <img class="cloudbg" src="../assets/bg.gif" alt="">
      <v-img :width="100" :height="220" aspect-ratio="16/9"
        cover src="../src/assets/bg.gif"></v-img>
    </div>

    <v-container class="w-100 content">
      <v-row no-gutters>
        <DayThumb :data="weather.current" :location="weather.location.name"
          key="current" :date="weather.current.last_updated_epoch * 1000"
          class="mx-auto mx-sm-0 ma-2 pa-2 v-col-12 v-col-sm-4 v-col-md-4" />
      </v-row>
      <v-row no-gutters>
        <v-autocomplete clearable v-model="newLocation" v-model:search="search"
          :items="locations" item-title="title" item-value="location"
          density="comfortable" hide-no-data hide-details variant="solo"
          label="Search by city" class="mt-md-2 ma-2"
          @input="searchLocation"></v-autocomplete>
      </v-row>
      <v-row no-gutters>
        <DayThumb v-for="day, key in forecastDays"
          :date="day.date_epoch * 1000" :data="day.day" :key="key"
          @showdetail="showDetail(day)" :is-day="true"
          class="mx-auto ma-2 pa-2 v-col-sm-3 v-col-12" />
      </v-row>
    </v-container>

    <WeatherDetail :dialog="detailDialog" :data="detail" @close="detailDialog = false" />
  </div>
</template>

<style lang="scss">
  $clouds: "../assets/clouds/";
  .page {
    min-height: 100vh;
    background-repeat: repeat-x;
    background-size: auto 25vh;
    background-blend-mode: overlay;
    background-attachment: fixed;
    &.winter { background-image: url('../assets/trees/winter.png'); }
    &.spring { background-image: url('../assets/trees/spring.png'); }
    &.summer { background-image: url('../assets/trees/summer.png'); }
    &.autumn { background-image: url('../assets/trees/autumn.png'); }
    background-position: bottom;
    $start: #014e9b;
    $middle: #1084C0;
    $main: #4d95be;
    background-color: $main;
    .cloudbg {
      background: $main;
      background: -webkit-linear-gradient(top, $start 0, $middle 68%, $main 100%);
      background: linear-gradient(to bottom, $start 0, $middle 68%, $main 100%);
      filter: progid:DXImageTransform.Microsoft.gradient(startColorstr=$start, endColorstr=$main, GradientType=0);
    }
    &.lightblue {
      $cloudbg1: #718fad !important;
      $cloudbg2: #93bbcd !important;
      $pagebg: #a4c1d3 !important;
      background-color: $pagebg;
      .cloudbg {
        background: $pagebg;
        background: -webkit-linear-gradient(top, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        background: linear-gradient(to bottom, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        filter: progid:DXImageTransform.Microsoft.gradient(startColorstr=$cloudbg1, endColorstr=$pagebg, GradientType=0);
      }
    }
    &.grey {
      $cloudbg1: #517394 !important;
      $cloudbg2: #74a4bb !important;
      $pagebg: #89a7b9 !important;
      background-color: $pagebg;
      .cloudbg {
        background: $pagebg;
        background: -webkit-linear-gradient(top, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        background: linear-gradient(to bottom, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        filter: progid:DXImageTransform.Microsoft.gradient(startColorstr=$cloudbg1, endColorstr=$pagebg, GradientType=0);
      }
    }
    &.darkgrey {
      $cloudbg1: #2c3f51 !important;
      $cloudbg2: #445f6d !important;
      $pagebg: #566873 !important;
      background-color: $pagebg;
      .cloudbg {
        background: $pagebg;
        background: -webkit-linear-gradient(top, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        background: linear-gradient(to bottom, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        filter: progid:DXImageTransform.Microsoft.gradient(startColorstr=$cloudbg1, endColorstr=$pagebg, GradientType=0);
      }
    }
    &.lightgrey {
      $cloudbg1: #bfbfbf !important;
      $cloudbg2: #d1d1d1 !important;
      $pagebg: #e4e4e4 !important;
      background-color: $pagebg;
      .cloudbg {
        background: $pagebg;
        background: -webkit-linear-gradient(top,$cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        background: linear-gradient(to bottom, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        filter: progid:DXImageTransform.Microsoft.gradient(startColorstr=$cloudbg1, endColorstr=$pagebg, GradientType=0);
      }
    }
    .cloudbg {
      left: 0;
      position: absolute;
      top: 0;
      width: 100% !important;
      z-index: 1;
    }
  }
  .v-container {
    max-width: 100% !important;
  }
  .blur,
  .v-field--variant-solo {
    -webkit-backdrop-filter: blur(10px);
    backdrop-filter: blur(10px);
    background-color: rgba(255, 255, 255, 0.5) !important;
  }
  .content {
    z-index: 4;
  }
  .header {
    line-height: 1.5;
    max-height: 100vh;
    height: 270px\9;
    line-height: 0;
    margin: 0;
    overflow: hidden;
    position: absolute;
    z-index: 1;
    &, & img { width: 100%; }
  }
  .clouds_one,
  .clouds_two,
  .clouds_three {
    background-size: contain;
    background-repeat: repeat;
    height: 100%;
    transform: translate3d(0,0,0);
    width: 500%;
    z-index: 3;
  }
  .clouds_one,
  .clouds_two,
  .clouds_three {
    -webkit-animation-duration: 90s;
    -webkit-animation-iteration-count: 1;
    -webkit-animation-timing-function: linear;
    -webkit-animation-fill-mode: forwards;
    -o-animation-duration: 90s;
    -o-animation-iteration-count: 1;
    -o-animation-timing-function: linear;
    -o-animation-fill-mode: forwards;
    animation-duration: 90s;
    animation-iteration-count: 1;
    animation-timing-function: linear;
    animation-fill-mode: forwards;
    position: absolute;
    top: 0;
  }
  .clouds_two,
  .clouds_three { left:0; }
  .clouds_one {
    left: -14%;
    background-image: url($clouds + 'cloud_1.png');
    -webkit-animation-name: cloud_one;
    -o-animation-name: cloud_one;
    animation-name: cloud_one
  }
  .clouds_two {
    background-image: url($clouds + 'cloud_2.png');
    -webkit-animation-name: cloud_two;
    -o-animation-name: cloud_two;
    animation-name: cloud_two
  }
  .clouds_three {
    background-image: url($clouds + 'cloud_3.png');
    -webkit-animation-name: cloud_three;
    -o-animation-name: cloud_three;
    animation-name: cloud_three;
  }

  // Animations
  @keyframes cloud_one {
    0% { left: -14%; }
    95% { left: -314%; }
    100% { left: -314%; }
  }
  @keyframes cloud_two {
    0% { left: 0; }
    95% { left: -200%; }
    100% { left: -200%; }
  }
  @keyframes cloud_three {
    0% { left: 0; }
    95% { left: -100%; }
    100% { left: -100%; }
  }
</style>
