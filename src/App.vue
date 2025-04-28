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
        <button class="btn btn-outline-secondary" @click="showBookmarks = !showBookmarks">
          <i class="fa-regular fa-bookmark"></i>
        </button>
      </section>

      <!-- Bookmarks Side Menu -->
      <aside class="bookmarks-menu" v-if="showBookmarks">
        <div class="bookmarks-header">
          <h3>Saved Locations</h3>
          <button class="btn btn-outline-secondary" @click="showBookmarks = false">
            <i class="fa-solid fa-xmark"></i>
          </button>
        </div>
        <ul>
          <li v-for="bookmark in bookmarks" :key="bookmark.name">
            <span @click="selectBookmark(bookmark.name)">
              {{ bookmark.name }}, {{ bookmark.country }}
            </span>
            <button @click="removeBookmark(bookmark.name)"><i class="fa-solid fa-xmark"></i></button>
          </li>
        </ul>
      </aside>

      <!-- Main Weather Component -->
      <section class="main-weather" :class="{ night: isNight }">
        <div class="weather-info" v-if="weather && weather.main">
          <!-- Bookmark Button -->
          <button class="bookmark-btn" @click="toggleBookmark" :class="{ saved: isBookmarked }">
            <i :class="isBookmarked ? 'fa-solid fa-bookmark' : 'fa-regular fa-bookmark'"></i>
          </button>
          <!-- Weather Icon and Info -->
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
          <button class="btn btn-outline-secondary" @click="fetchHourlyForecast">Hourly</button>
        </div>
        <!-- Charts -->
        <!-- Hourly Temperature Chart -->
        <canvas id="hourlyChart" width="800" height="300"></canvas>
        <!-- Hourly Rainfall Chart -->
        <canvas id="rainfallChart" width="800" height="300"></canvas>
      </section>
    </main>
  </div>
</template>

