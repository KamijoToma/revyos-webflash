<!-- filepath: /workspaces/webusb-fastboot/src/components/FastBootFlash.vue -->
<template>
    <div class="fastboot-flash">
        <h1>Fastboot Flashing</h1>
        <div v-if="currentStep === 1">
            <h2>Step 1: Connect to Stage 1 USB Device</h2>
            <button @click="connectToDevice" :disabled="isProcessing">
                {{ isProcessing ? "Connecting..." : "Connect" }}
            </button>
        </div>
        <div v-else-if="currentStep === 2">
            <h2>Step 2: Flash uboot.bin to RAM</h2>
            <input type="file" @change="onFileSelected('ubootBin', $event)" />
            <button @click="flashUbootToRam" :disabled="!files.ubootBin || isProcessing">
                {{ isProcessing ? "Flashing..." : "Flash to RAM" }}
            </button>
        </div>
        <div v-else-if="currentStep === 3">
            <h2>Step 3: Reboot to Stage 2</h2>
            <button @click="rebootToStage2" :disabled="isProcessing">
                {{ isProcessing ? "Rebooting..." : "Reboot" }}
            </button>
        </div>
        <div v-else-if="currentStep === 4">
            <h2>Step 4: Connect to Stage 2 USB Device</h2>
            <button @click="connectToStage2" :disabled="isProcessing">
                {{ isProcessing ? "Waiting..." : "Connect" }}
            </button>
        </div>
        <div v-else-if="currentStep === 5">
            <h2>Step 5: Flash Files to Device</h2>
            <label>uboot.bin:</label>
            <input type="file" @change="onFileSelected('ubootBin', $event)" />
            <label>boot.ext4:</label>
            <input type="file" @change="onFileSelected('bootExt4', $event)" />
            <label>root.ext4:</label>
            <input type="file" @change="onFileSelected('rootExt4', $event)" />
            <button @click="flashFilesToDevice"
                :disabled="!files.ubootBin || !files.bootExt4 || !files.rootExt4 || isProcessing">
                {{ isProcessing ? "Flashing..." : "Flash Files" }}
            </button>
        </div>
        <div v-else-if="currentStep === 6">
            <h2>Step 6: Reboot Device</h2>
            <button @click="rebootDevice" :disabled="isProcessing">
                {{ isProcessing ? "Rebooting..." : "Reboot" }}
            </button>
        </div>
        <div v-if="status" class="status">{{ status }}</div>
        <div class="navigation">
            <button @click="prevStep" :disabled="currentStep === 1 || isProcessing">Previous</button>
            <button @click="nextStep" :disabled="currentStep === 6 || isProcessing">Next</button>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import * as fastboot from "android-fastboot-ts";

const currentStep = ref(1);
const isProcessing = ref(false);
const status = ref("");
const files = ref<{ [key: string]: File | null }>({
    ubootBin: null,
    bootExt4: null,
    rootExt4: null,
});
const device = new fastboot.FastbootDevice();

// Enable verbose debug logging
fastboot.setDebugLevel(2);

function onFileSelected(key: string, event: Event) {
    const input = event.target as HTMLInputElement;
    if (input.files && input.files[0]) {
        files.value[key] = input.files[0];
    }
}

async function connectToDevice() {
    isProcessing.value = true;
    status.value = "Connecting to stage 1 USB device...";
    try {
        await device.connect();
        const product = await device.getVariable("product");
        const serial = await device.getVariable("serialno");
        status.value = `Connected to ${product} (serial: ${serial})`;
    } catch (error: any) {
        status.value = `Error: ${error.message}`;
    } finally {
        isProcessing.value = false;
    }
}

async function flashUbootToRam() {
    if (!files.value.ubootBin) return;
    isProcessing.value = true;
    status.value = "Flashing uboot.bin to RAM...";
    try {
        await device.flashBlob("ram", files.value.ubootBin);
        status.value = "Flashed uboot.bin to RAM successfully.";
    } catch (error: any) {
        status.value = `Error: ${error.message}`;
    } finally {
        isProcessing.value = false;
    }
}

async function rebootToStage2() {
    isProcessing.value = true;
    status.value = "Rebooting to stage 2...";
    try {
        await device.runCommand("reboot");
        status.value = "Rebooted to stage 2.";
    } catch (error: any) {
        status.value = `Error: ${error.message}`;
    } finally {
        isProcessing.value = false;
    }
}

async function connectToStage2() {
    isProcessing.value = true;
    status.value = "Waiting for stage 2 USB device...";
    try {
        let connected = false;
        while (!connected) {
            try {
                await device.connect();
                connected = true;
            } catch {
                await new Promise((resolve) => setTimeout(resolve, 1000));
            }
        }
        status.value = "Connected to stage 2 USB device.";
    } catch (error: any) {
        status.value = `Error: ${error.message}`;
    } finally {
        isProcessing.value = false;
    }
}

async function flashFilesToDevice() {
    if (!files.value.ubootBin || !files.value.bootExt4 || !files.value.rootExt4) return;
    isProcessing.value = true;
    status.value = "Flashing files to device...";
    try {
        await device.flashBlob("uboot", files.value.ubootBin);
        await device.flashBlob("boot", files.value.bootExt4);
        await device.flashBlob("root", files.value.rootExt4);
        status.value = "Flashed all files successfully.";
    } catch (error: any) {
        status.value = `Error: ${error.message}`;
    } finally {
        isProcessing.value = false;
    }
}

async function rebootDevice() {
    isProcessing.value = true;
    status.value = "Rebooting the device...";
    try {
        await device.runCommand("reboot");
        status.value = "Device rebooted successfully.";
    } catch (error: any) {
        status.value = `Error: ${error.message}`;
    } finally {
        isProcessing.value = false;
    }
}

function nextStep() {
    if (currentStep.value < 6) currentStep.value++;
}

function prevStep() {
    if (currentStep.value > 1) currentStep.value--;
}
</script>

<style scoped>
.fastboot-flash {
    text-align: center;
    margin-top: 20px;
}

button {
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
    margin: 10px;
}

.status {
    margin-top: 20px;
    font-size: 14px;
    color: #555;
}

.navigation {
    margin-top: 20px;
}
</style>
