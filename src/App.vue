<template>
  <v-app :class="weatherBackground || 'landing-page'">
    <v-container fluid class="fill-height">
      <v-row justify="center" align="center">
        <v-col cols="12" sm="10" md="8" lg="6">
          <v-card elevation="10" shaped class="weather-card">
            <v-card-title class="text-h4 font-weight-bold text-center py-4">
              Weather Forecast
            </v-card-title>
            <v-card-text>
              <v-form @submit.prevent="fetchWeather">
                <v-row align="center" no-gutters>
                  <v-col cols="12" sm="8">
                    <v-text-field
                      v-model="city"
                      label="Enter city"
                      prepend-inner-icon="mdi-map-marker"
                      outlined
                      dense
                      clearable
                      @keyup.enter="fetchWeather"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="4" class="pl-sm-2">
                    <v-btn
                      color="primary"
                      block
                      @click="fetchWeather"
                      :loading="loading"
                      :disabled="loading"
                    >
                      <v-icon left>mdi-cloud-search</v-icon>
                      Get Weather
                    </v-btn>
                  </v-col>
                </v-row>
              </v-form>

              <v-expand-transition>
                <v-alert v-if="error" type="error" dense text>{{ error }}</v-alert>
              </v-expand-transition>

              <v-slide-y-transition group>
                <v-card
                  v-for="(cityWeather, index) in weather"
                  :key="index"
                  class="mt-4"
                  elevation="4"
                >
                  <v-card-title class="text-h5 d-flex justify-space-between">
                    {{ cityWeather.name }}
                    <v-chip :color="getWeatherColor(cityWeather.weather[0].main)" dark>
                      {{ cityWeather.weather[0].main }}
                    </v-chip>
                  </v-card-title>
                  <v-card-text>
                    <v-row align="center">
                      <v-col cols="6">
                        <div class="text-h3 font-weight-bold">
                          {{ Math.round(cityWeather.main.temp) }}°C
                        </div>
                        <div class="text-subtitle-1">
                          Feels like {{ Math.round(cityWeather.main.feels_like) }}°C
                        </div>
                      </v-col>
                      <v-col cols="6" class="text-right">
                        <v-icon size="64" :color="getWeatherColor(cityWeather.weather[0].main)">
                          {{ getWeatherIcon(cityWeather.weather[0].main) }}
                        </v-icon>
                      </v-col>
                    </v-row>
                    <v-divider class="my-3"></v-divider>
                    <v-row>
                      <v-col cols="6">
                        <v-list-item>
                          <v-list-item-icon>
                            <v-icon>mdi-water-percent</v-icon>
                          </v-list-item-icon>
                          <v-list-item-content>
                            <v-list-item-title>Humidity</v-list-item-title>
                            <v-list-item-subtitle>{{ cityWeather.main.humidity }}%</v-list-item-subtitle>
                          </v-list-item-content>
                        </v-list-item>
                      </v-col>
                      <v-col cols="6">
                        <v-list-item>
                          <v-list-item-icon>
                            <v-icon>mdi-weather-windy</v-icon>
                          </v-list-item-icon>
                          <v-list-item-content>
                            <v-list-item-title>Wind Speed</v-list-item-title>
                            <v-list-item-subtitle>{{ cityWeather.wind.speed }} m/s</v-list-item-subtitle>
                          </v-list-item-content>
                        </v-list-item>
                      </v-col>
                    </v-row>
                    <v-row>
                      <v-col cols="6">
                        <v-list-item>
                          <v-list-item-icon>
                            <v-icon>mdi-gauge</v-icon>
                          </v-list-item-icon>
                          <v-list-item-content>
                            <v-list-item-title>Pressure</v-list-item-title>
                            <v-list-item-subtitle>{{ cityWeather.main.pressure }} hPa</v-list-item-subtitle>
                          </v-list-item-content>
                        </v-list-item>
                      </v-col>
                      <v-col cols="6">
                        <v-list-item>
                          <v-list-item-icon>
                            <v-icon>mdi-eye</v-icon>
                          </v-list-item-icon>
                          <v-list-item-content>
                            <v-list-item-title>Visibility</v-list-item-title>
                            <v-list-item-subtitle>{{ cityWeather.visibility / 1000 }} km</v-list-item-subtitle>
                          </v-list-item-content>
                        </v-list-item>
                      </v-col>
                    </v-row>
                    <v-row>
                      <v-col cols="6">
                        <v-list-item>
                          <v-list-item-icon>
                            <v-icon>mdi-weather-sunset-up</v-icon>
                          </v-list-item-icon>
                          <v-list-item-content>
                            <v-list-item-title>Sunrise</v-list-item-title>
                            <v-list-item-subtitle>{{ formatTime(cityWeather.sys.sunrise) }}</v-list-item-subtitle>
                          </v-list-item-content>
                        </v-list-item>
                      </v-col>
                      <v-col cols="6">
                        <v-list-item>
                          <v-list-item-icon>
                            <v-icon>mdi-weather-sunset-down</v-icon>
                          </v-list-item-icon>
                          <v-list-item-content>
                            <v-list-item-title>Sunset</v-list-item-title>
                            <v-list-item-subtitle>{{ formatTime(cityWeather.sys.sunset) }}</v-list-item-subtitle>
                          </v-list-item-content>
                        </v-list-item>
                      </v-col>
                    </v-row>
                  </v-card-text>
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="error" text @click="removeCity(index)">
                      <v-icon left>mdi-delete</v-icon>
                      Remove
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-slide-y-transition>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
    <div v-if="weatherBackground === 'snowy'" class="snow"></div>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      city: '',
      weather: [],
      error: '',
      loading: false,
    };
  },
  methods: {
    async fetchWeather() {
      if (this.city.trim() === '') {
        this.error = 'Please enter a city name.';
        return;
      }
      this.loading = true;
      try {
        const apiKey = '6ca42f7e3e5d054c94ff9c95ede20a55'; // Replace with your OpenWeatherMap API key
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=${apiKey}`
        );
        const data = await response.json();
        if (data.cod === 200) {
          this.weather.unshift(data);
          this.error = '';
          this.city = ''; // Clear input after successful fetch
        } else {
          this.error = 'City not found.';
        }
      } catch (error) {
        this.error = 'Unable to fetch weather data.';
      } finally {
        this.loading = false;
      }
    },
    formatTime(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    },
    removeCity(index) {
      this.weather.splice(index, 1);
    },
    getWeatherIcon(condition) {
      switch (condition.toLowerCase()) {
        case 'clear':
          return 'mdi-weather-sunny';
        case 'rain':
        case 'drizzle':
          return 'mdi-weather-rainy';
        case 'clouds':
          return 'mdi-weather-cloudy';
        case 'snow':
          return 'mdi-weather-snowy';
        case 'thunderstorm':
          return 'mdi-weather-lightning';
        default:
          return 'mdi-weather-cloudy';
      }
    },
    getWeatherColor(condition) {
      switch (condition.toLowerCase()) {
        case 'clear':
          return 'amber';
        case 'rain':
        case 'drizzle':
          return 'blue';
        case 'clouds':
          return 'grey';
        case 'snow':
          return 'light-blue';
        case 'thunderstorm':
          return 'deep-purple';
        default:
          return 'blue-grey';
      }
    },
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
.weather-card {
  backdrop-filter: blur(10px);
  background-color: rgba(255, 255, 255, 0.7) !important;
}

.landing-page {
  background: linear-gradient(120deg, #045c6a 0%, #cfcfcf 100%);
}

.sunny {
  background: linear-gradient(120deg, #fdb707 0%, #dec070 100%);
}

.rainy {
  background: linear-gradient(120deg, #0f5caa 0%, #8eb4f0 100%);
}

.cloudy {
  background: linear-gradient(120deg, #4c4c4d 0%, #cfdef3 100%);
}

.snowy {
  background: linear-gradient(120deg, #8bd4f7 0%, #c9eafc 100%);
  position: relative;
}

.snow {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
  z-index: 1000;
}

.snow:before, .snow:after {
  content: '';
  position: fixed;
  top: -50%;
  left: 0;
  right: 0;
  bottom: 0;
  background-image: url('data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1000 1000" preserveAspectRatio="none"%3E%3Cpath d="M1000 0L0 1000h1000V0z" fill="%23ffffff33"/%3E%3C/svg%3E'), url('data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1000 1000" preserveAspectRatio="none"%3E%3Cpath d="M0 0l1000 1000H0V0z" fill="%23ffffff33"/%3E%3C/svg%3E');
  background-size: 100px 100px;
  animation: snow 20s linear infinite;
  opacity: 0.2;
}

.snow:after {
  top: 0;
  animation: snow 15s linear infinite;
  animation-delay: -7.5s;
}

@keyframes snow {
  0% {
    transform: translateY(0);
  }
  100% {
    transform: translateY(100%);
  }
}
</style>