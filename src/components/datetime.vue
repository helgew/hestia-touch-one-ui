<template>
  <div>
    <div class="date">{{ date }}</div>
    <div class="time">{{ time }}</div>
  </div>
</template>

<script>

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
      var strftime = require('strftime')
      setInterval(() => {
        this.time = strftime("%H:%M:%S")
        this.date = strftime("%a, %b %e, %Y")
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
