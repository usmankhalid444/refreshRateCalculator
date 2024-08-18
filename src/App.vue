<script setup>
import { ref, onMounted } from "vue";

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

onMounted(() => {
  calculateRefreshRate();
});
</script>

<template>
  <div>
    <p>Screen Refresh Rate: {{ refreshRate }}</p>
  </div>
</template>
