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
    <div>{{ city }},{{ country }},{{ long }},{{ lat }}</div>
  </q-page>
</template>

<script setup>
import { onMounted, ref } from "vue";
import { api } from "src/boot/axios";
import { useQuasar } from "quasar";
const $q = useQuasar();
const city = ref("");
const city_name = ref("");
const country = ref("");
const long = ref("");
const lat = ref("");
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
    console.error("Error:", error);
  }
};
const weatherData = ref(null);
const fetchWeatherData = async () => {
  const apiKey = "3cfe20f5d0cc4fef8433f49da69bf86f";
  try {
    const response = await api.get("https://api.weatherbit.io/v2.0/current", {
      params: {
        lat: lat.value,
        lon: long.value,
        key: apiKey,
        include: "minutely",
      },
    });

    weatherData.value = response.data;
  } catch (error) {
    console.error("Error:", error);
  }
};

onMounted(() => {
  geoInfo();
});
</script>
