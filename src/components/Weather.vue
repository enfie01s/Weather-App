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
    bodyClass() {
      if (this.weather != undefined) {
        let condition = this.weather.current.condition.text.replace(" ", "-").toLowerCase();
        if (condition.search("drizzle") != -1
          || condition.search("rain") != -1
          || condition.search("overcast") != -1) return "grey";
        if (condition.search("thunder") != -1
          || condition.search("lightning") != -1
          || condition.search("storm") != -1) return "dark-grey";
        if (condition.search("snow") != -1
          || condition.search("blizzard") != -1
          || condition.search("mist") != -1) return "light-grey";
      }
      return "sunny";
    },
  },
}
</script>

<template>
  <div v-if="weather != undefined" :class="'page ' + bodyClass">
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
          :key="key" :date="weather.current.last_updated_epoch * 1000"
          class="mx-auto mx-sm-0 ma-2 pa-2 v-col-12 v-col-sm-4 v-col-md-4" />
      </v-row>
      <v-row no-gutters>
        <v-autocomplete clearable v-model="newLocation" v-model:search="search"
          :loading="loading" :items="locations" item-title="title" item-value="location"
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
    height: 100%;
    $start: #014e9b;
    $middle: #1084C0;
    $main: #4d95be;
    background: $main;
    .cloudbg {
      background: $main;
      background: -webkit-linear-gradient(top, $start 0, $middle 68%, $main 100%);
      background: linear-gradient(to bottom, $start 0, $middle 68%, $main 100%);
      filter: progid:DXImageTransform.Microsoft.gradient(startColorstr=$start, endColorstr=$main, GradientType=0);
    }
    &.grey {
      $cloudbg1: #517394 !important;
      $cloudbg2: #74a4bb !important;
      $pagebg: #89a7b9 !important;
      background: $pagebg;
      .cloudbg {
        background: $pagebg;
        background: -webkit-linear-gradient(top, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        background: linear-gradient(to bottom, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        filter: progid:DXImageTransform.Microsoft.gradient(startColorstr=$cloudbg1, endColorstr=$pagebg, GradientType=0);
      }
    }
    &.dark-grey {
      $cloudbg1: #2c3f51 !important;
      $cloudbg2: #445f6d !important;
      $pagebg: #566873 !important;
      background: $pagebg;
      .cloudbg {
        background: $pagebg;
        background: -webkit-linear-gradient(top, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        background: linear-gradient(to bottom, $cloudbg1 0, $cloudbg2 68%, $pagebg 100%);
        filter: progid:DXImageTransform.Microsoft.gradient(startColorstr=$cloudbg1, endColorstr=$pagebg, GradientType=0);
      }
    }
    &.light-grey {
      $cloudbg1: #bfbfbf !important;
      $cloudbg2: #d1d1d1 !important;
      $pagebg: #e4e4e4 !important;
      background: $pagebg;
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
