<template>
  <div id="home-screen">
    <div v-if="screensaver" class="dim" @click="screensaver = false"/>
    <div class="top-container">
      <div class="mode-btn heat"
           :class="{
          animated: modes.heat.running,
          'color-heat': modes.heat.active || modes.heat.running,
          'color-off': !(modes.heat.active || modes.heat.running)
        }"
           @click="toggleHeat">
        <icon-heat size="76%"/>
      </div>
      <div v-if="modes.heat.active"
           class="mode-btn boost color-off"
           :class="{ animated: modes.heat.boostEnabled }"
           @click="toggleBoost"
      >
        <icon-boost size="65%"/>
      </div>
      <div class="datetimedisplay unselectable">
        <DateTimeDisplay/>
      </div>
      <div class="mode-btn settings color-off" @click="toggleSettingsScreen">
        <icon-settings size="45%"/>
      </div>
    </div>
    <div class="grid">
      <!-- row -->
      <div :class="tempClass"
           @click="toggleControls"
           v-html="targetTemperature">
      </div>
      <div class="grid-home-icon unselectable">
        <icon-home size="100%"/>
      </div>
      <div class="current-temp unselectable">
        {{ currentTemperature }}<span class="symbol">°</span>
      </div>
      <!-- row -->
      <div v-if="showControls" class="value-controls unselectable">
        <div class="decrement" @click="decrement">-</div>
        <div class="increment" @click="increment">+</div>
      </div>
      <div class="grid-humidity-icon">
        <icon-humidity size="100%"/>
      </div>
      <div class="current-humidity unselectable">
        {{ currentHumidity }}<span class="symbol">%</span>
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
import { mapState } from 'vuex'
import iconHeat from './icon-heat.vue'
import iconBoost from './icon-boost.vue'
import iconHome from './icon-home.vue'
import iconHumidity from './icon-humidity.vue'
import iconSettings from './icon-settings.vue'
import DateTimeDisplay from "./datetime.vue";
import IconSettings from "@/components/icon-settings";

export default {
  data() {
    return {
      showTempControls   : false,
      tempClass          : 'hide-controls',
      secsControlsShowing: 0,
      screensaver        : false,
      screenAwake        : 0
    }
  }, components: {
    IconSettings,
    DateTimeDisplay,
    iconHeat,
    iconBoost,
    iconHome,
    iconHumidity,
    iconSettings
  }, computed  : {
    // Some variables in $store.state we want to read
    // https://vuex.vuejs.org/guide/state.html#the-mapstate-helper
    ...mapState(
      ['comfortMode', 'currentTemperature', 'currentHumidity', 'icons', 'hysteresis', 'modes',
        'selectedMode', 'showControls', 'showHeating']),
    powerSettingText() {
      const modes = {
        heat: () => 'Heating',
      }
      if (this.selectedMode && modes[this.selectedMode]) {
        const modeState = this.modes[this.selectedMode]
        const modeText = modes[this.selectedMode]()
        if (modeState.boostEnabled) {
          return `${modeText} boost mode, ${modeState.boostTimeRemaining} min. remaining`
        }

        if (modeState.active) {
          return `${modeText} auto`
        }

        return `${modeText} off`
      }
      return ''
    }, targetTemperature() {
      return this.$store.getters.targetTemperature
    }
  }, methods   : {
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
        this.screenAwake = 0
      }
    }, toggleHeat() {
      if (!this.modes.heat.boostEnabled) {
        var state = this.modes.heat.active ? "OFF" : "ON"
        this.$store.commit('selectPowerSetting', { mode: 'heat', powerOption: state } )
      }
    }, toggleBoost() {
      var state = this.modes.heat.boostEnabled ? "ON" : "Boost"
      this.$store.commit('selectPowerSetting', { mode: 'heat', powerOption: state })
    }, resetScreenAwake() {
      this.screenAwake = 0
    }
  }, mounted() {
    this.startTimers()
    document.addEventListener("mouseup", this.resetScreenAwake())
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

.show-controls {
  font-size: 30vh;
  left: 2%;
  line-height: 100%;
  top: 29%;
  width: 50vw;
}

.hide-controls {
  padding-left: 30vh;
  font-size: 40vh;
  left: 2%;
  line-height: 80%;
  top: 29%;
  width: 40vw;
  height: 53vh;
  z-index: 1;
  padding-top: 10vh;
  background: black;
}

.show-controls > .symbol {
  font-size: 18vh;
  vertical-align: text-top;
}

.bottom-container {
  bottom: 0;
  height: 8vh;
  left: 4%;
  position: absolute;
  text-align: left;
  width: 100%;
}

.bottom-container > .power-setting-text {
  display: inline;
  position: relative;
}

.bottom-container > .comfort-mode {
  display: inline;
  float: right;
  position: relative;
  right: 6%;
}

.current-temp {
  font-size: 24vh;
  right: 8%;
  text-align: right;
  top: 30%;
  width: 40%;
}

.current-temp > .symbol {
  font-size: 14vh;
  vertical-align: text-top;
}

.current-humidity {
  bottom: 8%;
  font-size: 24vh;
  right: 6.5%;
  text-align: right;
  width: 40%;
}

.current-humidity > .symbol {
  font-size: 10vh;
}

.grid {
  height: 100%;
  width: 100%;
}

.grid > div {
  position: absolute;
}

.grid-home-icon {
  height: 6vw;
  width: 6vw;
  right: 0.5%;
  bottom: 51%;
}

.grid-humidity-icon {
  bottom: 20%;
  height: 5vw;
  right: 1vw;
  width: 5vw;
}

.mode-btn {
  cursor: pointer;
  display: inline-block;
  float: left;
  height: 100%;
  margin-top: 4vh;
  width: 26vh;
}

.mode-btn.heat {
  margin-top: 4.4vh;
}

.datetimedisplay {
  display: flex;
  justify-content: center;
  position: absolute;
  left: 50%;
  top: 50%;
  width: 200px;
  height: 50px;
  margin-top: -20px;
  margin-left: -100px;
}

.mode-btn.settings {
  position: absolute;
  top: 2vh;
  right: 0.5%;
}

.mode-btn.boost {
  margin-top: 6vh;
}

.top-container {
  color: #e0e5e8;
  height: 25%;
  left: 0;
  max-height: 25%;
  position: absolute;
  top: 0;
  width: 100%;
}

.value-controls {
  bottom: 0;
  height: 50vh;
  width: 50vw;
  left: 2%;
}

.value-controls > div {
  border-style: solid;
  border-width: 2px;
  border-radius: 6px;
  bottom: 20%;
  cursor: pointer;
  height: 24vh;
  font-size: 20vh;
  display: inline-block;
  text-align: center;
  width: 46%;
}

.value-controls > .decrement {
  position: absolute;
  left: 0;
}

.value-controls > .increment {
  position: absolute;
  right: 0;
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
