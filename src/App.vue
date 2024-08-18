<script setup>
import { ref, onMounted } from "vue";
import WebGLFluid from "webgl-fluid";

// Refresh rate calculation
const refreshRate = ref("Calculating...");
const sampleCount = 100; // Number of samples to collect before determining the result
let refreshRateSamples = [];

const calculateRefreshRate = () => {
  let lastTimestamp = 0;

  const step = (timestamp) => {
    if (lastTimestamp) {
      const delta = timestamp - lastTimestamp;

      if (delta > 0) {
        // Only calculate if delta is greater than 0
        const currentRefreshRate = Math.round(1000 / delta);
        refreshRateSamples.push(currentRefreshRate);
      }

      if (refreshRateSamples.length >= sampleCount) {
        // Calculate the most frequent refresh rate
        const frequencyMap = {};
        refreshRateSamples.forEach((rate) => {
          frequencyMap[rate] = (frequencyMap[rate] || 0) + 1;
        });

        const mostFrequentRate = Object.keys(frequencyMap).reduce((a, b) =>
          frequencyMap[a] > frequencyMap[b] ? a : b
        );

        refreshRate.value = `${mostFrequentRate} Hz`;
        return; // Stop further animation frames
      }
    }

    lastTimestamp = timestamp;
    requestAnimationFrame(step);
  };

  requestAnimationFrame(step);
};

// WebGL fluid simulation
const canvasRef = ref(null);

onMounted(() => {
  calculateRefreshRate();

  const options = {
    IMMEDIATE: true,
    TRIGGER: "hover",
    SIM_RESOLUTION: 128,
    DYE_RESOLUTION: 1024,
    CAPTURE_RESOLUTION: 512,
    DENSITY_DISSIPATION: 1,
    VELOCITY_DISSIPATION: 0.3,
    PRESSURE: 0.8,
    PRESSURE_ITERATIONS: 20,
    CURL: 30,
    SPLAT_RADIUS: 0.35,
    SPLAT_FORCE: 6000,
    SHADING: true,
    COLORFUL: true,
    COLOR_UPDATE_SPEED: 10,
    PAUSED: false,
    BACK_COLOR: { r: 0, g: 0, b: 0 },
    TRANSPARENT: false,
    BLOOM: true,
    BLOOM_ITERATIONS: 1,
    BLOOM_RESOLUTION: 256,
    BLOOM_INTENSITY: 0.8,
    BLOOM_THRESHOLD: 0.2,
    BLOOM_SOFT_KNEE: 0.2,
    SUNRAYS: true,
    SUNRAYS_RESOLUTION: 196,
    SUNRAYS_WEIGHT: 1.0,
  };

  WebGLFluid(canvasRef.value, options);
});
</script>

<template>
  <div
    class="flex items-center justify-center min-h-[85vh] bg-[#253238] text-white relative"
  >
    <canvas ref="canvasRef" class="absolute inset-0 w-full h-full" />
    <h1 class="text-2xl font-bold relative z-10">
      Screen Refresh Rate: {{ refreshRate }}
    </h1>
    <div class="absolute w-full bottom-5 text-center text-gray-300">
      Developed by Usman
    </div>
  </div>
</template>

<style scoped>
canvas {
  width: 100vw;
  height: 100vh;
}
</style>
