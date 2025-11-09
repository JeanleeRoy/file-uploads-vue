<script setup lang="ts">
import { ref } from 'vue';

const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10 MB

const files = ref<File[]>([]);

const isDropping = ref(false);

const acceptedFileTypes = [
  'image/*',
  'video/*',
  'application/pdf',
  'text/plain',
];

type FilePreview = { type: 'image' | 'video'; url: string } | null;

const freeOldPreviewURL = (preview: FilePreview) => {
  if (!preview) return;
  URL.revokeObjectURL(preview.url);
};

const previews = computed<FilePreview[]>((oldPreviews) => {
  oldPreviews?.forEach(freeOldPreviewURL);

  return files.value.map((file) => {
    if (file.type.startsWith('image/')) {
      return {
        type: 'image',
        url: URL.createObjectURL(file),
      };
    } else if (file.type.startsWith('video/')) {
      return {
        type: 'video',
        url: URL.createObjectURL(file),
      };
    }

    return null;
  });
});

const isFileTypeAccepted = (file: File): boolean => {
  return acceptedFileTypes.some((type) => {
    if (type === 'image/*') {
      return file.type.startsWith('image/');
    } else if (type === 'video/*') {
      return file.type.startsWith('video/');
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
  validFiles.forEach((file) => {
    if (
      !files.value.find((f) => f.name === file.name && f.size === file.size)
    ) {
      files.value.push(file);
    }
  });
};

onUnmounted(() => {
  previews.value.forEach(freeOldPreviewURL);
});
</script>

<template>
  <div
    class="relative flex border-3 border-dashed border-gray-300 w-full max-w-3xl px-3 py-12 rounded"
    :class="{ 'bg-sky-100/50 border-sky-300': isDropping }"
    @dragover="isDropping = true"
    @dragleave="isDropping = false"
    @drop="
      isDropping = false;
      handleFileSelected($event);
    "
  >
    <input
      class="absolute inset-0 opacity-0 cursor-pointer"
      type="file"
      :accept="acceptedFileTypes.join(',')"
      multiple
      @change="handleFileSelected"
    />
    <div class="w-full text-center text-gray-500 self-center">
      <p class="text-xl" :class="{ 'text-sky-500': isDropping }">
        Upload files
      </p>
      <p
        class="w-full text-center text-gray-400 self-center"
        :class="{ 'text-sky-300': isDropping }"
      >
        Drag and drop files here
      </p>
    </div>
  </div>

  <ul class="mt-4">
    <li v-for="(file, index) in files" :key="file.name" class="pb-4">
      {{ file.name }}

      <div v-if="previews[index]" class="mt-2 max-h-48 overflow-hidden">
        <img
          v-if="previews[index].type === 'image'"
          :src="previews[index].url"
          :alt="file.name"
        />
        <video
          v-if="previews[index].type === 'video'"
          :src="previews[index].url"
          muted
          :autoplay="false"
        />
      </div>
    </li>
  </ul>
</template>
