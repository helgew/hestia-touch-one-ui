<template>
  <div id="home-screen" class="home-screen" @click="resetScreenAwake">
    <div v-if="screensaver" class="dim" @click="screensaver = false"/>
      <div class="top-container">
        <div class="left-column-top">
          <div class="mode-btn heat"
               :class="{
          animated: modes.heat.running,
          'color-heat': modes.heat.active || modes.heat.running,
          'color-off': !(modes.heat.active || modes.heat.running)
        }"
               @click="toggleHeat">
            <icon-heat size="68%"/>
          </div>
          <div v-if="modes.heat.active"
               class="mode-btn boost color-off"
               :class="{ animated: modes.heat.boostEnabled }"
               @click="toggleBoost"
          >
            <icon-boost size="60%"/>
          </div>
        </div>
        <div class="center-column-top">
          <div class="datetimedisplay unselectable">
            <DateTimeDisplay/>
          </div>
        </div>
        <div class="right-colum-top">
          <div class="mode-btn settings color-off" @click="toggleSettingsScreen">
            <icon-settings size="50%"/>
          </div>
        </div>
      </div>
      <div class="center-container">
        <div class="left-column-center">
          <div :class="tempClass"
               @click="toggleControls"
               v-html="targetTemperature">
          </div>
          <div v-if="showTempControls" class="temp-controls unselectable">
            <div @click="decrement">-</div>
            <div @click="increment">+</div>
          </div>
        </div>
        <div class="right-column-center">
          <div class="flex-row">
            <div class="current-temp current-weather unselectable">
              {{ currentTemperature }}<span class="symbol">°</span>
            </div>
            <div class="home-icon unselectable">
              <icon-home size="25"/>
            </div>
          </div>
          <div class="flex-row">
            <div class="outdoors-temp current-weather unselectable">
              {{ outdoorsTemperature }}<span class="symbol">°</span>
            </div>
            <div class="outdoors-icon unselectable">
              <icon-outdoors size="25"/>
            </div>
          </div>
          <div class="flex-row">
            <div class="current-humidity current-weather unselectable">
              {{ currentHumidity }}<span class="symbol">%</span>
            </div>
            <div class="humidity-icon">
              <icon-humidity size="20"/>
            </div>
          </div>
          <div class="flex-row">
            <div class="current-pressure current-weather unselectable">
              {{ currentPressure }} hPa
            </div>
            <div class="pressure-icon">
              <icon-pressure size="20"/>
            </div>
          </div>
        </div>
      </div>
      <div class="bottom-container">
        <div class="power-setting-text">{{ powerSettingText }}</div>
        <div class="comfort-mode">
          <span v-if="comfortMode">Comfort mode</span>
          <span v-else>Comfort range: &plusmn;{{ hysteresis }}&deg;</span>
        </div>
      </div>
  </div>
</template>

<script>
import {mapState} from 'vuex'
import iconHeat from '@/components/icon-heat.vue'
import iconBoost from '@/components/icon-boost.vue'
import iconHome from '@/components/icon-home.vue'
import iconHumidity from '@/components/icon-humidity.vue'
import iconPressure from '@/components/icon-pressure.vue'
import iconSettings from '@/components/icon-settings.vue'
import DateTimeDisplay from "@/components/datetime.vue";
import IconSettings from "@/components/icon-settings";
import IconOutdoors from "@/components/icon-outdoors";

