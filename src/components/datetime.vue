<template>
  <div>
    <div class="date">{{ date }}</div>
    <div class="time">{{ time }}</div>
  </div>
</template>

<script>

const { DateTime } = require("luxon")

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
    setTime() {
      setInterval(() => {
        const now = DateTime.now()
        this.time = now.toLocaleString(DateTime.TIME_24_WITH_SECONDS)
        this.date = now.toLocaleString(DateTime.DATE_MED_WITH_WEEKDAY)
      }, 1000)
    }
  }, mounted() {
    this.setTime()
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
