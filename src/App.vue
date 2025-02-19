<template>
  <div id="app">
    <ImageCarousel :images="images" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import ImageCarousel from "./components/ImageCarousel.vue";
import axios from "axios";

interface Image {
  id: string;
  download_url: string;
}

const images = ref<Image[]>([]);

const fetchImages = async () => {
  const response = await axios.get(
    "https://picsum.photos/v2/list?page=1&limit=20"
  );
  images.value = response.data;
};

onMounted(fetchImages);
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  text-align: center;
  margin-top: 20px;
  width: 100vw;
}
</style>
