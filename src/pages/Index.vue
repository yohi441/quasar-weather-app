<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search City"
        dark
        borderless>
        <template v-slot:before>
          <q-icon
          @click="getLocation"
            name="my_location" />
        </template>


        <template v-slot:append>
          <q-btn
            @click="getWeatherBySearch"
            round
            dense
            flat
            icon="search" />
        </template>
      </q-input>
    </div>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
          </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>
      <div class="col text-center">
        <img :src="`https://openweathermap.org/img/wn/${ weatherData.weather[0].icon }@2x.png`"/>
      </div>
      <q-btn
            @click="getLocation"
            class="col"
            flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find My Location</div>
          </q-btn>
    </template>
    <template v-if="!weatherData">
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Weather App
        </div>
          <q-btn
            @click="getLocation"
            class="col"
            flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find My Location</div>
          </q-btn>
      </div>
    </template>
    <div>
        <q-dialog v-model="alert">
        <q-card>
          <q-card-section>
            <div class="text-h6">Not Found</div>
          </q-card-section>

          <q-card-section class="q-pt-none">
            Invalid City
          </q-card-section>

          <q-card-actions align="right">
            <q-btn flat label="OK" color="primary" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>

    <div class="col skyline"></div>
  </q-page>
</template>


<script>
export default {
  name: 'PageIndex',

  data() {
    return {
      alert: false,
      search: null,
      weatherData: null,
      lat: null,
      lon: null,
      apiKey: process.env.WEATHER_API_KEY,
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather'
    }
  },
  methods: {
    getLocation() {
      this.$q.loading.show()
      navigator.geolocation.getCurrentPosition(
        position => {
          this.lat = position.coords.latitude
          this.lon = position.coords.longitude
          this.getWeatherByCoords()
        }
      )
    },
    getWeatherByCoords() {
      this.$q.loading.show()
      this.$axios(`${ this.apiUrl }?lat=${ this.lat }&lon=${ this.lon }&appid=${ this.apiKey }`).then(response => {
        this.weatherData = response.data
        this.$q.loading.hide()
      }).catch(err => {
        this.$q.loading.hide()
      })
    },  
    getWeatherBySearch() {
      if (this.search){
          this.$q.loading.show()
          this.$axios(`${ this.apiUrl }?q=${ this.search }&appid=${ this.apiKey }`).then(response => {
          this.weatherData = response.data
          this.$q.loading.hide()
          }).catch(err => {

            this.$q.loading.hide()
            this.alert = true
            this.search = ''
          })
      }
      
    }
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        }
        else {
          return 'bg-day'
        }
      }
    }
  }
}
</script>

<style lang="sass">
  .q-page
    background: linear-gradient(to bottom, #136a8a, #267871)
    &.bg-night
      background: linear-gradient(to bottom, #232526, #414345)
    &.bg-day
      background: linear-gradient(to bottom, #00b4db, #0083b0)

  .degree
    top: -44px

  .skyline
    flex: 0 0 100px
    background: url(../statics/skyline.png)
    background-size: contain
    background-position: center bottom
  
  
</style>