<script setup lang="ts">
import { ref, computed } from "vue";

const keys = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"];
const selectedKey = ref("C");

const strings = ["E", "B", "G", "D", "A", "E"];
const frets = Array.from({ length: 12 }, (_, i) => i);

const notesInKey = computed(() => {
  const keyIndex = keys.indexOf(selectedKey.value);
  return [
    keys[keyIndex],
    keys[(keyIndex + 2) % 12],
    keys[(keyIndex + 4) % 12],
    keys[(keyIndex + 5) % 12],
    keys[(keyIndex + 7) % 12],
    keys[(keyIndex + 9) % 12],
    keys[(keyIndex + 11) % 12],
  ];
});

const getNoteAtFret = (string: string, fret: number) => {
  const stringIndex = keys.indexOf(string);
  return keys[(stringIndex + fret) % 12];
};

const isNoteInKey = (note: string) => notesInKey.value.includes(note);
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
    <div class="border border-gray-300">
      <div v-for="string in strings" :key="string" class="flex">
        <div
          v-for="fret in frets"
          :key="fret"
          class="w-10 h-10 border border-gray-300 flex items-center justify-center text-sm"
          :class="{
            'bg-green-500 text-white font-bold': isNoteInKey(
              getNoteAtFret(string, fret)
            ),
          }"
        >
          {{ getNoteAtFret(string, fret) }}
        </div>
      </div>
    </div>
  </div>
</template>
