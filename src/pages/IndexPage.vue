<template>
  <q-page padding>
    <div class="row justify-center q-gutter-sm">
      <q-input
        borderless
        dense
        debounce="300"
        label="Enter Your Location Name"
        outlined
        rounded
        color="primary"
        v-model="city_name"
        :style="$q.platform.is.desktop ? 'width: 500px;' : 'width: 200px'"
      />
      <q-btn rounded icon="search" color="primary" @click="fetchCityData()">
        <q-tooltip>search</q-tooltip>
      </q-btn>
    </div>
    <div class="row justify-center q-mt-lg text-weight-bold">
      {{ city }},{{ country }}
    </div>

    <div class="row justify-center q-mt-xl">
      <q-card
        class="row justify-between"
        :style="$q.platform.is.desktop ? 'width: 800px;' : 'width: 300px'"
      >
        <q-card-section
          :style="$q.platform.is.desktop ? 'width: 400px;' : 'width: 300px'"
        >
          <p class="text-capitalize text-weight-bold text-subtitle1">
            current weather
          </p>
          <span class="row justify-between">
            <div>
              <img :src="weatherIconUrl" alt="Weather Icon" />
            </div>
            <div class="text-h2">{{ temperatureCelsius }}°C</div>
            <div>{{ temperatureFahrenheit }}°F</div>
          </span>
          <p>{{ desc }}</p>
        </q-card-section>
        <q-card-section
          class="q-mt-md"
          :style="$q.platform.is.desktop ? 'width: 250px;' : 'width: 300px'"
        >
          <div class="row justify-between">
            <p><q-icon name="mdi-air-filter" class="q-ml-xs" />Air Quality</p>
            <p>{{ airQualityDescription }}</p>
          </div>
          <q-separator />
          <div class="row justify-between">
            <p><q-icon name="mdi-water-percent" class="q-ml-xs" />Humidity</p>
            <p>{{ humidity }}%</p>
          </div>
          <q-separator />
          <div class="row justify-between">
            <p><q-icon name="mdi-weather-windy" class="q-ml-xs" />Wind</p>
            <p>{{ wind }}m/s</p>
          </div>
          <q-separator />
        </q-card-section>
      </q-card>
      <q-card
        flat
        class="q-mt-md"
        :style="$q.platform.is.desktop ? 'width: 1000px;' : 'width: 300px'"
      >
        <q-card-section>
          <p
            class="text-capitalize text-weight-bold text-center text-subtitle1"
          >
            5 days forecast
          </p>
        </q-card-section>
        <q-card class="q-pa-sm" flat>
          <q-table
            :rows="rows"
            :columns="columns"
            row-key="name"
            dense
            flat
            :hide-pagination="true"
          >
            <template #body-cell-icon="props">
              <q-td>
                <img
                  :src="forecastWeatherIconUrl(props.row.weather.icon)"
                  alt="Weather Icon"
                  style="width: 50px"
                />
              </q-td>
            </template>
          </q-table>
          <div class="q-mt-lg">
            <apexchart
              type="line"
              height="350"
              :options="chartOptions"
              :series="series"
            />
          </div>
        </q-card>
      </q-card>
    </div>
  </q-page>
</template>

<script setup>
import { computed, onMounted, reactive, ref } from "vue";
import { api } from "src/boot/axios";
import { useQuasar } from "quasar";
const $q = useQuasar();
const city = ref("");
const city_name = ref("");
const country = ref("");
const long = ref("");
const lat = ref("");
const desc = ref("");
const icon = ref(""); // Replace with your actual dynamic value
const airQuality = ref("");
const humidity = ref("");
const wind = ref("");
const time = ref([]);
const series = ref([
  {
    name: "temp",
    data: [],
  },
]);

const chartOptions = computed(() => {
  return {
    chart: {
      height: 350,
      type: "line",
      zoom: {
        enabled: true,
      },
      dropShadow: {
        enabled: true,
        color: "#000",
        top: 18,
        left: 7,
        blur: 10,
        opacity: 0.2,
      },
      toolbar: {
        show: false,
      },
    },
    dataLabels: {
      enabled: true,
    },
    stroke: {
      curve: "straight",
    },
    title: {
      text: "Hourly Temperature Forecast",
      align: "center",
    },
    grid: {
      borderColor: "#e7e7e7",
      row: {
        colors: ["#f3f3f3", "transparent"], // takes an array which will be repeated on columns
        opacity: 0.5,
      },
    },
    markers: {
      size: 1,
    },
    xaxis: {
      categories: time.value,
      title: {
        text: "Hours",
      },
    },
    yaxis: {
      title: {
        text: "Temperature",
      },
      min: 5,
      max: 40,
    },
    legend: {
      position: "top",
      horizontalAlign: "right",
      floating: true,
      offsetY: -25,
      offsetX: -5,
    },
  };
});

