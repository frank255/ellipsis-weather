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
        v-model="filter"
        :style="$q.platform.is.desktop ? 'width: 500px;' : 'width: 200px'"
      />
      <q-btn rounded icon="search" color="primary">
        <q-tooltip>search</q-tooltip>
      </q-btn>
    </div>
  </q-page>
</template>

<script setup>
import { defineComponent, onMounted } from "vue";
import { api } from "src/boot/axios";
const geoInfo = async () => {
  const API_KEY = "4ad4b9f166584859aa3e22e47a7af221";
  const API_URL =
    "https://ipgeolocation.abstractapi.com/v1/?api_key=" + API_KEY;
  const apiResponse = await fetch(API_URL);
  const data = await apiResponse.json();
  console.log(data);
};

onMounted(() => {
  geoInfo();
});
</script>
