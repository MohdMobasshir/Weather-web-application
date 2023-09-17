<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        dark
        borderless
        >
        
        <template v-slot:before>
          <q-icon 
            @click="getLocation"
            name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn 
            @click="getWeatherBySearch"
            icon="search" 
            dense 
            flat
            round  
            />
        </template>
      </q-input>
    </div>

  <template v-if="weatherData" >
    <div class="col text-white text-center">
      <div class="text-h4 text-weight-light">
        {{ weatherData.name }}
      </div>
      <div class="text-h6 text-weight-light">
        {{ weatherData.weather[0].main }}
      </div>
      <div class="text-h1 text-weight-thin q-my-lg relative -osition">
        <span> {{ Math.round(weatherData.main.temp) }} </span> 
        <span class="text-h4 relative-position degree "> &deg;C </span> 
      </div>
    </div>

    <div class="col text-center">
      <img :src="`https://openweathermap.org/img/wn/${ weatherData.weather[0].icon }@2x.png`" >
    </div>
  </template>

  <template v-else>
    <div class="col column text-center text-white">
      <div class="col text-h2 text-weight-thin">
        Weatherify
      </div>

      <q-btn 
        @click="getLocation"
        class="col" 
        flat
      >
        <q-icon 
          left size="3em" 
          name="my_location" />
        <div>Find my Location</div>
      </q-btn>

    </div>
  </template>

    <div class="col skyline">
    </div>
  </q-page>
</template>

<script>
import axios from 'axios';
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'PageIndex',
  data() {
    return{
      search:'',
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather',
      apiKey: '6e504104aa87d3bb360118a96bcba994'
    }
  },
  computed:{
    bgClass() {
      if(this.weatherData) {
        if(this.weatherData.weather[0].icon.endsWith('d')) {
          return 'bg-day'
        }
        else {
          return 'bg-night'
        }
      }
    }
  },

  methods: {
    getLocation() {

      this.$q.loading.show()

      if(this.$q.platform.is.electron) {
        this.$axios.get('https://freegeoip.app/json/').then(response => {
          this.lat = response.data.latitude
          this.lon = response.data.longitude
          this.getWeatherByCoords()
        })
      }

      else {
        navigator.geolocation.getCurrentPosition
        (position => {
          console.log('position: ', position)
          this.lat = position.coords.latitude
          this.lon = position.coords.longitude
          this.getWeatherByCoords()
      })
      }
    },

    getWeatherByCoords() {
      this.$q.loading.show()
      this.$axios(`${ this.apiUrl }?lat=${ this.lat }&lon=${ this.lon }&appid=${ this.apiKey }&units=metric`).then(response => {
        this.weatherData=response.data
        this.$q.loading.hide()
      })
    },

    getWeatherBySearch() {
      this.$q.loading.show()
      this.$axios(`${ this.apiUrl }?q=${ this.search }&appid=${ this.apiKey }&units=metric`).then(response => {
        this.weatherData=response.data
        this.$q.loading.hide()
      })
    }
  }
})
</script>

<style lang="sass">
  .q-page
    background: linear-gradient(to bottom, #136a8a, #267871)
    &.bg-night
      background: linear-gradient(to bottom, #232526, #414345)
    &.bg-day
      background: linear-gradient(to bottom, #1fa2ff, #12d8fa, #a6ffcb)
  .degree
    top: -44px
  .skyline
    flex: 0 0 100px
    background: url(../statics/skyline.png)
    background-size: contain
    background-position: center-botttom
</style>