<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";

const keys = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"];
const selectedKey = ref("C");

const strings = ["E", "B", "G", "D", "A", "E"];
const frets = Array.from({ length: 23 }, (_, i) => i); // 0 to 22 frets

const intervals = ["1", "2", "3", "4", "5", "6", "7"];

const notesInKey = computed(() => {
  const keyIndex = keys.indexOf(selectedKey.value);
  return intervals.map(
    (_, i) => keys[(keyIndex + [0, 2, 4, 5, 7, 9, 11][i]) % 12]
  );
});

const getNoteAtFret = (string: string, fret: number) => {
  const stringIndex = keys.indexOf(string);
  return keys[(stringIndex + fret) % 12];
};

const isNoteInKey = (note: string) => notesInKey.value.includes(note);

const getIntervalForNote = (note: string) => {
  const index = notesInKey.value.indexOf(note);
  return index > 0 ? intervals[index] : "";
};

const getNoteClass = (note: string, fret: number) => {
  let classes = "border-r border-gray-300 ";
  if (fret === 0) classes += "border-r-4 border-r-gray-600 mr-2 ";
  if (note === selectedKey.value)
    classes += "bg-green-500 text-white font-bold";
  else if (isNoteInKey(note)) classes += "bg-yellow-300 text-black font-bold";
  return classes;
};

const isSmallScreen = ref(false);

const updateScreenSize = () => {
  isSmallScreen.value = window.innerWidth < 1024; // Adjust this value as needed
};

onMounted(() => {
  updateScreenSize();
  window.addEventListener("resize", updateScreenSize);
});

onUnmounted(() => {
  window.removeEventListener("resize", updateScreenSize);
});

const fretboardClass = computed(() => {
  return isSmallScreen.value ? "overflow-x-auto" : "overflow-x-visible";
});
</script>

<template>
  <div class="flex flex-col items-center mt-8">
    <div class="mb-4">
      <label for="key-select" class="mr-2">Select Key:</label>
      <select
        id="key-select"
        v-model="selectedKey"
        class="px-2 py-1 border rounded"
      >
        <option v-for="key in keys" :key="key" :value="key">{{ key }}</option>
      </select>
    </div>
    <p v-if="isSmallScreen" class="text-sm text-gray-600 mb-2">
      Scroll horizontally to see all frets
    </p>
    <div class="border border-gray-300 w-full" :class="fretboardClass">
      <div :class="{ 'inline-block': isSmallScreen, 'w-full': !isSmallScreen }">
        <!-- Fret numbers at the top -->
        <div class="flex sticky top-0 bg-white z-10">
          <div
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300 sticky left-0 bg-white z-20"
          >
            Fret
          </div>
          <div
            v-for="fret in frets"
            :key="fret"
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300"
            :class="{ 'border-r-4 border-r-gray-600': fret === 0 }"
          >
            {{ fret }}
          </div>
        </div>
        <!-- Fretboard -->
        <div v-for="string in strings" :key="string" class="flex">
          <div
            class="w-12 h-12 flex items-center justify-center text-sm border-r border-gray-300 sticky left-0 bg-white z-10"
          >
            {{ string }}
          </div>
          <div
            v-for="fret in frets"
            :key="fret"
            class="w-12 h-12 flex flex-col items-center justify-center text-sm relative"
            :class="getNoteClass(getNoteAtFret(string, fret), fret)"
          >
            <span>{{ getNoteAtFret(string, fret) }}</span>
            <span
              v-if="
                isNoteInKey(getNoteAtFret(string, fret)) &&
                getNoteAtFret(string, fret) !== selectedKey
              "
              class="text-xs absolute bottom-0.5"
            >
              {{ getIntervalForNote(getNoteAtFret(string, fret)) }}
            </span>
          </div>
        </div>
        <!-- Fret numbers at the bottom -->
        <div class="flex sticky bottom-0 bg-white z-10">
          <div
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300 sticky left-0 bg-white z-20"
          >
            Fret
          </div>
          <div
            v-for="fret in frets"
            :key="fret"
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300"
            :class="{ 'border-r-4 border-r-gray-600': fret === 0 }"
          >
            {{ fret }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.overflow-x-auto {
  -webkit-overflow-scrolling: touch;
}

@media (min-width: 1024px) {
  .w-full {
    width: max-content;
    margin: 0 auto;
  }
}
</style>
