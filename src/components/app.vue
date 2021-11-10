<template>
  <div id="app" class="color-off">
    <settings-screen v-if="showSettingsScreen" />
    <home-screen v-if="!showSettingsScreen" />
  </div>
</template>

<script>
import 'typeface-roboto'
import homeScreen from './home-screen.vue'
import settingsScreen from './settings-screen.vue'

export default {
  components: {
    homeScreen,
    settingsScreen
  },
  computed: {
    colorClass() {
      const mode = this.$store.state.selectedMode
      if (mode && this.$store.state.modes[mode].active) {
        return {
          [`color-${mode}`]: true
        }
      }
      return {
        'color-off': true
      }
    },
    showSettingsScreen() {
      return this.$store.state.showSettingsScreen
    }
  }
}
</script>

<style>
html * {cursor:none!important}

html {
  -ms-touch-action: manipulation;
  touch-action: manipulation;
  background-color: black;
  overflow: hidden;
}

#app {
  font-family: 'Roboto', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}

/* Hide kweb scroolbar */
#element::-webkit-scrollbar {
  display: none;
}

.color-heat {
  color: #fd684e;
}

.color-off {
  color: #e0e5e8;
}

/* Prevent dragging screen text when tapping */
.unselectable {
  -webkit-user-select: none; /* Safari */
  -moz-user-select: -moz-none; /* Firefox */
  -ms-user-select: none; /* IE */
  user-select: none; /* Standard */
}
</style>
