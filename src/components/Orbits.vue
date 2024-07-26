<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import axios from 'axios';
import Orb from './Orb.vue'; // Adjust the path as needed

const circles = ref([]);
const scrollY = ref(0);

const staticCircleData = [
  { size: 1300, top: '10%' },
  { size: 1200, top: '20%' },
  { size: 1100, top: '30%' },
  { size: 1000, top: '40%' },
  { size: 900, top: '50%' },
  { size: 800, top: '60%' },
  { size: 700, top: '70%' },
  { size: 600, top: '80%' },
  { size: 500, top: '90%' }
];

const fetchCircleData = async () => {
  try {
    const response = await axios.get(`https://xsrr-l2ye-dpbj.f2.xano.io/api:oUvfVMO5/receive_week?start_date=18`); // Replace with your API endpoint
    const dynamicCircleData = response.data; // Assuming the response data is an array of objects
    console.log(dynamicCircleData)

    // Merge static and dynamic data
    circles.value = staticCircleData.map((staticData, index) => ({
      ...staticData,
      ...dynamicCircleData[index], // Merge with corresponding dynamic data
      originalSize: staticData.size,
      showSpan: false
    }));
  } catch (error) {
    console.error('Error fetching circle data:', error);
  }
};

const updateCircleSizes = () => {
  const scrollTop = scrollY.value;
  const viewportHeight = window.innerHeight;
  const documentHeight = document.documentElement.scrollHeight;
  const scrollMax = documentHeight - viewportHeight;
  const scrollPercentage = Math.min(scrollTop / scrollMax, 1);

  circles.value.forEach(circle => {
    const sizeAdjustment = scrollPercentage * 300;
    circle.size = Math.max(circle.originalSize + sizeAdjustment, circle.originalSize);

    const circleTopInPixels = parseFloat(circle.top) / 100 * viewportHeight;
    circle.showSpan = Math.abs(circleTopInPixels - scrollTop) <= viewportHeight * 0.1;
  });
};

const onScroll = () => {
  scrollY.value = window.pageYOffset || document.documentElement.scrollTop;
  updateCircleSizes();
};

const handleImageClick = (image) => {
  console.log('Image clicked:', image);
};

const disableArrowKeysAndPageUpDown = (event) => {
  const keys = ["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight", "PageUp", "PageDown"];
  if (keys.includes(event.key)) {
    event.preventDefault();
  }
};

onMounted(async () => {
  await fetchCircleData();
  window.addEventListener('scroll', onScroll);
  window.addEventListener('keydown', disableArrowKeysAndPageUpDown);
  updateCircleSizes();
});

onUnmounted(() => {
  window.removeEventListener('scroll', onScroll);
  window.removeEventListener('keydown', disableArrowKeysAndPageUpDown);
});
</script>

<template>
  <div class="relative w-full min-h-[180vh] pt-3 no-scrollbar scroll-smooth">
    <Orb
        v-for="(circle, index) in circles"
        :key="index"
        :circle="circle"
        :index="index"
        :onImageClick="handleImageClick"
    />
  </div>
</template>

<style>
/* Basic styling for visibility */
</style>
