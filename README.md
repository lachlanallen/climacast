# Climacast 🌦️
Climacast is a weather forecasting web application built with Vue.js. It provides real-time weather information, a 5-day forecast, and hourly weather charts for any location around the globe. Users can bookmark their favorite locations for quick access.

## Features
- Current Weather: Displays the current temperature, weather condition, and high/low temperatures for the day.
- 5-Day Forecast: View the weather forecast for the next 5 days, including high/low temperatures and weather conditions.
- Hourly Charts: Visualize hourly temperature and rainfall data using interactive charts.
- Bookmarks: Save and manage your favorite locations for quick access.
- Day/Night Mode: Automatically adjusts the background based on the time of day.

## Technologies Used
- Vue.js: Frontend framework for building the user interface.
- Chart.js: For rendering interactive weather charts.
- OpenWeather API: For fetching weather and geolocation data.
- Bootstrap: For responsive design and styling.

### Installation
Prerequisites
- Node.js and npm installed on your system.
- A valid API key from OpenWeather.
1. Clone the repository:
```
git clone https://github.com/lachlanallen/climacast.git
cd climacast
```
2. Install dependencies:
```
npm install
```
3. Add your OpenWeather API key:
   - Open 'App.vue'.
   - Replace the 'api_key' value with your OpenWeather API key:
```
api_key: 'YOUR_API_KEY',
```
4. Start the development server:
```
npm run serve
```
5. Open the app in your browser
