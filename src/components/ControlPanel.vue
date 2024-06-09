<template>
  <div :class="['control-panel', { minified: isMinified }]">
    <h2>Control Panel</h2>
    <div>
      <h3>Select Bacteria</h3>
      <select v-model="selectedBacteria">
        <option v-for="bacteria in bacteriaTypes" :key="bacteria" :value="bacteria">{{ bacteria }}</option>
      </select>
      <button @click="applyBacteria">Apply Bacteria</button>
    </div>
    <div>
      <h3>Evolve Transmission</h3>
      <button @click="evolveTransmission('Saliva Spread')" :disabled="!canEvolve('Saliva Spread')">Saliva Spread ({{ costs.salivaspread }} PP)</button>
      <button @click="evolveTransmission('Food Residue')" :disabled="!canEvolve('Food Residue')">Food Residue ({{ costs.foodresidue }} PP)</button>
      <button @click="evolveTransmission('Plaque Formation')" :disabled="!canEvolve('Plaque Formation')">Plaque Formation ({{ costs.plaqueformation }} PP)</button>
    </div>
    <div>
      <h3>Evolve Symptoms</h3>
      <button @click="evolveSymptom('Enamel Erosion')" :disabled="!canEvolve('Enamel Erosion')">Enamel Erosion ({{ costs.enamelerosion }} PP)</button>
      <button @click="evolveSymptom('Cavity Formation')" :disabled="!canEvolve('Cavity Formation')">Cavity Formation ({{ costs.cavityformation }} PP)</button>
      <button @click="evolveSymptom('Gum Recession')" :disabled="!canEvolve('Gum Recession')">Gum Recession ({{ costs.gumrecession }} PP)</button>
    </div>
    <div>
      <h3>Evolve Abilities</h3>
      <button @click="evolveAbility('Acid Resistance')" :disabled="!canEvolve('Acid Resistance')">Acid Resistance ({{ costs.acidresistance }} PP)</button>
      <button @click="evolveAbility('Biofilm Fortification')" :disabled="!canEvolve('Biofilm Fortification')">Biofilm Fortification ({{ costs.biofilmfortification }} PP)</button>
      <button @click="evolveAbility('Antibiotic Resistance')" :disabled="!canEvolve('Antibiotic Resistance')">Antibiotic Resistance ({{ costs.antibioticresistance }} PP)</button>
    </div>
    <div>
      <h3>Resources</h3>
      <p>Plaque Points: {{ plaquePoints }}</p>
    </div>
    <div>
      <h3>Game Status</h3>
      <p>Infected Teeth: {{ infectedTeeth }} / 32</p>
      <p>Destroyed Teeth: {{ destroyedTeeth }} / 32</p>
    </div>
    <div>
      <h3>Hygiene Efforts</h3>
      <button @click="counterHygiene('Brushing')" :disabled="!canCounterHygiene('Brushing')">Brushing</button>
      <button @click="counterHygiene('Flossing')" :disabled="!canCounterHygiene('Flossing')">Flossing</button>
      <button @click="counterHygiene('Mouthwash')" :disabled="!canCounterHygiene('Mouthwash')">Mouthwash</button>
    </div>
    <div>
      <h3>Game Control</h3>
      <button @click="startGame" :disabled="isRunning">Start</button>
      <button @click="pauseGame" :disabled="!isRunning">Pause</button>
    </div>
    <div>
      <h3>Time</h3>
      <p>{{ elapsedTime }} seconds</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedBacteria: "",
      bacteriaTypes: ["Streptococcus mutans", "Lactobacillus", "Actinomyces"],
      plaquePoints: 10,
      infectedTeeth: 0,
      destroyedTeeth: 0,
      isRunning: false,
      elapsedTime: 0,
      timer: null,
      transmissionMethods: {
        salivaspread: false,
        foodresidue: false,
        plaqueformation: false,
      },
      symptoms: {
        enamelerosion: false,
        cavityformation: false,
        gumrecession: false,
      },
      abilities: {
        acidresistance: false,
        biofilmfortification: false,
        antibioticresistance: false,
      },
      costs: {
        salivaspread: 3,
        foodresidue: 3,
        plaqueformation: 3,
        enamelerosion: 5,
        cavityformation: 5,
        gumrecession: 5,
        acidresistance: 7,
        biofilmfortification: 7,
        antibioticresistance: 7,
      },
      isMinified: false,
    };
  },
  methods: {
    toggleMinify() {
      this.isMinified = !this.isMinified;
    },
    canEvolve(item) {
      if (this.transmissionMethods[item.toLowerCase().replace(" ", "")] !== undefined) {
        return !this.transmissionMethods[item.toLowerCase().replace(" ", "")] && this.plaquePoints >= this.costs[item.toLowerCase().replace(" ", "")];
      }
      if (this.symptoms[item.toLowerCase().replace(" ", "")] !== undefined) {
        return !this.symptoms[item.toLowerCase().replace(" ", "")] && this.plaquePoints >= this.costs[item.toLowerCase().replace(" ", "")];
      }
      if (this.abilities[item.toLowerCase().replace(" ", "")] !== undefined) {
        return !this.abilities[item.toLowerCase().replace(" ", "")] && this.plaquePoints >= this.costs[item.toLowerCase().replace(" ", "")];
      }
      return false;
    },
    applyBacteria() {
      this.$emit("apply-bacteria", this.selectedBacteria);
    },
    evolveTransmission(method) {
      if (this.canEvolve(method)) {
        this.transmissionMethods[method.toLowerCase().replace(" ", "")] = true;
        this.plaquePoints -= this.costs[method.toLowerCase().replace(" ", "")];
        this.$emit("evolve-transmission", method);
      }
    },
    evolveSymptom(symptom) {
      if (this.canEvolve(symptom)) {
        this.symptoms[symptom.toLowerCase().replace(" ", "")] = true;
        this.plaquePoints -= this.costs[symptom.toLowerCase().replace(" ", "")];
        this.$emit("evolve-symptom", symptom);
      }
    },
    evolveAbility(ability) {
      if (this.canEvolve(ability)) {
        this.abilities[ability.toLowerCase().replace(" ", "")] = true;
        this.plaquePoints -= this.costs[ability.toLowerCase().replace(" ", "")];
        this.$emit("evolve-ability", ability);
      }
    },
    canCounterHygiene(action) {
      return true; // Add logic for hygiene countermeasure feasibility
    },
    counterHygiene(action) {
      // Add logic for countering hygiene efforts
    },
    startGame() {
      if (!this.isRunning) {
        this.isRunning = true;
        this.timer = setInterval(() => {
          this.elapsedTime++;
          this.gameTick();
        }, 1000);
      }
    },
    pauseGame() {
      if (this.isRunning) {
        clearInterval(this.timer);
        this.isRunning = false;
      }
    },
    gameTick() {
      // Game logic to spread bacteria and affect teeth
      if (this.transmissionMethods.salivaspread || this.transmissionMethods.foodresidue || this.transmissionMethods.plaqueformation) {
        this.infectTeeth();
      }
      this.updateTeethStatus();
      this.$emit("update-game-status", { infectedTeeth: this.infectedTeeth, destroyedTeeth: this.destroyedTeeth });
    },
    infectTeeth() {
      if (this.infectedTeeth < 32) {
        this.infectedTeeth++;
        this.plaquePoints += 2; // Earn plaque points for infecting teeth
      }
    },
    updateTeethStatus() {
      if (this.symptoms.enamelerosion || this.symptoms.cavityformation || this.symptoms.gumrecession) {
        if (this.infectedTeeth > 0 && this.destroyedTeeth < 32) {
          this.destroyedTeeth++;
          this.infectedTeeth--;
        }
      }
    },
  },
  watch: {
    infectedTeeth(newVal) {
      this.$emit("update-game-status", { infectedTeeth: newVal, destroyedTeeth: this.destroyedTeeth });
    },
    destroyedTeeth(newVal) {
      this.$emit("update-game-status", { infectedTeeth: this.infectedTeeth, destroyedTeeth: newVal });
    },
  },
};
</script>

<style scoped>
.control-panel {
  padding: 1rem;
  background: #f0f0f0;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  transition: width 0.3s ease;
  height: 100vh;
}

.control-panel.minified {
  width: 200px; /* Adjust the width as needed */
}

.toggle-button {
  margin-bottom: 0.5rem;
}

h2,
h3 {
  margin: 0;
}

button {
  margin: 0.2rem 0;
}
</style>
