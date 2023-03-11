<script>
import DayThumb from './DayThumb.vue'

export default {
  name: "WeatherDetail",
  components: { DayThumb },
  props: {
    dialog: {
      type: Boolean,
      default: false
    },
    data: {
      type: Object,
      default: undefined
    },
  },
  data() {
    return {
      dialogVisible: false,
      hour_scroll: null,
    }
  },
  methods: {
    scrollToHour() {
      let time = new Date();
      const hour = document.getElementById('hour' + time.getHours());
      hour.scrollIntoView({
        behavior: 'auto',
        block: 'center',
        inline: 'center'
      });
    }
  },
  computed: {
    formattedDate() {
      if (this.isToday) return "Today";
      let dateObj = new Date(this.data.date);
      return dateObj.toDateString().slice(0, -5);
    },
    dataHours() {
      if (this.data != undefined) {
        return this.data.hour;
      }
      return [];
    },
    isToday() {
      let today = new Date();
      let selectedDay = new Date(this.data.date);
      return today.toDateString() == selectedDay.toDateString();
    },
  },
  watch: {
    dialog() {
      this.dialogVisible = this.dialog;
      if (this.dialog && this.isToday) {
        this.$nextTick(() => {
          this.scrollToHour();
        });
      }
    },
  },
}
</script>

<template>
  <v-row justify="center" v-show="dialogVisible" transition="fade-transition">
    <v-dialog v-model="dialogVisible" scrollable
      persistent width="auto">
      <v-card>
        <v-toolbar :title="'Hourly Forecast for ' + formattedDate">
          <v-btn icon="mdi-close" color="blue-darken-1" @click="$emit('close')"></v-btn>
        </v-toolbar>
        <v-divider></v-divider>
        <v-card-text style="width: 100%;">
          <div class="d-flex align-start" id="hour-holder">
            <DayThumb v-for="hour, key in dataHours" :is-hour="true" min-width="30vw"
              :date="hour.time_epoch * 1000" :data="hour" :key="key"
              class="mx-auto ma-2 pa-2 v-col-2" :id="'hour' + key" :is-forecast="false" />
          </div>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<style scoped>
#hour-holder > div {
  min-width: 10em;
}
</style>
