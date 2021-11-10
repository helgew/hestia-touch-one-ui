<template>
  <div class="container color-off">
    <info-screen v-if="showInfoScreen"/>
    <div class="info" @click="toggleInfoScreen">
      <icon-info size="75%"/>
    </div>
    <div class="close" @click="toggleSettingsScreen">
      <icon-close size="74%"/>
    </div>
  </div>
</template>

<script>
import iconClose from './icon-close.vue'
import iconInfo from './icon-info.vue'
import infoScreen from './info-screen.vue'

export default {
  components : {
    iconClose, infoScreen, iconInfo
  }, computed: {
    showInfoScreen() {
      return this.$store.state.showInfoScreen
    }
  }, methods : {
    toggleSettingsScreen() {
      this.$store.commit('toggleSettingsScreen')
      if (this.showInfoScreen) {
        this.$store.commit('toggleInfoScreen')
      }
    }, toggleInfoScreen() {
      this.$store.commit('toggleInfoScreen')
    }
  }
}
</script>

<style scoped>
.close {
  cursor: pointer;
  float: right;
  height: 26vh;
  margin-right: 1vh;
  margin-top: 3.9vh;
  position: absolute;
  right: 0;
  top: 0;
  width: 26vh;
}

.container {
  height: 100%;
  left: 0;
  position: absolute;
  top: 0;
  width: 100%;
}

.info {
  position: absolute;
  right: 0.5%;
  bottom: 3%;
  display: inline-block;
  width: 26vh;
}

</style>
