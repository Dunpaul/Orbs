<script setup>
import { ref, computed, nextTick } from 'vue';
import 'bootstrap-icons/font/bootstrap-icons.css';

const props = defineProps({
  circle: Object,
  index: Number,
  onImageClick: Function
});

// console.log(props.circle)

const options = {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric',
  hour: '2-digit',
  minute: '2-digit',
  timeZone: 'America/New_York',
  timeZoneName: 'short'
};

const hoveredImage = ref(null);
const hoveredPosition = ref({ top: '0px', left: '0px' });
const isImageHovered = ref(false);
const isCardHovered = ref(false);
const isParagraphExpanded = ref(false); // Add reactive state for paragraph expansion

const handleMouseEnterImage = async (event, image) => {
  const imageElement = event.target;

  // Wait for the next DOM update to ensure styles are applied
  await nextTick();

  const imageRect = imageElement.getBoundingClientRect();
  const circleCenterX = imageElement.parentElement.getBoundingClientRect().left + imageElement.parentElement.offsetWidth / 2;
  const newTop = imageRect.top + window.scrollY; // Adjust for page scroll
  let newLeft = imageRect.left; // Adjust for page scroll

  // Determine if the image is on the right side of the arc
  if (imageRect.left < circleCenterX) {
    newLeft -= 700;
  } else {
    newLeft -= 1000;
  }

  hoveredImage.value = image;
  hoveredPosition.value = {
    top: `${newTop}px`,
    left: `${newLeft}px`
  };

  isImageHovered.value = true;
};

const handleMouseLeaveImage = () => {
  isImageHovered.value = false;
  if (!isCardHovered.value) {
    // Hide the card only if neither the image nor the card is hovered
    setTimeout(() => {
      if (!isImageHovered.value && !isCardHovered.value) {
        hoveredImage.value = null;
      }
    }, 100); // Delay to handle slight mouse movements
  }
};

const handleMouseEnterCard = () => {
  isCardHovered.value = true;
};

const handleMouseLeaveCard = () => {
  isCardHovered.value = false;
  // Hide the card if the mouse leaves both the image and the card
  setTimeout(() => {
    if (!isImageHovered.value && !isCardHovered.value) {
      hoveredImage.value = null;
    }
  }, 100); // Delay to handle slight mouse movements
};

const toggleParagraphExpansion = () => {
  isParagraphExpanded.value = !isParagraphExpanded.value;
};

const calculateImagePosition = (index, totalImages, circleSize, showSpan) => {
  const startAngle = 290; // starting angle in degrees
  const endAngle = 430; // ending angle in degrees
  const radius = circleSize / 2;
  const centerIndex = Math.floor((totalImages - 1) / 2);

  // Calculate the angle for each image
  let angle = startAngle + (index * (endAngle - startAngle) / (totalImages - 1));
  if (props.circle.showSpan) {
    if (totalImages % 2 !== 0 && index === centerIndex) {
      angle += 8; // Adjust the middle image when odd
    } else if (totalImages % 2 === 0 && (index === centerIndex || index === centerIndex + 1)) {
      angle += (index === centerIndex) ? -5 : 5; // Adjust the two middle images when even
    }
  }

  const radians = angle * (Math.PI / 180); // Convert to radians

  const top = radius - (radius * Math.cos(radians)) - 20; // Subtract 20 to center the image
  const left = radius + (radius * Math.sin(radians)) - 20; // Subtract 20 to center the image

  return {
    top: `${top}px`,
    left: `${left}px`,
    transform: showSpan && index === centerIndex ? 'translateX(10px)' : 'translateX(0px)',
    transition: 'transform 3s ease-in-out, left 5s ease-in-out, right 5s ease-in-out,' // Add a smooth transition for the slide animation
  };
};

const showDelayedContent = ref(false);

