<script setup>
import { ref, onMounted } from "vue";
import WebGLFluid from "webgl-fluid";

// Refresh rate calculation
const refreshRate = ref("Calculating...");
const screenResolution = ref("");
const screenSizeInches = ref("");
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

// Determine screen resolution and categorize it dynamically
const calculateScreenResolution = () => {
  const width = window.screen.width;
  const height = window.screen.height;
  const devicePixelRatio = window.devicePixelRatio;

  // Calculate the resolution label in "K" format
  const widthK = ((width * devicePixelRatio) / 1000).toFixed(1); // Calculate K value by dividing by 1000

  screenResolution.value = `${widthK}K (${width * devicePixelRatio} x ${
    height * devicePixelRatio
  })`;

  // Calculate the diagonal resolution in pixels
  const diagonalPixels = Math.sqrt(Math.pow(width, 2) + Math.pow(height, 2));

  // Assume a standard PPI if none is available (fallback mechanism)
  const standardPPI = 96;
  const estimatedPPI = diagonalPixels / standardScreenDiagonal(); // Function to estimate screen diagonal

  // Calculate the screen size in inches
  const diagonalInches = (
    diagonalPixels / (estimatedPPI || standardPPI)
  ).toFixed(1);

  screenSizeInches.value = `${diagonalInches}"`;
};

// Estimate standard screen diagonal based on a formula or common values
const standardScreenDiagonal = () => {
  // Check for common devices or fallback to a standard estimate
  const diagonals = {
    mobile: 5.5,
    tablet: 10,
    laptop: 15.6,
    desktop: 24,
  };

  // Try to infer the device type based on the screen resolution
  const width = window.screen.width;
  const height = window.screen.height;

  // Use device type estimation to select a standard diagonal
  if (width < 768 && height < 1024) return diagonals.mobile; // Likely mobile
  if (width < 1024 && height < 1366) return diagonals.tablet; // Likely tablet
  if (width < 1920 && height < 1080) return diagonals.laptop; // Likely laptop

  return diagonals.desktop; // Default to desktop
};

// WebGL fluid simulation
const canvasRef = ref(null);

const triggerFluidAnimation = () => {
  const event = new KeyboardEvent("keydown", { key: " " });
  window.dispatchEvent(event);
};

onMounted(() => {
  calculateRefreshRate();
  calculateScreenResolution();

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

  window.addEventListener("click", triggerFluidAnimation);
});
</script>

<template>
  <div
    class="flex items-center justify-center min-h-screen bg-[#253238] text-white relative"
  >
    <canvas ref="canvasRef" class="absolute inset-0 w-full h-full" />
    <div class="relative z-10 text-center">
      <h1 class="text-2xl font-bold">Screen Refresh Rate: {{ refreshRate }}</h1>
      <p class="mt-2">Screen Resolution: {{ screenResolution }}</p>
      <p class="mt-2">Screen Size: {{ screenSizeInches }} inches</p>
    </div>
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
