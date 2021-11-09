<template>
  <div id="home-screen">
    <div v-if="screensaver" class="dim" @click="screensaver = false" />
    <power-settings-modal
      :mode="lastTappedMode"
      :on-option-select="powerModalCallback"
      v-if="showPowerModal"
    />
    <div class="top-container">
      <div v-if="showHeating"
           class="mode-btn heat"
           :class="{
          animated: modes.heat.running || modes.heat2.running,
          'color-heat': (selectedMode || lastTappedMode) === 'heat'
                            && (modes.heat.active || modes.heat.boostEnabled),
          'color-off': (selectedMode || lastTappedMode) !== 'heat'
        }"
           @click="openPowerModal('heat')">
        <!-- Switch out icon for 2nd-stage heating -->
        <icon-heat size="76%" v-if="!modes.heat2.running"/>
        <icon-heat2 size="76%" v-if="modes.heat2.running"/>
      </div>
      <div v-if="showCooling"
           class="mode-btn cool"
           :class="{
          animated: modes.cool.running,
          'color-cool': (selectedMode || lastTappedMode) === 'cool',
          'color-off': (selectedMode || lastTappedMode) !== 'cool'
        }"
           @click="openPowerModal('cool')">
        <icon-cool size="80%"/>
      </div>
      <div v-if="showFan"
           class="mode-btn fan"
           :class="{
          animated: modes.fan.running,
          'color-fan': (selectedMode || lastTappedMode) === 'fan',
          'color-off': (selectedMode || lastTappedMode) !== 'fan'
        }"
           @click="openPowerModal('fan')">
        <icon-fan size="82%"/>
      </div>
      <div v-if="showHumidity"
           class="mode-btn humidity"
           :class="{
          animated: modes.humidity.running,
          'color-humidity': (selectedMode || lastTappedMode) === 'humidity',
          'color-off': (selectedMode || lastTappedMode) !== 'humidity'
        }"
           @click="openPowerModal('humidity')">
        <icon-humidity size="82%"/>
      </div>
      <div v-if="showHotWater"
           class="mode-btn hotwater"
           :class="{
          animated: modes.hotwater.running,
          'color-hotwater': (selectedMode || lastTappedMode) === 'hotwater',
          'color-off': (selectedMode || lastTappedMode) !== 'hotwater'
        }"
           @click="openPowerModal('hotwater')">
        <icon-hotwater size="70%"/>
      </div>
      <div class="mode-btn">&nbsp;</div>
      <div class="mode-btn datetimedisplay unselectable" style="color: #999999">
        <DateTimeDisplay/>
      </div>
      <div class="mode-btn info color-off" @click="toggleInfoScreen">
        <icon-info size="75%"/>
      </div>
    </div>
    <div class="grid">
      <!-- row -->
      <div :class="tempClass"
           @click="toggleTempClass"
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
import powerSettingsModal from './power-settings-modal.vue'
import iconCool from './icon-cool.vue'
import iconHeat from './icon-heat.vue'
import iconHeat2 from './icon-heat2.vue'
import iconHome from './icon-home.vue'
import iconFan from './icon-fan.vue'
import iconHumidity from './icon-humidity.vue'
import iconHotwater from './icon-hotwater.vue'
import iconInfo from './icon-info.vue'
import DateTimeDisplay from "./datetime.vue";

