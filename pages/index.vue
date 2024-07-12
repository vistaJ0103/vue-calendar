<script setup lang="ts">
import { ref } from "vue";
import { watchOnce } from "@vueuse/core";
import {
  Carousel,
  type CarouselApi,
  CarouselContent,
  CarouselItem,
  CarouselNext,
  CarouselPrevious,
} from "@/components/ui/carousel";
import { Card, CardContent } from "@/components/ui/card";
import { Progress } from "@/components/ui/progress";

const emblaMainApi = ref<CarouselApi>();
const emblaThumbnailApi = ref<CarouselApi>();
const selectedIndex = ref(0);

// Function to generate dates from today to December 31
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
  updateProgress();
}

function onThumbClick(index: number) {
  if (!emblaMainApi.value || !emblaThumbnailApi.value) return;
  emblaMainApi.value.scrollTo(index);
}

watchOnce(emblaMainApi, (emblaMainApi) => {
  if (!emblaMainApi) return;

  onSelect();
  emblaMainApi.on("select", onSelect);
  emblaMainApi.on("reInit", onSelect);
});
const progress = ref(0);
function updateProgress() {
  progress.value = ((selectedIndex.value + 1) / dates.length) * 100;
}
</script>

<template>
  <div class="w-full sm:w-auto">
    <Carousel
      class="relative w-full max-w-2xl"
      @init-api="(val) => (emblaMainApi = val)"
    >
      <CarouselContent class="flex gap-1 ml-0">
        <CarouselItem
          v-for="(date, index) in dates"
          :key="index"
          class="pl-0 basis-[1/10] cursor-pointer"
          @click="onThumbClick(index)"
        >
          <div class="p-1" :class="index === selectedIndex ? '' : 'opacity-50'">
            <Card>
              <CardContent
                class="flex aspect-square items-center justify-center p-2 bg-[#8A8A8A] rounded-md border border-gray-900"
              >
                <span class="text-base font-semibold">{{
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
      <CarouselPrevious />
      <CarouselNext />
    </Carousel>
    <Progress v-model="progress" class="w-full" />
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
