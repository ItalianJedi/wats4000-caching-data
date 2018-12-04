<template>
  <div>
    <favorite-cities v-bind:favoriteCities="favoritesList"></favorite-cities>
    <h2>City Search</h2>
    <message-container v-bind:messages="messages"></message-container>
    <form v-on:submit.prevent="getCities">
        <p>Enter city name: <input type="text" v-model="query" placeholder="Paris, TX"> <button type="submit">Go</button></p>
    </form>
    <load-spinner v-if="showLoading"></load-spinner>
    <ul class="cities" v-if="results && results.list.length > 0">
      <li v-for="city in results.list">
        <h2>{{ city.name }}, {{ city.sys.country }}</h2>
        <p><router-link v-bind:to="{ name: 'CurrentWeather', params: { cityId: city.id } }">View Current Weather</router-link></p>

        <weather-summary v-bind:weatherData="city.weather"></weather-summary>

        <weather-data v-bind:weatherData="city.main"></weather-data>
        <p><button class="save" v-on:click="saveCity(city)">Save City to Favorites</button></p>
      </li>
    </ul>
  </div>
</template>

<script>
import {API} from '@/common/api';
import WeatherSummary from '@/components/WeatherSummary';
import WeatherData from '@/components/WeatherData';
import CubeSpinner from '@/components/CubeSpinner';
import MessageContainer from '@/components/MessageContainer';
import FavoriteCities from '@/components/FavoriteCities';


export default {
  name: 'CitySearch',
  components: {
    'weather-summary': WeatherSummary,
    'weather-data': WeatherData,
    'load-spinner': CubeSpinner,
    'message-container': MessageContainer,
    'favorite-cities' : FavoriteCities
  },
  data () {
    return {
      results: null,
      query: '',
      showLoading: false,
      messages: [],
      favoritesList: []
    }
  },
  created () {
    if (this.$ls.get('favoriteCities')) {
      this.favoritesList = this.$ls.get('favoriteCities');
    }

  },
  methods: {
    saveCity: function (city) {
      this.favoritesList.push(city);
      this.$ls.set('favoriteCities', this.favoritesList);
    },
    getCities: function () {
      this.results = null;
      this.showLoading = true;

      let cacheLabel = `citySearch_${this.query}`;

      let cacheExpiry = 15 * 60 * 1000;

      // TODO: Wrap this API call in a conditional to check if the request should be made.
      // Use this.$ls.get() to check if there is a cached query
      // If there is a cached query, use that data instead of making an API request
      // If not, make the API request and then cache the value for the amount of time specified in `cacheExpiry`
      if(!this.$ls.get(cacheLabel)){
      //No cache exists. Perform API call.
      console.log(`No cache detected for ${cacheLabel}`);
      API.get('find', {
        params: {
            q: this.query
        }
      })
      .then(response => {
        this.results = response.data;
        //Save API results to localStorage cache
        this.$ls.set(cacheLabel, this.results, cacheExpiry);
        this.showLoading = false;
      })
      .catch(error => {
        this.messages.push({
          type: 'error',
          text: error.message
        });
        this.showLoading = false;
      });
      } else {
        //Cache exists.
        console.log(`Valid cache detected for ${cacheLabel}.`);
        this.results = this.$ls.get(cacheLabel);
        this.showLoading = false;
      }
    }
  }
}
</script>

<style scoped>
.errors li {
  color: red;
  border: solid red 1px;
  padding: 5px;
}
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  width: 300px;
  min-height: 300px;
  border: solid 1px #e8e8e8;
  padding: 10px;
  margin: 5px;
}



a {
  color: #42b983;
}
</style>


