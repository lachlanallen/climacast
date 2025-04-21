<template>
  <div id="app">
    <main>
      <section class="nav-container">
        <div class="input-group">
          <!-- Search Icon -->
          <span class="input-group-text">
            <i class="fa-solid fa-magnifying-glass"></i>
          </span>
          <!-- Search Input -->
          <input type="text" class="form-control" v-model="searchQuery" @keydown.enter="fetchWeather"
            placeholder="Search for a location.." />
        </div>
        <!-- Bookmark Button -->
        <button class="btn btn-outline-secondary"><i class="fa-regular fa-bookmark"></i></button>
      </section>
      <section class="main-weather" :class="{ night: isNight }">
        <div class="weather-info" v-if="weather && weather.main">
          <i class="fa-solid fa-cloud weather-icon"></i>
          <h2 class="city-name">{{ weather.name }}</h2>
          <p class="location">{{ weather.name }}, {{ weather.sys?.country }}</p>
          <p class="current-temp">{{ Math.round(weather.main.temp) }}°F</p>
          <p class="weather-condition">{{ weather.weather[0].main }}</p>
          <p class="high-low">H:{{ Math.round(weather.main.temp_max) }}°F L:{{ Math.round(weather.main.temp_min) }}°F
          </p>
        </div>
        <div v-else>
          <p>Loading weather data...</p>
        </div>
      </section>
      <section class="forecast-container">
        <h2>Forecasts</h2>
        <div class="button-container">
          <button class="btn btn-outline-secondary">5-Days</button>
          <button class="btn btn-outline-secondary">Hourly</button>
        </div>
      </section>
    </main>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      api_key: '4678aa73a2c81a5a92de8b076aef4fb8',
      url_base: 'https://api.openweathermap.org/data/2.5/',
      searchQuery: '',
      weather: {},
      sunrise: null,
      sunset: null,
    };
  },
  computed: {
    isNight() {
      if (!this.sunrise || !this.sunset) return false;

      const currentTime = new Date().getTime() / 1000;
      return currentTime < this.sunrise || currentTime > this.sunset;
    },
  },
  methods: {
    fetchWeather(event) {
      if (event.key === 'Enter') {
        fetch(`${this.url_base}weather?q=${this.searchQuery}&APPID=${this.api_key}&units=imperial`)
          .then(response => response.json())
          .then(data => {
            this.setResults(data);
          })
          .catch(error => console.error('Error fetching weather data:', error));
      }
    },
    setResults(results) {
      this.weather = results;
      if (results.sys) {
        this.sunrise = results.sys.sunrise;
        this.sunset = results.sys.sunset;
      }
    },
  },
};
</script>

<style>
* {
  font-family: "Poppins", sans-serif;
}

/* Nav Bar Styles */
.nav-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  padding: 1rem 2rem;
}

.search-bar {
  min-width: 60vw;
}

/* Main Weather Styles */
.main-weather {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  background-image: url('@/assets/day-sky.jpg');
  background-size: cover;
  min-height: 60vh;
  margin: 1rem 2rem;
  padding: 2rem;
  border-radius: 15px;
  transition: 300ms ease-out;
}

.main-weather.night {
  background-image: url('@/assets/night-sky.jpg');
}

.weather-info {
  background-color: rgba(255, 255, 255, 0.65);
  padding: 6rem;
  min-width: 60vw;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.weather-icon {
  font-size: 3rem;
}

.city-name {
  font-size: 2rem;
}

.current-temp {
  font-size: 4rem;
  font-weight: 500;
}

.forecast-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  margin-top: 2rem;
}

.button-container {
  display: flex;
  gap: 2rem;
  margin-top: 1rem;
}
</style>