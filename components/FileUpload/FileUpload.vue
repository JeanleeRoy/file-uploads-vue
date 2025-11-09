<script setup lang="ts">
import { ref } from "vue";

const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10 MB

const files = ref<File[]>([]);

const acceptedFileTypes = [
  "image/*",
  "video/*",
  "application/pdf",
  "text/plain",
];

const isFileTypeAccepted = (file: File): boolean => {
  return acceptedFileTypes.some((type) => {
    if (type === "image/*") {
      return file.type.startsWith("image/");
    } else if (type === "video/*") {
      return file.type.startsWith("video/");
    } else {
      return file.type === type;
    }
  });
};

const isValidFile = (file: File): boolean => {
  if (!isFileTypeAccepted(file)) {
    console.warn(`File type not accepted: ${file.name}`);
    return false;
  }
  if (file.size > MAX_FILE_SIZE) {
    console.warn(`File size exceeds limit: ${file.name}`);
    return false;
  }

  return true;
};

const handleFileSelected = (e: Event) => {
  const target = e.target as HTMLInputElement;
  if (!target.files) return;

  const selectedFiles = Array.from(target.files);
  const validFiles = selectedFiles.filter(isValidFile);
  files.value = validFiles;
};
</script>

<template>
  <label class="block border w-full max-w-3xl p-3 rounded cursor-pointer">
    <input
      type="file"
      :accept="acceptedFileTypes.join(',')"
      class="hidden"
      @change="handleFileSelected"
    />
    <p class="text-center text-gray-500">Upload files</p>
  </label>

  <ul class="mt-4">
    <li v-for="file in files" :key="file.name">
      {{ file.name }}
    </li>
  </ul>
</template>
