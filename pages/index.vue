<script setup lang="ts">
import { ref, onMounted } from "vue";
import { watchOnce } from "@vueuse/core"; // Assuming @vueuse/core is used for lifecycle hooks
import {
  Carousel,
  type CarouselApi,
  CarouselContent,
  CarouselItem,
  CarouselNext,
  CarouselPrevious,
} from "@/components/ui/carousel";
import { Card, CardContent } from "@/components/ui/card";
// Import ScrollArea, ScrollBar if necessary

const emblaMainApi = ref<CarouselApi>();
const emblaThumbnailApi = ref<CarouselApi>();
const selectedIndex = ref(0);
const scrollbarWidth = ref("50%"); // Initialize with 50%
const width = 0;

function generateDates() {
  const dates = [];
  const today = new Date();
  const endDate = new Date(today.getFullYear(), 11, 31); // December 31 of this year

  for (let date = today; date <= endDate; date.setDate(date.getDate() + 1)) {
    dates.push(new Date(date));
  }

  return dates;
}

const dates = generateDates();

function onSelect() {
  if (!emblaMainApi.value || !emblaThumbnailApi.value) return;
  selectedIndex.value = emblaMainApi.value.selectedScrollSnap();
  emblaThumbnailApi.value.scrollTo(emblaMainApi.value.selectedScrollSnap());
}

function onThumbClick(index: number) {
  if (!emblaMainApi.value || !emblaThumbnailApi.value) return;
  emblaMainApi.value.scrollTo(index);
}

let divElement: any;
onMounted(() => {
  const scrollContainers = document.getElementsByClassName("overflow-x-scroll");
  divElement = scrollContainers[0];
  divElement.addEventListener("scroll", updateScrollbarThumbWidth);
});

function updateScrollbarThumbWidth() {
  const scrollLeft = divElement.scrollLeft;
  const scrollWidth = divElement.scrollWidth - divElement.clientWidth;
  const newWidthPercentage = ((scrollWidth - scrollLeft) / scrollWidth) * 50; // Calculate percentage

  // Convert the calculated percentage to a string and append '%'
  scrollbarWidth.value = `${newWidthPercentage}%`;
}

watchOnce(emblaMainApi, (emblaMainApi) => {
  if (!emblaMainApi) return;
  onSelect();
  emblaMainApi.on("select", onSelect);
  emblaMainApi.on("reInit", onSelect);
});
</script>

<template>
  <div class="w-full sm:w-auto">
    <Carousel
      class="relative w-full max-w-2xl mb-10 h-20"
      @init-api="(val) => (emblaMainApi = val)"
    >
      <div
        class="horizon_scrollbar"
        id="scroll-container"
        :style="{ '--scrollbar-thumb-width': scrollbarWidth }"
      >
        <CarouselContent class="flex gap-1 ml-0">
          <CarouselItem
            v-for="(date, index) in dates"
            :key="index"
            class="pl-0 basis-[1/10] cursor-pointer"
            @click="onThumbClick(index)"
          >
            <div
              class="p-1"
              :class="index === selectedIndex ? '' : 'opacity-50'"
            >
              <div
                class="flex items-center justify-center px-2 py-4 bg-[#8A8A8A] rounded-md border border-gray-900 mb-5"
              >
                <span class="text-base font-semibold">{{
                  date.toLocaleDateString("en-US", {
                    month: "short",
                    day: "numeric",
                  })
                }}</span>
              </div>
            </div>
          </CarouselItem>
        </CarouselContent>
      </div>
      <CarouselPrevious />
      <CarouselNext />
    </Carousel>
    <Carousel
      class="relative w-full max-w-2xl"
      @init-api="(val) => (emblaThumbnailApi = val)"
    >
      <CarouselContent>
        <CarouselItem v-for="(date, index) in dates" :key="index">
          <div class="p-1">
            <Card>
              <CardContent
                class="flex aspect-square items-center justify-center p-6 bg-[#8A8A8A]"
              >
                <span class="text-4xl font-semibold">{{
                  date.toLocaleDateString("en-US", {
                    month: "long",
                    day: "numeric",
                  })
                }}</span>
              </CardContent>
            </Card>
          </div>
        </CarouselItem>
      </CarouselContent>
    </Carousel>
  </div>
</template>
<style scoped>
.horizon_scrollbar {
  ::-webkit-scrollbar {
    height: 8px;
  }
  ::-webkit-scrollbar-track {
    background: #f1f1f1;
    border-radius: 0.25rem;
  }
  ::-webkit-scrollbar-thumb {
    background: #000;
    border-radius: 0.25rem;
    width: var(--scrollbar-thumb-width);
  }
  ::-webkit-scrollbar-thumb:hover {
    background: #555;
  }
}
</style>