<script>
import { Chart, registerables } from 'chart.js';
Chart.register(...registerables);

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
      hourlyData: [],
      chart: null,
      bookmarks: [],
      showBookmarks: false,
    };
  },
  computed: {
    isNight() {
      if (!this.sunrise || !this.sunset) return false;

      const currentTime = new Date().getTime() / 1000;
      return currentTime < this.sunrise || currentTime > this.sunset;
    },
    isBookmarked() {
      return this.bookmarks.some(b => b.name === this.weather.name);
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
    fetchHourlyForecast() {
      fetch(`${this.url_base}forecast?q=${this.searchQuery}&APPID=${this.api_key}&units=imperial`)
        .then(response => response.json())
        .then(data => {
          this.hourlyData = data.list.slice(0, 8);
          this.updateChart();
          this.updateRainfallChart();
        })
        .catch(error => console.error('Error fetching hourly forecast:', error));
    },
    updateRainfallChart() {
      const labels = this.hourlyData.map(item => {
        const date = new Date(item.dt * 1000);
        let hours = date.getHours();
        const ampm = hours >= 12 ? 'PM' : 'AM';
        // Convert to 12-hour clock format
        hours = hours % 12 || 12;
        return `${hours}${ampm}`;
      });

      const rainfall = this.hourlyData.map(item => {
        const rainVolume = item.rain?.['3h'] || 0;
        // Ensure rainfall chart only shows positive values
        const rainInInches = Math.max(0, rainVolume / 25.4);
        return rainInInches.toFixed(2);
      });

      if (this.rainfallChart) {
        this.rainfallChart.destroy();
      }

      const ctx = document.getElementById('rainfallChart').getContext('2d');
      this.rainfallChart = new Chart(ctx, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              data: rainfall,
              borderColor: 'rgba(54, 162, 235, 1)',
              backgroundColor: 'rgba(54, 162, 235, 0.2)',
              borderWidth: 2,
              tension: 0.4,
              fill: true,
              pointBackgroundColor: 'rgba(54, 162, 235, 1)',
              pointBorderColor: 'rgba(54, 162, 235, 1)',
              pointRadius: 2,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: true,
          plugins: {
            legend: {
              display: false,
            },
            title: {
              display: true,
              text: 'Precipitation',
              font: {
                family: 'Poppins',
                size: 18,
                weight: '400',
              },
              padding: {
                top: 10,
                bottom: 10,
              },
            },
          },
          scales: {
            x: {
              title: {
                display: false,
              },
            },
            y: {
              // Ensure rainfall chart only shows positive values
              beginAtZero: true,
              min: 0,
              title: {
                display: false,
              },
            },
          },
        },
      });
    },
    updateChart() {
      const labels = this.hourlyData.map(item => {
        const date = new Date(item.dt * 1000);
        let hours = date.getHours();
        const ampm = hours >= 12 ? 'PM' : 'AM';
        // Convert to 12-hour format
        hours = hours % 12 || 12;
        return `${hours}${ampm}`;
      });
      const temperatures = this.hourlyData.map(item => Math.round(item.main.temp));

      if (this.chart) {
        this.chart.destroy();
      }

      const ctx = document.getElementById('hourlyChart').getContext('2d');
      this.chart = new Chart(ctx, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              data: temperatures,
              borderColor: 'rgba(75, 192, 192, 1)',
              backgroundColor: 'rgba(75, 192, 192, 0.2)',
              borderWidth: 2,
              tension: 0.4,
              fill: true,
              pointBackgroundColor: 'rgba(75, 192, 192, 1)',
              pointBorderColor: 'rgba(75, 192, 192, 1)',
              pointRadius: 2,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: true,
          plugins: {
            legend: {
              display: false,
            },
            title: {
              display: true,
              text: 'Temperature',
              font: {
                family: 'Poppins',
                size: 18,
                weight: '400',
              },
              padding: {
                top: 10,
                bottom: 10,
              },
            },
          },
          scales: {
            x: {
              title: {
                display: false
              },
            },
            y: {
              title: {
                display: false
              },
            },
          },
        },
      });
    },
    addBookmark() {
      if (this.weather.name && !this.bookmarks.some(b => b.name === this.weather.name)) {
        this.bookmarks.push({
          name: this.weather.name,
          country: this.weather.sys?.country,
        });
      }
    },
    removeBookmark(locationName) {
      this.bookmarks = this.bookmarks.filter(b => b.name !== locationName);
    },
    selectBookmark(locationName) {
      this.searchQuery = locationName;
      this.fetchWeather({ key: 'Enter' });
      this.showBookmarks = false;
    },
    toggleBookmark() {
      if (this.isBookmarked) {
        // Remove the location if it's already bookmarked
        this.bookmarks = this.bookmarks.filter(b => b.name !== this.weather.name);
      } else {
        // Add the location to bookmarks
        if (this.weather.name) {
          this.bookmarks.push({
            name: this.weather.name,
            country: this.weather.sys?.country,
          });
        }
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
  position: relative;
  background-color: rgba(255, 255, 255, 0.65);
  padding: 6rem;
  min-width: 60vw;
  max-width: 80vw;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.bookmark-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  font-size: 1.5rem;
  color: #333;
  cursor: pointer;
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

#hourlyChart,
#rainfallChart {
  font-family: "Poppins", sans-serif;
  max-width: 90vw;
  height: auto;
  margin: 1rem auto;
}

.bookmarks-menu {
  position: fixed;
  top: 0;
  right: 0;
  width: 300px;
  height: 100%;
  background-color: #f8f9fa;
  box-shadow: -2px 0 5px rgba(0, 0, 0, 0.1);
  padding: 1rem;
  overflow-y: auto;
  z-index: 1000;
}

.bookmarks-menu h3 {
  margin-bottom: 1rem;
}

.bookmarks-menu ul {
  list-style: none;
  padding: 0;
}

.bookmarks-menu li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
  background-color: aliceblue;
  border: #6c757d 1px solid;
  border-radius: 15px;
  padding: 1rem;
}

.bookmarks-menu li span {
  cursor: pointer;
}

.bookmarks-menu li button {
  background: none;
  border: none;
  color: #333;
  cursor: pointer;
}

.bookmarks-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.bookmarks-header h3 {
  margin: 0;
}
</style>