export default {
  data() {
    return {
      showTempControls: false,
      tempClass: 'hide-controls',
      secsControlsShowing: 0,
      screensaver: false,
      screenAwake: 0
    }
  }, components: {
    IconOutdoors,
    IconSettings,
    DateTimeDisplay,
    iconHeat,
    iconBoost,
    iconHome,
    iconHumidity,
    iconPressure,
    iconSettings
  }, computed: {
    // Some variables in $store.state we want to read
    // https://vuex.vuejs.org/guide/state.html#the-mapstate-helper
    ...mapState(
      ['comfortMode', 'currentTemperature', 'currentHumidity', 'currentPressure', 'icons',
        'hysteresis', 'modes', 'selectedMode', 'outdoorsTemperature']),
    powerSettingText() {
      if (this.modes['heat'].active) {
        return `Heating auto`
      }
      return `Heating off`
    }, targetTemperature() {
      return this.$store.getters.targetTemperature
    }
  }, methods: {
    decrement() {
      this.$store.commit('decrementTargetValue')
    }, increment() {
      this.$store.commit('incrementTargetValue')
    }, toggleSettingsScreen() {
      this.$store.commit('toggleSettingsScreen')
    }, startTimers() {
      this.sleepTimer = setInterval(() => {
        if (this.showTempControls && this.secsControlsShowing > 9) {
          this.screenAwake = 0
          this.showTempControls = false
          this.tempClass = 'hide-controls'
        } else if (this.showTempControls) {
          this.secsControlsShowing += 1
        }
      }, 1000)
      this.screenTimer = setInterval(() => {
        if (!this.screensaver && this.screenAwake > 29) {
          this.screenAwake = 0
          this.screensaver = true
        } else if (!this.screensaver && !this.showTempControls) {
          this.screenAwake += 1
        }
      }, 1000)
    }, toggleControls() {
      if (!this.showTempControls) {
        this.showTempControls = true
        this.tempClass = 'show-controls unselectable'
        this.secsControlsShowing = 0
      }
    }, toggleHeat() {
      if (!this.modes.heat.boostEnabled) {
        var state = this.modes.heat.active ? "OFF" : "ON"
        this.$store.commit('selectPowerSetting', {mode: 'heat', powerOption: state})
      }
    }, toggleBoost() {
      var state = this.modes.heat.boostEnabled ? "ON" : "Boost"
      this.$store.commit('selectPowerSetting', {mode: 'heat', powerOption: state})
    }, resetScreenAwake() {
      this.screenAwake = 0
      this.secsControlsShowing = 0
    }
  }, mounted() {
    this.startTimers()
  }
}
</script>

<!-- "scoped" attribute limits CSS to this component only -->
<style scoped>
@keyframes changecolor {
  from {
    color: black;
  }

  to {
  }
}

/* Apply to any mode that is running to give it a glowing effect */
.animated {
  animation: changecolor 2s steps(8) infinite alternate;
}

.home-screen {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.home-screen > div {
  flex: 1;
}

.top-container {
  height: 25vh;
  display: flex;
  flex-direction: row;
  flex: 1;
  margin-top: 2%;
}

.left-column-top {
  width: 30vw;
  display: flex;
}

.center-column-top {
  width: 40vw;
  display: flex;
  justify-content: center;
  align-items: center;
}

.right-colum-top {
  width: 30vw;
  display: flex;
  justify-content: right;
}

.mode-btn {
  height: 100%;
  width: 50%;
}

.datetimedisplay {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-size: 8vh;
}

.boost, .settings {
  margin-top: 2px;
}

.center-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  height: 65%;
  min-height: 65%;
  width: 100%;
  padding-bottom: 15px;
}

.left-column-center {
  display: flex;
  flex-direction: column;
  flex: 1;
  width: 56vw;
}

.right-column-center {
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  height: 100%;
}

.show-controls {
  font-size: 25vh;
}

.hide-controls {
  font-size: 40vh;
}

.show-controls > .symbol {
  font-size: 18vh;
  vertical-align: text-top;
}

.temp-controls {
  display: flex;
  flex-direction: row;
  align-content: center;
  justify-content: center;
  align-items: flex-end;
  height: 20%;
  width: 100%;
}

.temp-controls > div {
  border-style: solid;
  border-width: 2px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 15vh;
  width: 35%;
  margin-left: 10px;
  margin-right: 10px;
}

.bottom-container {
  display: flex;
  flex-direction: row;
  padding-left: 10px;
  padding-right: 10px;
}

.power-setting-text {
  width: 50%;
  text-align: left;
}

.comfort-mode {
  width: 50%;
  text-align: right;
}

.flex-row {
  display: flex;
  flex-direction: row;
  height: 10vh;
  margin-right: 5px;
}

.current-weather {
  font-size: 6vh;
  text-align: right;
  margin-right: 10px;
  flex: 1;
}

.current-temp > .symbol {
  margin-right: -4px;
}

.outdoors-temp > .symbol {
  margin-right: -4px;
}

.current-humidity > .symbol {
}

.current-pressure > .symbol {
}

.home-icon {
}

.humidity-icon {
  justify-content: center;
  margin-right: 3px;
}

.pressure-icon {
  margin-right: 3px;
}

.dim {
  height: 100%;
  width: 100%;
  position: fixed;
  left: 0;
  top: 0;
  z-index: 10 !important;
  background-color: black;
  filter: alpha(opacity=75); /* internet explorer */
  -khtml-opacity: 0.75; /* khtml, old safari */
  -moz-opacity: 0.75; /* mozilla, netscape */
  opacity: 0.75; /* fx, safari, opera */
}
</style>
