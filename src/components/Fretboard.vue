<script setup lang="ts">
import { ref, computed } from "vue";

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
    <div class="border border-gray-300 overflow-x-auto">
      <div class="inline-block min-w-full">
        <!-- Fret numbers at the top -->
        <div class="flex">
          <div
            v-for="fret in frets"
            :key="fret"
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300"
            :class="{ 'border-r-4 border-r-gray-600 mr-2': fret === 0 }"
          >
            {{ fret }}
          </div>
        </div>
        <!-- Fretboard -->
        <div v-for="string in strings" :key="string" class="flex">
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
        <div class="flex">
          <div
            v-for="fret in frets"
            :key="fret"
            class="w-12 h-6 flex items-center justify-center text-xs border-r border-gray-300"
            :class="{ 'border-r-4 border-r-gray-600 mr-2': fret === 0 }"
          >
            {{ fret }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
