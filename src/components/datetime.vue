<template>
  <div>
    <div class="date">{{ date }}</div>
    <div class="time">{{ time }}</div>
  </div>
</template>

<script>

import strftime from "strftime";

export default {
  name      : 'DateTimeDisplay', props: {
    color: {
      type: String, default: 'white'
    }
  }, data() {
    return {
      date: "", time: ""
    }
  }, methods: {
    startClock() {
      setInterval(() => {
        this.setTime()
      }, 1000)
    }, setTime() {
      this.time = strftime("%H:%M:%S")
      this.date = strftime("%a, %b %e, %Y")
    }
  }, mounted() {
    this.setTime()
    this.startClock()
  }
}

</script>

<style>
.time {
  font-size: 8vh;
  display: flex;
  justify-content: center;
}

.date {
  font-size: 4vh;
  display: flex;
  justify-content: center;
}
</style>