const onCardEnter = () => {
  // Delay the appearance of the content by 500ms
  setTimeout(() => {
    showDelayedContent.value = true;
  }, 100);
};
const onCardLeave = () => {
  // Reset the delayed content visibility when the card leaves
  showDelayedContent.value = false;
};

const fullText = props.circle.array[0]._orbits_last_message.message

const truncateLength = 58;

const truncatedText = computed(() => {
  if (fullText.length <= truncateLength) {
    return fullText;
  }
  return isParagraphExpanded.value ? fullText : `${fullText.substring(0, truncateLength)}...`;
});

const  timeSince = (date) => {
  const seconds = Math.floor((new Date() - new Date(date)) / 1000);

  let interval = Math.floor(seconds / 31536000);
  if (interval > 1) {
    return interval + " years ago";
  }
  interval = Math.floor(seconds / 2592000);
  if (interval > 1) {
    return interval + " months ago";
  }
  interval = Math.floor(seconds / 86400);
  if (interval > 1) {
    return interval + " days ago";
  }
  interval = Math.floor(seconds / 3600);
  if (interval > 1) {
    return interval + " hours ago";
  }
  interval = Math.floor(seconds / 60);
  if (interval > 1) {
    return interval + " minutes ago";
  }
  return Math.floor(seconds) + " seconds ago";
}

const formatDate = (dateString) => {
  const date = new Date(dateString);
  const options = { weekday: 'short', month: 'short', day: 'numeric' };
  return date.toLocaleDateString('en-US', options);
};

</script>


