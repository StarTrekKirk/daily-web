<template>
  <div id="app">
    <!--日期-->
    <div>日期：{{date}}</div>
    <!--持续时间-->
    <div>上班时长：{{duration}}</div>
    <i-button type="primary" @click="signin"><p>{{btnCaption}}</p><p>{{current.toTimeString().split(' ')[0]}}</p> </i-button>
    <!--上班时间-->
    <div>上班时间：{{startTime}}</div>
    <!--下班时间-->
    <div>下班时间：{{endTime}}</div>
  </div>
</template>
<style>
  #app{
    font-size:16px;
  }
  #app div{
    padding: 5px;
  }
</style>
<script>

export default {
  name: 'App',
  data: function () {
    return {
      date: new Date().toLocaleDateString(),
      noon: new Date(new Date().toLocaleDateString() + ' 12:00:00'),
      oneoclock: new Date(new Date().toLocaleDateString() + ' 13:00:00'),
      start: null,
      end: null,
      duration: 0,
      current: new Date(),
      record: {}
    }
  },
  computed: {
    btnCaption: function () {
      return this.start == null ? '上班' : '下班'
    },
    startTime: function () {
      if (this.start) {
        return new Date(this.start).toTimeString().split(' ')[0]
      }
    },
    endTime: function () {
      if (this.end) {
        return new Date(this.end).toTimeString().split(' ')[0]
      }
    }
  },
  mounted: function () {
    setInterval(this.calcDuration, 1000)
    setInterval(() => {
      this.current = new Date()
    }, 1000)
    this.$axios.get('/work/record').then(res => {
      this.recordR(res.data.result)
      this.calcDuration()
    })
  },
  methods: {
    recordR: function (result) {
      this.record = result
      this.start = result.startWork
      this.end = result.offWork
    },
    calcDuration: function () {
      let start = this.start
      if (start == null) {
        this.duration = 0
      }
      let rest = 0
      let now = new Date()
      if (now > this.noon) {
        if (now < this.oneoclock) {
          rest = now - this.noon
        } else {
          rest = this.oneoclock - this.noon
        }
      }
      this.duration = this.formatDuring(new Date() - new Date(start) - rest)
    },
    format: function (time) {
      time = time + ''
      if (time.length === 1) {
        time = '0' + time
      }
      return time
    },
    formatDuring: function (mss) {
      // var days = parseInt(mss / (1000 * 60 * 60 * 24))
      var hours = parseInt((mss % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60))
      var minutes = parseInt((mss % (1000 * 60 * 60)) / (1000 * 60))
      var seconds = Math.floor((mss % (1000 * 60)) / 1000)
      return this.format(hours) + ':' + this.format(minutes) + ':' + this.format(seconds)
    },
    signin: function () {
      if (this.start == null) {
        this.$axios.get('/work/startWork').then(res => {
          this.recordR(res.data.result)
        }).catch(error => {
          console.log(error)
        })
      } else {
        this.$axios.get('/work/offWork?id=' + this.record.id).then(res => {
          this.recordR(res.data.result)
        }).catch(error => {
          console.log(error)
        })
      }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
