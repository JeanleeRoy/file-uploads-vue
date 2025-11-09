<script setup lang="ts">
import { ref, onUnmounted } from 'vue';

const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10 MB

const files = ref<File[]>([]);
type FilePreview = { type: 'image' | 'video' | null; url: string | null };
const previews = ref<FilePreview[]>([]);

const isDropping = ref(false);

const acceptedFileTypes = [
  'image/*',
  'video/*',
  'application/pdf',
  'text/plain',
];

const isFileTypeAccepted = (file: File): boolean => {
  return acceptedFileTypes.some((type) => {
    if (type === 'image/*') return file.type.startsWith('image/');
    if (type === 'video/*') return file.type.startsWith('video/');
    return file.type === type;
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

const createPreview = (file: File): FilePreview => {
  try {
    if (file.type.startsWith('image/')) {
      return { type: 'image', url: URL.createObjectURL(file) };
    }
    if (file.type.startsWith('video/')) {
      return { type: 'video', url: URL.createObjectURL(file) };
    }
  } catch {
    // fallthrough
  }
  return { type: null, url: null };
};

const handleFileSelected = (e: Event) => {
  const target = e.target as HTMLInputElement | null;
  const dt = (e as DragEvent).dataTransfer;

  let fileList: File[] = [];

  if (dt && dt.files) {
    fileList = Array.from(dt.files);
  } else if (target && target.files) {
    fileList = Array.from(target.files);
  }

  const validFiles = fileList.filter(isValidFile);
  validFiles.forEach((file) => {
    const exists = files.value.find(
      (f) => f.name === file.name && f.size === file.size
    );
    if (!exists) {
      files.value.push(file);
      previews.value.push(createPreview(file));
    }
  });
};

const removeFile = (index: number) => {
  const p = previews.value[index];
  if (p && p.url) URL.revokeObjectURL(p.url);
  previews.value.splice(index, 1);
  files.value.splice(index, 1);
};

const formatBytes = (bytes: number, hasSpace = true) => {
  if (bytes === 0) return '0 B';
  const k = 1024;
  const sizes = ['B', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  const num = parseFloat((bytes / Math.pow(k, i)).toFixed(2));
  return num + (hasSpace ? ' ' : '') + sizes[i];
};

onUnmounted(() => {
  previews.value.forEach((p) => {
    if (p.url) URL.revokeObjectURL(p.url);
  });
});
</script>

<template>
  <div class="w-full max-w-3xl">
    <div
      class="relative flex flex-col items-center justify-center rounded-lg border-2 border-dashed border-gray-300 px-6 py-10 cursor-pointer hover:border-sky-300"
      :class="{ 'bg-sky-50 border-sky-300': isDropping }"
      @dragover.prevent="isDropping = true"
      @dragleave.prevent="isDropping = false"
      @drop.prevent="
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

      <!-- Icon -->
      <Icon name="solar:upload-linear" size="4rem" class="text-sky-400 mb-3" />

      <p class="text-lg font-semibold text-gray-700">
        Upload files <span class="text-sky-500">or drag and drop</span>
      </p>
      <p class="text-sm text-gray-400 mt-1">
        All files up to {{ formatBytes(MAX_FILE_SIZE, false) }}
      </p>
    </div>

    <!-- Files list -->
    <ul class="mt-4 space-y-3">
      <li
        v-for="(file, index) in files"
        :key="file.name + file.size"
        class="bg-white shadow-sm rounded-lg p-3 flex items-center gap-4"
      >
        <!-- Thumbnail/avatar -->
        <div
          class="w-12 h-12 flex items-center justify-center bg-gray-50 rounded overflow-hidden border border-gray-200"
        >
          <template v-if="previews[index] && previews[index].url">
            <img
              v-if="previews[index].type === 'image'"
              :src="previews[index].url"
              :alt="file.name"
              class="w-full h-full object-cover"
            />
            <video
              v-if="previews[index].type === 'video'"
              :src="previews[index].url"
              class="w-full h-full object-cover"
              muted
              :autoplay="true"
            />
          </template>
          <svg
            v-else
            class="w-6 h-6 text-gray-400"
            viewBox="0 0 24 24"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M4 7h16v10H4z"
              stroke="currentColor"
              stroke-width="1.2"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
            <path
              d="M8 11l2 2 3-3 5 5"
              stroke="currentColor"
              stroke-width="1.2"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
        </div>

        <!-- Name & size -->
        <div class="flex-1 min-w-0">
          <div class="flex items-center gap-2">
            <p class="text-sm font-medium text-gray-800 truncate">
              {{ file.name }}
            </p>
          </div>
          <p class="text-xs text-gray-400 mt-0.5">
            {{ formatBytes(file.size) }}
          </p>
        </div>

        <!-- Status pill -->
        <div class="flex items-center gap-3">
          <span
            class="inline-flex items-center gap-2 bg-emerald-100 text-emerald-700 text-xs font-medium px-3 py-1 rounded-full"
          >
            <svg
              class="w-3 h-3"
              viewBox="0 0 24 24"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M20 6L9 17l-5-5"
                stroke="currentColor"
                stroke-width="1.5"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
            Done
          </span>

          <!-- remove button -->
          <button
            class="text-gray-400 hover:text-gray-600 p-1 rounded-full"
            @click.prevent="removeFile(index)"
          >
            <Icon
              name="material-symbols:close-rounded"
              class="cursor-pointer"
            />
          </button>
        </div>
      </li>
    </ul>
  </div>
</template>
