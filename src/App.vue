<template>
  <div id="app">
    <ThreeScene ref="threeScene" />
    <div class="ui-panel" :class="{ minified: isMinified }">
      <button class="toggle-button" @click="toggleMinify">{{ isMinified ? "Expand" : "Minify" }}</button>
      <ControlPanel
        :isMinified="isMinified"
        @apply-bacteria="handleApplyBacteria"
        @evolve-transmission="handleEvolveTransmission"
        @evolve-symptom="handleEvolveSymptom"
        @evolve-ability="handleEvolveAbility"
        @update-game-status="handleUpdateGameStatus"
      />
    </div>
  </div>
</template>

<script>
import ThreeScene from "./components/ThreeScene.vue";
import ControlPanel from "./components/ControlPanel.vue";

export default {
  name: "App",
  components: {
    ThreeScene,
    ControlPanel,
  },
  data() {
    return {
      isMinified: true,
    };
  },
  methods: {
    handleApplyBacteria(detail) {
      window.dispatchEvent(new CustomEvent("apply-bacteria", { detail }));
    },
    handleEvolveTransmission(method) {
      window.dispatchEvent(new CustomEvent("evolve-transmission", { detail: method }));
    },
    handleEvolveSymptom(symptom) {
      window.dispatchEvent(new CustomEvent("evolve-symptom", { detail: symptom }));
    },
    handleEvolveAbility(ability) {
      window.dispatchEvent(new CustomEvent("evolve-ability", { detail: ability }));
    },
    handleUpdateGameStatus(status) {
      window.dispatchEvent(new CustomEvent("update-game-status", { detail: status }));
    },
    toggleMinify() {
      this.isMinified = !this.isMinified;
    },
  },
};
</script>

<style>
#app {
  width: 100%;
  height: 100vh;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

.ui-panel {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background: rgba(255, 255, 255, 0.9);
  padding: 1rem;
  border-bottom: 1px solid #ccc;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  transition: width 0.3s ease;
}

.ui-panel.minified {
  width: 300px; /* Adjust the width as needed */
  padding: 0.5rem;
  overflow: hidden;
}

.control-panel.hidden {
  display: none;
}

.toggle-button {
  margin-bottom: 1rem;
  position: absolute;
  top: 1rem;
  right: 1rem;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0;
}
</style>
