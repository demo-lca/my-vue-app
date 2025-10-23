<script setup>
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <div>
    <!-- 토글 버튼 -->
    <button @click="togglePanel" class="secondary small">
      {{ isCollapsed ? '>' : '<' }}
    </button>

    <div class="container">
      <!-- 왼쪽 패널 -->
      <div id="left-panel" class="panel" :class="{ collapsed: isCollapsed }">
        <h2>Simulation Options</h2>

        <form @submit.prevent="runSimulation" @reset="resetForm">
          <!-- Ship Info -->
          <h3>Ship Information</h3>
          <div class="section card">
            <div class="form-row">
              <label>Ship Type</label><br>
              <select v-model="shipType">
                <option>LNGC</option>
                <option>LPGC</option>
                <option>Container</option>
                <option>VLCC</option>
                <option>Car Carrier</option>
                <option>Bulk</option>
              </select>
            </div>

            <div class="form-row">
              <label>Ship Size</label><br>
              <input type="number" v-model="shipSizeValue" placeholder="Enter size" min="0" />
              <select v-model="shipSizeUnit">
                <option>TEU</option>
                <option>DWT</option>
                <option>CBM</option>
                <option>CEU</option>
              </select>
            </div>
          </div>

          <!-- Annual Fuel Consumption -->
          <h3>Annual Fuel Consumption</h3>
          <div class="section">
            <table class="card input-table">
              <thead>
                <tr>
                  <th>Fuel Type</th>
                  <th>Consumption (g/year)</th>
                  <th>Action</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(item, idx) in fuelData" :key="idx">
                  <td>
                    <select v-model="item.type">
                      <option>Fuel 1</option>
                      <option>Fuel 2</option>
                      <option>Fuel 3</option>
                      <option>Fuel 4</option>
                    </select>
                  </td>
                  <td>
                    <input type="number" v-model="item.value" placeholder="e.g. 100000" min="0" />
                    <span class="unit">g/year</span>
                  </td>
                  <td>
                    <button type="button" @click="removeFuel(idx)">✕</button>
                  </td>
                </tr>
              </tbody>
            </table>
            <button type="button" @click="addFuel">+ Add Fuel Data</button>
          </div>

          <!-- Fuel Option -->
          <h3>Fuel Option</h3>
          <div class="section">
            <table class="card input-table">
              <thead>
                <tr>
                  <th>Fuel Type</th>
                  <th>Cost (USD)</th>
                  <th>Intensity (gCO2/MJ)</th>
                  <th>Action</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(opt, idx) in fuelOptions" :key="idx">
                  <td>
                    <select v-model="opt.type">
                      <option>Fuel 1</option>
                      <option>Fuel 2</option>
                      <option>Fuel 3</option>
                      <option>Fuel 4</option>
                    </select>
                  </td>
                  <td>
                    <input type="number" v-model="opt.cost" placeholder="e.g. 120" min="0" step="0.01" />
                    <span class="unit">USD</span>
                  </td>
                  <td>
                    <input type="number" v-model="opt.intensity" placeholder="e.g. 80" min="0" step="0.01" />
                    <span class="unit">gCO2/MJ</span>
                  </td>
                  <td>
                    <button type="button" @click="removeFuelOption(idx)">✕</button>
                  </td>
                </tr>
              </tbody>
            </table>
            <button type="button" @click="addFuelOption">+ Add Fuel Option</button>
          </div>

          <!-- Conversion -->
          <h3>Conversion Target Fuel</h3>
          <div class="section card">
            <select v-model="conversionFuel">
              <option>Ammonia</option>
              <option>Methanol</option>
            </select>
          </div>

          <!-- Dry Dock -->
          <h3>Dry Dock Start Year</h3>
          <div class="section card">
            <select v-model="dryDockYear">
              <option v-for="year in dryDockYears" :key="year" :value="year">{{ year }}</option>
            </select>
          </div>

          <!-- ESD -->
          <h3>ESD Retrofit</h3>
          <div class="section card">
            <label v-for="esd in esdOptions" :key="esd">
              <input type="checkbox" v-model="selectedEsd" :value="esd" /> {{ esd }}
            </label>
          </div>

          <!-- Run/Reset -->
          <div class="actions">
            <button type="submit" class="primary">Run Simulation</button>
            <button type="reset" class="secondary">Reset</button>
          </div>
        </form>
      </div>

      <!-- 오른쪽 패널 -->
      <div id="right-panel" class="panel">
        <h2>Simulation Results</h2>

        <h3>Current Data</h3>
        <div class="card">
          <table>
            <tbody>
              <tr v-for="(row, idx) in resultCurrent" :key="idx">
                <td>{{ row.type }}</td>
                <td>{{ row.value }} g/year</td>
              </tr>
            </tbody>
          </table>
        </div>

        <h3>Simulation Results</h3>
        <div class="card">
          <table>
            <tbody>
              <tr>
                <td>Average Fuel Cost</td>
                <td>{{ avgCost }} USD</td>
              </tr>
            </tbody>
          </table>
        </div>

        <h3>Recommended Dry Dock Plan</h3>
        <div class="card">
          <table>
            <tbody>
              <tr>
                <td>Planned Dry Dock Year</td>
                <td>{{ dryDockYear }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    const currentYear = new Date().getFullYear();
    return {
      isCollapsed: false,
      shipType: "LNGC",
      shipSizeValue: null,
      shipSizeUnit: "TEU",
      fuelData: [{ type: "Fuel 1", value: 0 }],
      fuelOptions: [{ type: "Fuel 1", cost: 0, intensity: 0 }],
      conversionFuel: "Ammonia",
      dryDockYear: currentYear,
      dryDockYears: Array.from({ length: 6 }, (_, i) => currentYear + i),
      esdOptions: ["ESD1", "ESD2", "ESD3", "ESD4", "ESD5"],
      selectedEsd: [],
      resultCurrent: [],
      avgCost: 0
    };
  },
  methods: {
    togglePanel() {
      this.isCollapsed = !this.isCollapsed;
    },
    addFuel() {
      this.fuelData.push({ type: "Fuel 1", value: 0 });
    },
    removeFuel(idx) {
      this.fuelData.splice(idx, 1);
    },
    addFuelOption() {
      this.fuelOptions.push({ type: "Fuel 1", cost: 0, intensity: 0 });
    },
    removeFuelOption(idx) {
      this.fuelOptions.splice(idx, 1);
    },
    runSimulation() {
      // Current Data
      this.resultCurrent = this.fuelData.map(f => ({ type: f.type, value: f.value || 0 }));

      // Simulation Results (평균 비용 예시)
      const costs = this.fuelOptions.map(f => parseFloat(f.cost) || 0);
      this.avgCost = costs.length ? (costs.reduce((a, b) => a + b, 0) / costs.length).toFixed(2) : 0;

      // 사이드바 자동 접기
      this.isCollapsed = true;
    },
    resetForm() {
      this.fuelData = [{ type: "Fuel 1", value: 0 }];
      this.fuelOptions = [{ type: "Fuel 1", cost: 0, intensity: 0 }];
      this.resultCurrent = [];
      this.avgCost = 0;
      this.selectedEsd = [];
    }
  }
};
</script>

<style>
@import './assets/style.css';
</style>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
