<template>
  <q-layout view="hHh LpR fFf">
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
import { useRoute } from "vue-router";
import { api } from "src/boot/axios";
import { useRouter } from "vue-router";

// import { useAuthStore } from "stores/auth";
const route = useRoute();
const dark_mode = ref(false);
const $q = useQuasar();


// const user = JSON.parse(userString);
// const role = sessionStorage.getItem("role");
const leftDrawerOpen = ref(false);
const toggleLeftDrawer = () => (leftDrawerOpen.value = !leftDrawerOpen.value);
watch(dark_mode, (value) => {
  $q.dark.set(value);
});

const updatePassword = async () => {
  try {
    const response = await api.post("/updatePassword", {
      user_id: user_id,
      old_password: old_password.value,
      new_password: new_password.value,
    });
    // console.log(response.data.data);
    $q.notify({
      message: response.data.data,
      color: "green",
      icon: "check",
      position: "top-right",
    });
  } catch (error) {
    console.log(error);
    $q.notify({
      message: "old password is invalid",
      color: "red",
      icon: "error",
      position: "top-right",
    });
  }
};
const logout = async () => {
  api.post("/logout");
  sessionStorage.removeItem("token");
  sessionStorage.removeItem("job_title");
  sessionStorage.removeItem("auth");
  sessionStorage.removeItem("user_id");
  sessionStorage.removeItem("access_privileges");
  router.push({ name: "Login" });
};
// Function to format the breadcrumb label
const getBreadcrumbLabel = (path) => {
  const parts = path.split("/").filter((part) => part !== ""); // Split and remove empty parts
  return `Home > ${parts.join(" > ")}`; // Join parts with ' > '
};
var currentYear = ref("");

onMounted(() => {
  currentYear.value = new Date().getFullYear();
});
</script>
