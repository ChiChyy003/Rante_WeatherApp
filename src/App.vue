<template>
  <v-app :class="weatherBackground || 'landing-page'">
    <v-container>
      <v-row justify="center">
        <v-col cols="12" md="6">
          <v-card class>
            <v-card-title>
              <h2 class="text-center">Weather App</h2>
            </v-card-title>
            <v-card-text>
              <v-text-field
                v-model="city"
                label="Enter city"
                @keyup.enter="fetchWeather"
                outlined
              ></v-text-field>
              <v-btn color="primary" @click="fetchWeather">
                Get Weather
              </v-btn>
              <v-alert v-if="error" type="error">{{ error }}</v-alert>
              <v-card v-if="weather.length" class="mt-4">
                <v-row v-for="(cityWeather, index) in weather" :key="index" class="mb-3">
                  <v-col>
                    <v-card class="pa-3">
                      <v-card-title class="text-h5">{{ cityWeather.name }}</v-card-title>
                      <v-card-subtitle>
                        {{ cityWeather.weather[0].description }}
                      </v-card-subtitle>
                      <v-card-text>
                        <p><strong>Temperature:</strong> 
                          {{ cityWeather.main.temp }} °C / 
                          {{ convertToFahrenheit(cityWeather.main.temp) }} °F
                        </p>
                        <p><strong>Humidity:</strong> {{ cityWeather.main.humidity }}%</p>
                        <p><strong>Wind Speed:</strong> {{ cityWeather.wind.speed }} m/s</p>
                        <v-chip class="ma-1" color="indigo" text-color="white">
                          Pressure: {{ cityWeather.main.pressure }} hPa
                        </v-chip>
                        <v-chip class="ma-1" color="deep-purple" text-color="white">
                          Sunrise: {{ formatTime(cityWeather.sys.sunrise) }}
                        </v-chip>
                        <v-chip class="ma-1" color="orange" text-color="white">
                          Sunset: {{ formatTime(cityWeather.sys.sunset) }}
                        </v-chip>
                      </v-card-text>
                      <v-btn color="error" @click="removeCity(index)">Remove</v-btn>
                    </v-card>
                  </v-col>
                </v-row>
              </v-card>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
    <!-- Snow effect if it's snowy -->
    <div v-if="weatherBackground === 'snowy'" class="snow"></div>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      city: '',
      weather: [],
      error: ''
    };
  },
  methods: {
    async fetchWeather() {
      if (this.city.trim() === '') {
        this.error = 'Please enter a city name.';
        return;
      }
      try {
        const apiKey = '6ca42f7e3e5d054c94ff9c95ede20a55'; // Replace with your OpenWeatherMap API key
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=${apiKey}`
        );
        const data = await response.json();
        if (data.cod === 200) {
          this.weather.push(data);
          this.error = '';
          this.city = ''; // Clear input after successful fetch
        } else {
          this.error = 'City not found.';
        }
      } catch (error) {
        this.error = 'Unable to fetch weather data.';
      }
    },
    formatTime(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleTimeString();
    },
    removeCity(index) {
      this.weather.splice(index, 1);
    },
    convertToFahrenheit(tempCelsius) {
      return (tempCelsius * 9) / 5 + 32;
    }
  },
  computed: {
    weatherBackground() {
      if (!this.weather.length) return '';
      const condition = this.weather[0].weather[0].main.toLowerCase();
      switch (condition) {
        case 'clear':
          return 'sunny';
        case 'rain':
        case 'drizzle':
          return 'rainy';
        case 'clouds':
          return 'cloudy';
        case 'snow':
          return 'snowy';
        default:
          return '';
      }
    }
  }
};
</script>

<style scoped>
/* Background for the landing page (before any city is entered) */
.landing-page {
  background: linear-gradient(120deg, #045c6a 0%, #cfcfcf 100%);
  min-height: 100vh;
}

/* Sunny weather background */
.sunny {
  background: linear-gradient(120deg, #fdb707 0%, #dec070 100%);
}

/* Rainy weather background */
.rainy {
  background: linear-gradient(120deg, #0f5caa 0%, #8eb4f0 100%);
}

/* Cloudy weather background */
.cloudy {
  background: linear-gradient(120deg, #4c4c4d 0%, #cfdef3 100%);
}

/* Snowy weather background with snow effect */
.snowy {
  background: linear-gradient(120deg, #8bd4f7 0%, #c9eafc 100%);
  position: relative;
}

.snow {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
  z-index: 10;
}

.snow:before, .snow:after {
  content: '';
  position: absolute;
  top: 0;
  left: 50%;
  width: 100vw;
  height: 100vh;
  background-image: url('https://img.freepik.com/free-vector/hand-drawn-flat-winter-landscape-illustration_23-2149149134.jpg?t=st=1728274883~exp=1728278483~hmac=0480459c5af46bdfd3caf1c148af3e8985a0b7d9d59718f5e9b213785e2d7b78&w=1380');
  opacity: 0.3;
  z-index: 11;
  animation: fall 10s infinite;
}

@keyframes fall {
  0% {
    transform: translateY(-10vh);
  }
  100% {
    transform: translateY(110vh);
  }
}

body {
  font-family: 'Roboto', sans-serif;
}
</style>
