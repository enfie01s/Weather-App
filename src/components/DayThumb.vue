<script>
export default {
  name: "DayThumb",
  props: {
    data: {
      type: Object,
      default: undefined
    },
    location: {
      type: String,
      default: ''
    },
    date: {
      type: [String, Number],
      default: ''
    },
    isDay: {
      type: Boolean,
      default: false
    },
    isHour: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    formattedDate() {
      let dateObj = new Date(this.date);
      if (this.isHour) return dateObj.toLocaleTimeString().slice(0, -3);
      if (this.isToday) return "Today";
      return dateObj.toDateString().slice(0, -5);
    },
    subTitle() {
      if (this.isDay) return this.data.daily_chance_of_rain + '% Chance of precipitation';
      else if (this.isHour) return this.data.chance_of_rain + '% Chance of precipitation';
      else return this.data.condition.text;
    },
    title() {
      return this.isDay || this.isHour ? this.formattedDate : this.location;
    },
    temp() {
      return this.isDay ? this.data.avgtemp_c : this.data.temp_c;
    },
    isToday() {
      let today = new Date();
      let selectedDay = new Date(this.date);
      return today.toDateString() == selectedDay.toDateString();
    },
  },
}
</script>

<template>
  <div v-if="data != undefined">
    <v-card class="mx-auto" :class="{'blur': !isDay && !isHour, 'text-center': isHour}">
      <v-card-item :title="title">
        <template v-slot:subtitle>
          {{ subTitle }}
        </template>
      </v-card-item>
      <v-card-text class="py-0">
        <v-row align="center" no-gutters>
          <span class="text-body-2 text-sm-h6 text-md-h5" :class="{'mx-auto': isHour}">
            <img :src="data.condition.icon" alt="" :class="!isHour ? 'weather-icon' : 'weather-icon-large'" /> {{ temp }}&deg;C
          </span>
        </v-row>
      </v-card-text>
      <v-divider v-if="isDay"></v-divider>
      <v-card-actions v-if="isDay">
        <v-btn @click="$emit('showdetail')" class="mx-auto">
          More Detail
        </v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>

<style>
.weather-icon {
  height: 1.5em;
}
.weather-icon-large {
  display: block;
  height: 2.5em;
}
.v-card-subtitle {
  white-space: normal !important;
}
</style>
