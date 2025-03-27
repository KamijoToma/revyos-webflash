<script lang="ts">
// filepath: /workspaces/webusb-fastboot/src/components/Download.vue
import { defineComponent, ref } from 'vue';
import { ZstdCodec } from 'zstd-codec';

export default defineComponent({
  name: 'Download',
  setup() {
    const selectedFile = ref<File | null>(null);
    const outputFolder = ref<string>('');
    const isProcessing = ref(false);

    const selectFile = async () => {
      const input = document.createElement('input');
      input.type = 'file';
      input.accept = '.zst';
      input.onchange = (event: Event) => {
        const target = event.target as HTMLInputElement;
        if (target.files && target.files.length > 0) {
          selectedFile.value = target.files[0];
        }
      };
      input.click();
    };

    const selectFolder = async () => {
      // Use the File System Access API (modern browsers) to select a folder
      if ('showDirectoryPicker' in window) {
        const dirHandle = await (window as any).showDirectoryPicker();
        outputFolder.value = dirHandle.name;
        return dirHandle;
      } else {
        alert('Your browser does not support folder selection.');
        return null;
      }
    };

    const extractFile = async () => {
      if (!selectedFile.value) {
        alert('Please select a .zst file first.');
        return;
      }

      const dirHandle = await selectFolder();
      if (!dirHandle) return;

      isProcessing.value = true;

      try {
        const file = selectedFile.value;
        const fileStream = file.stream();
        const reader = fileStream.getReader();

        const Zstd = await new Promise<typeof ZstdCodec>((resolve) => {
          ZstdCodec.run((zstd: typeof ZstdCodec) => resolve(zstd));
        });

        const decompressor = new Zstd.Streaming();

        const writer = await dirHandle.getFileHandle(file.name.replace('.zst', ''), { create: true });
        const writableStream = await writer.createWritable();

        let chunk;
        while (!(chunk = await reader.read()).done) {
          const decompressedChunk = decompressor.decompressChunks(chunk.value);
          await writableStream.write(decompressedChunk);
        }

        await writableStream.close();
        alert('File extracted successfully!');
      } catch (error) {
        console.error('Error during extraction:', error);
        alert('An error occurred during extraction.');
      } finally {
        isProcessing.value = false;
      }
    };

    return {
      selectedFile,
      outputFolder,
      isProcessing,
      selectFile,
      extractFile,
    };
  },
});
</script>

<template>
  <div>
    <h1>Extract Zstd Archive</h1>
    <button @click="selectFile">Select .zst File</button>
    <p v-if="selectedFile">Selected File: {{ selectedFile.name }}</p>
    <button @click="extractFile" :disabled="isProcessing">
      {{ isProcessing ? 'Extracting...' : 'Extract to Folder' }}
    </button>
  </div>
</template>

<style scoped>
button {
  margin: 5px;
  padding: 10px;
  font-size: 16px;
}
</style>
