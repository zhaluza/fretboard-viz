<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";

const keys = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"];
const selectedKey = ref("C");
const selectedKeyType = ref("major");
const selectedIntervals = ref("all");

const strings = ["E", "B", "G", "D", "A", "E"];
const frets = Array.from({ length: 23 }, (_, i) => i); // 0 to 22 frets

const notesInKey = computed(() => {
  const keyIndex = keys.indexOf(selectedKey.value);
  const intervalPattern =
    selectedKeyType.value === "major"
      ? [0, 2, 4, 5, 7, 9, 11]
      : [0, 2, 3, 5, 7, 8, 10]; // Minor scale pattern
  return intervalPattern.map((interval) => keys[(keyIndex + interval) % 12]);
});

const scaleIntervals = computed(() => {
  return selectedKeyType.value === "major"
    ? ["1", "2", "3", "4", "5", "6", "7"]
    : ["1", "2", "b3", "4", "5", "b6", "b7"];
});

const activeIntervals = computed(() => {
  if (selectedKeyType.value === "major") {
    switch (selectedIntervals.value) {
      case "pentatonic":
        return [0, 1, 2, 4, 5];
      case "triads":
        return [0, 2, 4];
      default:
        return [0, 1, 2, 3, 4, 5, 6];
    }
  } else {
    // minor
    switch (selectedIntervals.value) {
      case "pentatonic":
        return [0, 2, 3, 4, 6];
      case "triads":
        return [0, 2, 4];
      default:
        return [0, 1, 2, 3, 4, 5, 6];
    }
  }
});

const getNoteAtFret = (string: string, fret: number) => {
  const stringIndex = keys.indexOf(string);
  return keys[(stringIndex + fret) % 12];
};

const isNoteInKey = (note: string) => {
  const noteIndex = notesInKey.value.indexOf(note);
  return noteIndex !== -1 && activeIntervals.value.includes(noteIndex);
};

const getIntervalForNote = (note: string) => {
  const noteIndex = notesInKey.value.indexOf(note);
  if (noteIndex === -1 || !activeIntervals.value.includes(noteIndex)) return "";
  return scaleIntervals.value[noteIndex];
};

const getNoteClass = (note: string, fret: number) => {
  let classes = "border-r border-gray-300 dark:border-gray-600 ";
  if (fret === 0)
    classes += "border-r-4 border-r-gray-600 dark:border-r-gray-400 mr-2 ";
  if (note === selectedKey.value)
    classes += "bg-green-500 text-white font-bold";
  else if (isNoteInKey(note) && getIntervalForNote(note))
    classes +=
      "bg-yellow-300 text-black font-bold dark:bg-blue-600 dark:text-white";
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
    <div class="mb-4 flex space-x-4">
      <div>
        <label for="key-select" class="mr-2">Select Key:</label>
        <select
          id="key-select"
          v-model="selectedKey"
          class="px-2 py-1 border rounded text-black dark:text-white bg-white dark:bg-gray-700"
        >
          <option v-for="key in keys" :key="key" :value="key">{{ key }}</option>
        </select>
      </div>
      <div>
        <label for="key-type-select" class="mr-2">Key Type:</label>
        <select
          id="key-type-select"
          v-model="selectedKeyType"
          class="px-2 py-1 border rounded text-black dark:text-white bg-white dark:bg-gray-700"
        >
          <option value="major">Major</option>
          <option value="minor">Minor</option>
        </select>
      </div>
      <div>
        <label for="intervals-select" class="mr-2">Intervals:</label>
        <select
          id="intervals-select"
          v-model="selectedIntervals"
          class="px-2 py-1 border rounded text-black dark:text-white bg-white dark:bg-gray-700"
        >
          <option value="all">All</option>
          <option value="pentatonic">Pentatonic</option>
          <option value="triads">Triads</option>
        </select>
      </div>
    </div>
    <p
      v-if="isSmallScreen"
      class="text-sm text-gray-600 dark:text-gray-400 mb-2"
    >
      Scroll horizontally to see all frets
    </p>
    <div
      class="border border-gray-300 dark:border-gray-600 w-full"
      :class="fretboardClass"
    >
      <div :class="{ 'inline-block': isSmallScreen, 'w-full': !isSmallScreen }">
        <!-- Fret numbers at the top -->
        <div class="flex sticky top-0 bg-white dark:bg-gray-900 z-10">
          <div
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300 dark:border-gray-600 sticky left-0 bg-white dark:bg-gray-900 z-20"
          >
            Fret
          </div>
          <div
            v-for="fret in frets"
            :key="fret"
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300 dark:border-gray-600"
            :class="{
              'border-r-4 border-r-gray-600 dark:border-r-gray-400': fret === 0,
            }"
          >
            {{ fret }}
          </div>
        </div>
        <!-- Fretboard -->
        <div v-for="string in strings" :key="string" class="flex">
          <div
            class="w-12 h-12 flex items-center justify-center text-sm border-r border-gray-300 dark:border-gray-600 sticky left-0 bg-white dark:bg-gray-900 z-10"
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
        <div class="flex sticky bottom-0 bg-white dark:bg-gray-900 z-10">
          <div
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300 dark:border-gray-600 sticky left-0 bg-white dark:bg-gray-900 z-20"
          >
            Fret
          </div>
          <div
            v-for="fret in frets"
            :key="fret"
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300 dark:border-gray-600"
            :class="{
              'border-r-4 border-r-gray-600 dark:border-r-gray-400': fret === 0,
            }"
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