<template>
  <div
      class="absolute z-0 left-1/2 transform -translate-x-1/2 border border-b-0 border-[white] m-0 rounded-t-full flex flex-row justify-center content-start orbit"
      :style="{ width: `${props.circle.size}px`, height: `${props.circle.size / 2}px`, top: props.circle.top }"
  >
    <template v-for="(image, imgIndex) in props.circle.array" :key="`image-${imgIndex}`">
      <img
          v-if="imgIndex < Math.floor(props.circle.array.length / 2)"
          :src="image.img"
          :class="['thumbnail border-[0.5px] border-white cursor-pointer absolute rounded-full w-[40px] h-[40px]', { 'thumbnail-slide': props.circle.showSpan && props.circle.array.length % 2 !== 0 && imgIndex === Math.floor(props.circle.array.length / 2) }]"
          :style="calculateImagePosition(imgIndex, props.circle.array.length, props.circle.size)"
          alt="Thumbnail"
          @click="onImageClick(image)"
          @mouseenter="handleMouseEnterImage($event, image)"
          @mouseleave="handleMouseLeaveImage"
      />
      <transition name="fade">
        <span
            v-if="props.circle.showSpan && imgIndex === Math.floor(props.circle.array.length / 2)"
            class="p-1 bg-stone-950 text-stone-500 text-sm font-semibold z-20 relative top-[-2%]"
        >
          {{ formatDate(props.circle.contact_date) }}
        </span>
      </transition>
      <img
          v-if="imgIndex >= Math.floor(props.circle.array.length / 2)"
          :src="image.img"
          :class="['thumbnail cursor-pointer border-[0.5px] border-white absolute rounded-full w-[40px] h-[40px]', { 'thumbnail-slide': props.circle.showSpan && props.circle.array.length % 2 !== 0 && imgIndex === Math.floor(props.circle.array.length / 2) }]"
          :style="calculateImagePosition(imgIndex, props.circle.array.length, props.circle.size)"
          alt="Thumbnail"
          @click="onImageClick(image)"
          @mouseenter="handleMouseEnterImage($event, image)"
          @mouseleave="handleMouseLeaveImage"
      />
    </template>
  </div>
  <transition name="smooth-fade" @after-enter="onCardEnter" @before-leave="onCardLeave">
    <div
        v-if="hoveredImage"
        :style="hoveredPosition"
        class="hover-card absolute bg-stone-950 bg-opacity-50 text-white shadow-lg rounded-lg z-50 w-[400px] backdrop-blur-md backdrop-brightness-110"
        @mouseenter="handleMouseEnterCard"
        @mouseleave="handleMouseLeaveCard"
    >
      <div class="bg-stone-950 flex p-1 relative">
        <span class="flex justify-center items-center mx-2 w-[60px] h-[60px] rounded-full gradient-border">
          <img :src="hoveredImage.img" alt="Hovered Image" class="rounded-full z-10 w-[60px] h-[60px]" />
        </span>
        <div>
          <h6 class="font-bold text-lg">{{ hoveredImage.name }}</h6>
          <p class="font-normal text-sm">{{ hoveredImage.position }}</p>
          <p class="text-xs">{{ hoveredImage.city }}</p>
          <div class="flex">
            <div class="flex me-2 pt-1">
              <img
                  :src="hoveredImage.img"
                  alt="Micro images"
                  class="w-5 h-5 rounded-full border-[1px] border-stone-400"
              >
              <img
                  :src="hoveredImage.img"
                  alt="Micro images"
                  class="w-5 h-5 rounded-full border-[1px] border-stone-400 ms-[-3px]"
              >
              <img
                  :src="hoveredImage.img"
                  alt="Micro images"
                  class="w-5 h-5 rounded-full border-[1px] border-stone-400 ms-[-3px]"
              >
              <img
                  :src="hoveredImage.img"
                  alt="Micro images"
                  class="w-5 h-5 rounded-full border-[1px] border-stone-400 ms-[-3px]"
              >
            </div>
            <div>
              <p class="text-xs">
                Peter Swage, John Eremic, <br /> and 281 other mutual connections
              </p>
            </div>
          </div>
        </div>
      </div>
      <transition name="fade">
        <div v-show="showDelayedContent" class="p-2">
          <h6 class="bg-stone-900 font-light py-1 px-2 mb-1">
            <i class="bi bi-envelope font-normal text-xl py-1"></i>
            <span class="px-2">Reply from Emery Wells</span>
          </h6>
          <div class="flex justify-between">
            <p class="text-xs text-stone-400">{{ new Date(hoveredImage.created_at).toLocaleDateString('en-US', options) }}</p>
            <p class="text-xs text-stone-400">{{ timeSince(hoveredImage.created_at) }}</p>
          </div>
          <div>
            <h6 class="font-bold text-sm py-1">
              {{ hoveredImage._orbits_last_message.message_head }}
            </h6>
            <p class="font-normal text-sm">
              {{ truncatedText }}
            </p>
            <span class="font-normal text-xs cursor-pointer" @click="toggleParagraphExpansion">
              {{ isParagraphExpanded ? 'Less' : 'More' }}
              <i :class="isParagraphExpanded ? 'bi bi-chevron-up' : 'bi bi-chevron-down'"></i>
            </span>
          </div>
        </div>
      </transition>
    </div>
  </transition>
</template>

<style scoped>
.hover-card {
  transition: opacity 0.5s, transform 0.5s;
  opacity: 1;
}

.smooth-fade-enter-active,
.smooth-fade-leave-active {
  transition: opacity 0.5s ease-in-out, transform 0.5s ease-in-out;
}

.smooth-fade-enter-from,
.smooth-fade-leave-to {
  opacity: 0;
  transform: translateY(-10px); /* Optional: for a slight slide effect */
}
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease-in-out, transform 0.5s ease-in-out;
}

.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0;
  transform: translateY(-10px);
}

.thumbnail {
  transition: transform 0.3s;
}

.thumbnail:hover {
  transform: scale(1.1);
}

.thumbnail::before{
  content: '';
  position: absolute;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background-image: linear-gradient(to bottom, white, black);
  z-index: 1;
}

.gradient-border::before {
  content: '';
  position: absolute;
  width: 62px;
  height: 62px;
  border-radius: 50%;
  background-image: linear-gradient(to bottom, white, black);
  z-index: 1;
}
</style>