const weatherIconUrl = computed(() => {
  const modifiedIcon = ref(icon.value.substring(1));
  // Construct the complete URL by concatenating the dynamic part
  return `https://openweathermap.org/img/wn/${modifiedIcon.value}@2x.png`;
});
const forecastWeatherIconUrl = computed(() => (icon) => {
  const modifiedIcon = icon.substring(1);
  // Construct the complete URL by concatenating the dynamic part
  return `https://openweathermap.org/img/wn/${modifiedIcon}@2x.png`;
});
const columns = [
  {
    name: "date",
    label: "Date",
    align: "left",
    field: (row) => row.valid_date,
  },
  {
    name: "icon",
    label: "",
    align: "left",
    field: (row) => row.weather.icon,
  },
  {
    name: "temp",
    label: "Temp°C",
    align: "left",
    field: (row) => row.temp,
  },
  {
    name: "desc",
    label: "Description",
    align: "left",
    field: (row) => row.weather.description,
  },
  {
    name: "high-temp",
    label: "High-temp°C",
    align: "left",
    field: (row) => row.high_temp,
  },
  {
    name: "low-temp",
    label: "Low-temp°C",
    align: "left",
    field: (row) => row.low_temp,
  },
];
const rows = ref([]);
const temperatureCelsius = ref(null); // Replace with your actual temperature value in Celsius

const temperatureFahrenheit = computed(() => {
  // Convert Celsius to Fahrenheit and limit to 2 decimal places
  const fahrenheitValue = (temperatureCelsius.value * 9) / 5 + 32;
  return fahrenheitValue.toFixed(2); // Limit to 2 decimal places
});

const getAirQualityDescription = (airQuality) => {
  if (airQuality >= 0 && airQuality <= 50) {
    return "Good";
  } else if (airQuality >= 51 && airQuality <= 100) {
    return "Moderate";
  } else if (airQuality >= 101 && airQuality <= 150) {
    return "Unhealthy for Sensitive Groups";
  } else if (airQuality >= 151 && airQuality <= 200) {
    return "Unhealthy";
  } else if (airQuality >= 201 && airQuality <= 300) {
    return "Very Unhealthy";
  } else if (airQuality >= 301 && airQuality <= 500) {
    return "Hazardous";
  } else {
    return "Invalid AQI Value";
  }
};

const airQualityDescription = getAirQualityDescription(airQuality.value);
const geoInfo = async () => {
  const API_KEY = "4ad4b9f166584859aa3e22e47a7af221";
  const API_URL =
    "https://ipgeolocation.abstractapi.com/v1/?api_key=" + API_KEY;

  try {
    const response = await api.get(API_URL);
    const data = response.data;

    city.value = data.city;
    country.value = data.country;
    long.value = data.longitude;
    lat.value = data.latitude;
    fetchWeatherData();
  } catch (error) {
    $q.notify({
      message: "Network Error, try again later",
      color: "red",
      icon: "warning",
      position: "top-right",
    });
    console.error("Error:", error);
  }
};

const fetchCityData = async () => {
  const name = city_name.value;
  const apiKey = "g5Lb5BeE6K3EpAbk/mY4sA==IbKCsBJR0rD8Th4T"; // Replace with your API key

  try {
    const response = await api.get("https://api.api-ninjas.com/v1/city", {
      params: {
        name,
      },
      headers: {
        "X-Api-Key": apiKey,
      },
    });
    if (response.data.length === 0) {
      // Handle invalid city name
      $q.notify({
        message: "Invalid city name, please check and try again.",
        color: "red",
        icon: "warning",
        position: "top-right",
      });
    } else {
      // Valid city name, update values and fetch weather data
      city.value = response.data[0].name;
      long.value = response.data[0].longitude;
      lat.value = response.data[0].latitude;
      country.value = response.data[0].country;
      fetchWeatherData();
    }
  } catch (error) {
    // Handle invalid city name
    $q.notify({
      message: "Invalid city name, please check and try again.",
      color: "red",
      icon: "warning",
      position: "top-right",
    });
  }
};
const fetchWeatherData = async () => {
  const apiKey = "3cfe20f5d0cc4fef8433f49da69bf86f";
  try {
    const response = await api.get("https://api.weatherbit.io/v2.0/current", {
      params: {
        lat: lat.value,
        lon: long.value,
        key: apiKey,
      },
    });
    const forecastdaily = await api.get(
      "https://api.weatherbit.io/v2.0/forecast/daily",
      {
        params: {
          lat: lat.value,
          lon: long.value,
          key: apiKey,
        },
      }
    );
    series.value[0].data = [];
    time.value = [];
    const forecasthourly = await api.get(
      "https://api.weatherbit.io/v2.0/forecast/hourly",
      {
        params: {
          lat: lat.value,
          lon: long.value,
          key: apiKey,
        },
      }
    );

    temperatureCelsius.value = response.data.data[0].temp;
    humidity.value = response.data.data[0].rh;
    airQuality.value = response.data.data[0].aqi;
    wind.value =
      response.data.data[0].wind_cdir + response.data.data[0].wind_spd;
    icon.value = response.data.data[0].weather.icon;
    desc.value = response.data.data[0].weather.description;
    rows.value = forecastdaily.data.data.slice(0, 5);

    const today = new Date();
    const todayDate = today.toISOString().split("T")[0]; // Get today's date in yyyy-mm-dd format

    const filteredTemp = forecasthourly.data.data.filter((item) => {
      const itemDate = item.timestamp_local.split("T")[0]; // Extract date part from timestamp_local
      return itemDate === todayDate; // Compare with today's date
    });
    const chartDatas = filteredTemp.map((item) => item.temp);
    const chartLabels = filteredTemp.map(
      (item) => item.timestamp_local.split("T")[1]
    );
    series.value[0].data.push(...chartDatas);
    // chartOptions.value.xaxis.categories.push(...chartLabels);
    time.value = chartLabels;

    console.log(time.value);
  } catch (error) {
    console.error("Error:", error);
  }
};

onMounted(() => {
  geoInfo();
});
</script>
