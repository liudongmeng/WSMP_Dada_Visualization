<template>
  <div>
    <!-- <h1>这里是第一集</h1>
    <div class="title">Information</div>
    <div class="items">
      <div class="item">
        <div class="name">Path:</div>
        <div class="value">{{ path }}</div>
      </div>
      <div class="item">
        <div class="name">Route Name:</div>
        <div class="value">{{ name }}</div>
      </div>
      <div class="item">
        <div class="name">Vue.js:</div>
        <div class="value">{{ vue }}</div>
      </div>
      <div class="item">
        <div class="name">Electron:</div>
        <div class="value">{{ electron }}</div>
      </div>
      <div class="item">
        <div class="name">Node:</div>
        <div class="value">{{ node }}</div>
      </div>
      <div class="item">
        <div class="name">Platform:</div>
        <div class="value">{{ platform }}</div>
      </div>
    </div> -->
    <div>
      <p>开始时间</p>
      <input id="start" type="datetime-local" v-model="startTime" v-on:change="log('开始时间:',startTime)">
    </div>
    <div>
      <p>结束时间</p>
      <input id="end" type="datetime-local" v-model="endTime" v-on:change="log('结束时间:',startTime)">
    </div>
    <div>
      <input type="text" v-model="scadaApiUrl">
    </div>
    <div>
      <input type="text" v-model="scadaIdList">
    </div>
    <button v-on:click="add">测试按钮</button>
    <button v-on:click="redis_get">测试RedisClient</button>
    <button v-on:click="post">测试Post</button>
    <button v-on:click="clearChart">清除曲线</button>
    <div id="chartContainer"></div>
  </div>
</template>

<script>
import HighStock from 'highcharts/highstock'
import './highcharts-zh_CN.js'
// require('./highcharts-zh_CN.js')
import _ from 'lodash'
// import 'highcharts/css/highcharts.css'
import redis from 'redis'
import moment from 'moment'
// import http from 'http'
import request from 'request'
// console.log(moment('2017-01-01 00:00:00').format('YYYY-MM-DDThh:mm:ss'))
export default {
  data() {
    return {
      electron: process.versions['atom-shell'],
      name: this.$route.name,
      node: process.versions.node,
      path: this.$route.path,
      platform: require('os').platform(),
      vue: require('vue/package.json').version,
      startTime: moment('2017-01-01 00:00:00').format('YYYY-MM-DDThh:mm'),
      endTime: moment('2019-04-02 00:00:00').format('YYYY-MM-DDThh:mm'),
      scadaIdList: '911',
      scadaApiUrl: 'http://128.1.10.10:28803/Api/wsmp/v1/scadaQuery/lines',
      chartOptions: {
        chart: {
          type: 'line',
          zoomType: 'x',
          animation: true
        },
        rangeSelector: {
          selected: 2
        },
        title: {
          text: '曲线查询'
        },
        plotOptions: {
          series: {
            showInLegend: true
          }
        },
        tooltip: {
          split: false,
          shared: true
        },
        // credits: {
        //   enabled: false
        // },
        // title: {
        //   text: 'Test Chart'
        // },
        // colors: ['#058DC7', '#50B432', '#ED561B', '#DDDF00', '#24CBE5'],
        // xAxis: {
        //   categories: ['1号', '2号', '3号', '3号', '3号']
        // },
        // yAxis: {
        //   title: {
        //     text: '查询结果'
        //   }
        // },
        // plotOptions: {
        //   column: {
        //     colorByPoint: true
        //   }
        // },
        series: []
      },
      chart: null,
      redis_client: redis.createClient({ host: '128.1.3.93', port: 6379 })
    }
  },
  mounted() {
    debugger
    this.chart = HighStock.stockChart('chartContainer', this.chartOptions)
  },
  methods: {
    add() {
      this.chart.series[0].addPoint(Math.random() * 4000)
    },
    redis_get() {
      this.redis_client.keys('*', function(err, reply) {
        console.log('redis:', err, reply)
      })
    },
    log() {
      console.log(arguments)
    },
    queryScada() {
      _.forEach(this.scadaIdList, function(id) {})
    },
    post(id) {
      // var options = {
      //   hostname: '128.1.10.10',
      //   port: 28803,
      //   path: '/Api/wsmp/v1/scadaQuery/lines',
      //   method: 'POST',
      //   headers: {
      //     'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
      //   }
      // }
      let start = moment(this.startTime).format('YYYY-MM-DDThh:mm:ss')
      let end = moment(this.endTime).format('YYYY-MM-DDThh:mm:ss')
      var condition = {
        ObjectIds: _.map(this.scadaIdList.split(','), function(scada) {
          return Number(scada)
        }),
        StartTime: start,
        EndTime: end,
        Step: 1
      }
      condition.ObjectIds = _.map(this.scadaIdList.split(','), function(scada) {
        return Number(scada)
      })
      // options.body = condition
      // options.data = condition
      debugger
      var chart = this.chart
      request.post(
        {
          url: this.scadaApiUrl,
          json: condition,
          headers: {
            'Content-Type': 'application/json',
            Accept: 'application/json'
          }
        },
        function(err, resp, c) {
          // debugger
          if (err) {
            alert(err)
          } else {
            _.forEach(resp.body.Data, function(data) {
              chart.addSeries({
                type: 'line',
                name: `#${data[0].Id}`,
                data: _.map(data, function(d) {
                  // return { value: d.Values[0], time: d.Time }
                  return [new Date(d.Time).valueOf(), d.Values[0]]
                })
              })
            })
          }
        }
      )
      // var req = http.request(options, function(res) {
      //   console.log('STATUS: ' + res.statusCode)
      //   console.log('HEADERS: ' + JSON.stringify(res.headers))
      //   res.setEncoding('utf8')
      //   res.on('data', function(chunk) {
      //     console.log('BODY: ' + chunk)
      //     // JSON.parse(chunk)
      //   })
      // })

      // req.once('success', function(a, b, c) {
      //   console.log('success:', a, b, c)
      // })
    },
    clearChart() {
      // _.forEach(this.chart.series, function(series) {
      //   series.remove(true)
      //   console.log(`清除成功`)
      // })
      while (this.chart.series.length > 0) {
        this.chart.series[0].remove(true)
      }
    }
  }
}
var test = _.debounce(function() {
  console.log('')
}, 5000)
test()

// require('highcharts/modules/stock')(Highcharts)
</script>

<style scoped>
.title {
  color: #888;
  font-size: 18px;
  font-weight: initial;
  letter-spacing: 0.25px;
  margin-top: 10px;
}

.items {
  margin-top: 8px;
}

.item {
  display: flex;
  margin-bottom: 6px;
}

.item .name {
  color: #6a6a6a;
  margin-right: 6px;
}

.item .value {
  color: #35495e;
  font-weight: bold;
}
</style>