export default {
  data() {
    return {
      // Called by the power settings modal when you tap on a mode
      powerModalCallback: () => {
      }, // Give the power settings modal context of what options to show
      lastTappedMode: '', showPowerModal: false, sleeping: true, tempClass: 'sleep-temp',
      secondsAwake: 0, screensaver: false, screenAwake: 0
    }
  }, components: {
    DateTimeDisplay,
    iconCool,
    iconFan,
    iconHeat,
    iconHeat2,
    iconHome,
    iconHumidity,
    iconHotwater,
    iconInfo,
    powerSettingsModal
  }, computed  : {
    // Some variables in $store.state we want to read
    // https://vuex.vuejs.org/guide/state.html#the-mapstate-helper
    ...mapState(
      ['comfortMode', 'currentTemperature', 'currentHumidity', 'icons', 'hysteresis', 'modes',
        'selectedMode', 'showControls', 'showCooling', 'showFan', 'showHeating', 'showHotWater',
        'showHumidity']), powerSettingText() {
      const modes = {
        cool    : () => 'Cooling',
        heat    : () => this.modes.heat2.running ? '2nd-stage heating' : 'Heating',
        hotwater: () => 'Hot water',
        humidity: () => 'Humidity control',
        fan     : () => 'Fan'
      }
      if (this.selectedMode && modes[this.selectedMode]) {
        const modeState = this.modes[this.selectedMode]
        const modeText = modes[this.selectedMode]()
        if (modeState.boostEnabled) {
          return `${modeText} boost mode, ${modeState.boostTimeRemaining} min. remaining`
        }

        if ((modeText == 'Fan') && ((modeState.active) || (modeState.running))) {
          return `${modeText} on`
        } else {
          if (modeState.active) {
            return `${modeText} auto`
          }
        }

        return `${modeText} off`
      }
      return ''
    }, targetTemperature() {
      return this.$store.getters.targetTemperature
    }
  }, methods   : {
    openPowerModal(mode) {
      // First tap, only select
      if (this.selectedMode !== mode) {
        this.$store.commit('selectMode', mode)
        return
      }

      // Second tap, open the modal
      this.lastTappedMode = mode
      this.powerModalCallback = powerOption => {
        this.$store.commit('selectPowerSetting', { mode, powerOption })
        this.lastTappedMode = ''
        this.showPowerModal = false
      }
      this.showPowerModal = true
    }, decrement() {
      this.secondsAwake = 0
      this.$store.commit('decrementTargetValue')
    }, increment() {
      this.secondsAwake = 0
      this.$store.commit('incrementTargetValue')
    }, toggleInfoScreen() {
      this.$store.commit('toggleInfoScreen')
    }, startTimers() {
      this.sleepTimer = setInterval(() => {
        if (!this.sleeping && this.secondsAwake > 9) {
          this.secondsAwake = 0
          this.sleeping = true
          this.tempClass = 'sleep-temp'
        } else if (!this.sleeping) {
          this.secondsAwake += 1
        }
      }, 1000)
      this.screenTimer = setInterval(() => {
        if (!this.screensaver && this.screenAwake > 29) {
          this.screenAwake = 0
          this.screensaver = true
        } else if (!this.screensaver && this.sleeping) {
          this.screenAwake += 1
        }
      }, 1000)
    }, toggleTempClass() {
      if (this.sleeping) {
        this.sleeping = false
        this.tempClass = 'active-temp unselectable'
        this.screenAwake = 0
      }
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

.active-temp {
  font-size: 30vh;
  left: 2%;
  line-height: 100%;
  top: 29%;
  width: 50vw;
}

.sleep-temp {
  padding-left: 210px;
  font-size: 40vh;
  left: 2%;
  line-height: 80%;
  top: 29%;
  width: 40vw;
  height: 310px;
  z-index: 1;
  padding-top: 100px;
  background: black;
}

.active-temp > .symbol {
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

.mode-btn.cool {
  margin-top: 3vh;
}

.mode-btn.heat {
  margin-top: 4.4vh;
}

.mode-btn.datetimedisplay {
  margin-top: 5vh;
  width: 40vh;
}

.mode-btn.humidity {
  margin-top: 2.8vh;
}

.mode-btn.hotwater {
  margin-top: 4.4vh;
}

.mode-btn.info {
  position: absolute;
  right: 0.5%;
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
  height:100%;
  width:100%;
  position:fixed;
  left:0;
  top:0;
  z-index:10 !important;
  background-color:black;
  filter: alpha(opacity=75); /* internet explorer */
  -khtml-opacity: 0.75;      /* khtml, old safari */
  -moz-opacity: 0.75;      /* mozilla, netscape */
  opacity: 0.75;      /* fx, safari, opera */
}
</style>
