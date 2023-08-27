<template>
  <q-layout view="hHh LpR fFf" class="bg-images">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-img
          alt="Ellipsis logo"
          src="~assets/elipsis.png"
          :style="$q.platform.is.desktop ? 'width: 90px;' : 'width: 70px'"
        />
        <q-toolbar-title>Weather-App </q-toolbar-title>
        <q-toggle
          v-model="dark_mode"
          checked-icon="mdi-weather-night"
          size="60px"
          color="grey"
          unchecked-icon="mdi-white-balance-sunny"
        />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view class="q-pa-lg" />

      <div class="bg-grey-1 q-py-sm text-black text-center">
        <p class="text-xs">
          &copy; {{ currentYear }}. Developed by IlungaTheOmniDev
        </p>
      </div>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { onMounted, ref, watch } from "vue";
import { useQuasar } from "quasar";

// import { useAuthStore } from "stores/auth";
const dark_mode = ref(false);
const $q = useQuasar();

watch(dark_mode, (value) => {
  $q.dark.set(value);
});

var currentYear = ref("");

onMounted(() => {
  currentYear.value = new Date().getFullYear();
});
</script>
<style>
.bg-images {
  position: relative;
}
.bg-images::before {
  content: "";
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url("../assets/weather1.jpeg");
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.2; /* Change the opacity value as desired */
}
</style>
