<script setup lang="ts">
import { ref } from "vue";

const files = ref<File[]>([]);

const handleFileSelected = (e: Event) => {
  const target = e.target as HTMLInputElement;
  if (!target.files) return;

  const selectedFiles = target.files;
  files.value = Array.from(selectedFiles);
  console.log("Selected files:", files.value);
};
</script>

<template>
  <label class="block border w-full max-w-3xl p-3 rounded cursor-pointer">
    <input type="file" class="hidden" @change="handleFileSelected" />
    <p class="text-center text-gray-500">Upload files</p>
  </label>

  <div class="mt-3">
    <ul>
      <li v-for="file in files" :key="file.name">
        {{ file.name }}
      </li>
    </ul>
  </div>
</template>
