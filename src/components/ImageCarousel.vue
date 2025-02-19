<template>
  <div class="carousel-wrapper">
    <div class="carousel-container">
      <button @click="prevImage" class="nav-button left">⟨</button>

      <transition-group
        name="slide-horizontal"
        mode="out-in"
        tag="div"
        class="carousel"
      >
        <div
          v-for="image in visibleImages"
          :key="image.id"
          class="carousel-item"
          :class="{ expand: isExpanded(image.id) }"
          @mouseenter="hoveredImage = image.id"
          @mouseleave="hoveredImage = null"
        >
          <img
            :src="image.download_url"
            :alt="`Image ${image.id}`"
            @click="toggleSelectedImage(image)"
            :class="{ selected: selectedImages.includes(image) }"
            class="carousel-image"
          />
        </div>
      </transition-group>

      <button @click="nextImage" class="nav-button right">⟩</button>
    </div>

    <div class="selected-images">
      <h3>Selected Images:</h3>
      <transition-group name="fade" tag="ul" class="images-url_list">
        <li class="image-url" v-for="image in selectedImages" :key="image.id">
          {{ image.download_url }}
        </li>
      </transition-group>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";

interface Image {
  id: string;
  download_url: string;
}

const props = defineProps<{ images: Image[] }>();

const selectedImages = ref<Image[]>([]);
const hoveredImage = ref<string | null>(null);
const currentIndex = ref(0);
const imagesToShow = ref(1);

const updateImagesToShow = () => {
  const width = window.innerWidth;
  if (width >= 1200) {
    imagesToShow.value = 5;
  } else if (width >= 992) {
    imagesToShow.value = 4;
  } else if (width >= 768) {
    imagesToShow.value = 3;
  } else if (width >= 576) {
    imagesToShow.value = 2;
  } else {
    imagesToShow.value = 1;
  }
};

onMounted(() => {
  window.addEventListener("resize", updateImagesToShow);
  updateImagesToShow();
});

onUnmounted(() => {
  window.removeEventListener("resize", updateImagesToShow);
});

const visibleImages = computed(() => {
  const totalImages = props.images.length;
  const endIndex = currentIndex.value + imagesToShow.value;

  if (endIndex <= totalImages) {
    return props.images.slice(currentIndex.value, endIndex);
  }
  return [
    ...props.images.slice(currentIndex.value, totalImages),
    ...props.images.slice(0, endIndex % totalImages),
  ];
});

const nextImage = () => {
  preloadImages(
    (currentIndex.value + imagesToShow.value) % props.images.length
  );
  currentIndex.value = (currentIndex.value + 1) % props.images.length;
};

const prevImage = () => {
  preloadImages(
    (currentIndex.value - 1 + props.images.length) % props.images.length
  );
  currentIndex.value =
    (currentIndex.value - 1 + props.images.length) % props.images.length;
};

const toggleSelectedImage = (image: Image) => {
  const isSelected = selectedImages.value.some((img) => img.id === image.id);
  if (isSelected) {
    selectedImages.value = selectedImages.value.filter(
      (img) => img.id !== image.id
    );
  } else {
    selectedImages.value.push(image);
  }
};

const isExpanded = (id: string) => {
  return (
    selectedImages.value.some((img) => img.id === id) ||
    hoveredImage.value === id
  );
};

const preloadImages = (index: number) => {
  const img = new Image();
  img.src = props.images[index].download_url;
};
</script>

<style scoped lang="scss">
.carousel-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 400px;
}

.carousel-container {
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  margin: 0 auto;
  width: 100%;

  .nav-button {
    background: linear-gradient(135deg, #1f1f2e, #353569);
    color: #00ffff;
    border: none;
    padding: 20px;
    font-size: 36px;
    cursor: pointer;
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    transition: all 0.3s ease;
    box-shadow: 0px 4px 10px rgba(0, 255, 255, 0.5);

    &:hover {
      background: linear-gradient(135deg, #00ffff, #005f73);
      box-shadow: 0px 6px 15px rgba(0, 255, 255, 0.7);
    }

    &.left {
      left: -60px;
    }
    &.right {
      right: -60px;
    }
  }
}

.carousel {
  display: flex;
  width: 100%;
  justify-content: space-around;
  padding: 20px 0;
  position: relative;
  overflow: hidden;
}

.slide-horizontal-enter-active,
.slide-horizontal-leave-active {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  display: flex;
  justify-content: space-around;
  transition: transform 0.6s ease-in;
}

.slide-horizontal-enter-from {
  transform: translateX(100%);
}
.slide-horizontal-leave-to {
  transform: translateX(-100%);
}

.carousel-item {
  position: relative;
  margin: 5px;
  text-align: center;
  flex: 1;
  transition: all 0.3s ease;

  &.expand {
    flex: 2;
  }
}

.carousel-image {
  max-width: 300px;
  width: 100%;
  height: auto;
  border-radius: 5px;
  cursor: pointer;
  transition: transform 0.3s, box-shadow 0.3s;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  background: linear-gradient(145deg, #212330, #3a3f58);
  border: 2px solid #00ffff;

  &:hover {
    box-shadow: 0 4px 12px rgba(0, 255, 255, 0.5);
  }

  &.selected {
    box-shadow: 0 12px 30px rgba(0, 255, 255, 0.5);
  }
}

.selected-images {
  width: 80%;
  margin-top: 20px;
  text-align: center;
}

.images-url_list {
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0;
  overflow-anchor: none;
}

.image-url {
  list-style-type: none;
  font-size: 1em;
  color: #00ffff;
  background-color: rgba(0, 0, 0, 0.5);
  padding: 5px;
  border-radius: 5px;
  margin: 5px 0;
  display: inline-block;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.7s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

@media (max-width: 900px) {
  .carousel-container {
    max-width: 70vw;
    margin: 0 auto;

    .nav-button {
      font-size: 28px;
      padding: 15px;

      &.left {
        left: -10vw;
      }
      &.right {
        right: -10vw;
      }
    }
  }
}

@media (max-width: 576px) {
  .carousel-container {
    .nav-button {
      font-size: 24px;
      padding: 10px;
    }
  }
}
</style>
