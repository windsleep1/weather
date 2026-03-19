<template>
  <div class="app-container">
    <video src="/WeChat_20250315093424.mp4" autoplay muted loop></video>

    <!-- 顶部区域 -->
    <div class="top">
      <div class="container clearfix">
        <h2>全球城市天气预报</h2>
        <span style="color: #fff; position: absolute; left: 400px"
          >在这里,您可以查询到全球各城市的天气信息,希望对您有帮助!</span
        >
        <ul class="nav clearfix">
          <!-- <li>首页</li>
          <li>今日天气</li> -->
          <li @click="showModal = true" style="margin-left: 400px">更多天气</li>
          <!-- <li>新闻资讯</li>
          <li>联系我们</li> -->
        </ul>
        <div class="top-right">
          <img src="/adre.png" alt="定位图标" />
          <span>{{ currentCity }}</span>
          <input type="text" v-model="searchCity" @keyup.enter="handleSearch" />
          <button
            @click="handleSearch"
            style="
              height: 32px;
              position: relative;
              top: 1px;
              border: none;
              background-color: skyblue;
              margin-left: 10px;
              font-size: 14px;
            "
          >
            查询
          </button>
        </div>
      </div>
    </div>

    <!-- 中间区域 -->
    <div class="middle">
      <div class="weather">
        <div class="weather-top">
          <span>{{ currentCity }}今日天气</span>
          <span style="font-size: 14px; font-weight: 600">[刚刚更新]</span>
        </div>
        <div class="weather-main">
          <div class="riqi">
            <span class="date">{{ currentDate }}</span>
            <span class="time">{{ currentTime }}</span>
          </div>
          <div class="now">
            <span class="wd" style="font-size: 30px; font-weight: 600; vertical-align: middle"
              >{{ weatherData.now?.temp }}
            </span>
            <span>℃</span>
            <img
              :src="`/weather-icon-S2/64/${weatherData.now?.icon}.png`"
              alt="天气图标"
              v-if="weatherData.now?.icon"
            />
          </div>
          <div class="wendu">
            <span>{{ dailyWeather[0]?.tempMin }}°C ~ {{ dailyWeather[0]?.tempMax }}°C</span>
          </div>
          <div class="aa1 clearfix">
            <div class="aa1-left">
              <span>天气: &nbsp;</span>
              <span class="tianqi">{{ weatherData.now?.text }}</span>
            </div>
            <div class="aa1-right">
              <span>湿度: &nbsp;</span>
              <span class="shidu">{{ weatherData.now?.humidity }}%</span>
            </div>
          </div>
          <div class="aa1 clearfix">
            <div class="aa1-left">
              <span>风向: &nbsp;</span>
              <span class="fengxiang">{{ weatherData.now?.windDir }}</span>
            </div>
            <div class="aa1-right">
              <span>风力等级: &nbsp;</span>
              <span class="fengli">{{ weatherData.now?.windScale }}级</span>
            </div>
          </div>
          <h3>温馨提示</h3>
          <div class="cyzs">{{ indicesData.daily?.[0]?.text }}</div>
        </div>
      </div>

      <!-- 图表 -->
      <div class="chart" ref="chartRef"></div>
    </div>

    <!-- 底部区域 -->
    <div class="bottom">
      <div class="container" style="text-align: center">
        <span class="copy">数据来源:和风天气</span>
      </div>
    </div>

    <!-- 弹窗 -->
    <div class="tanchuang" v-if="showModal">
      <div class="tanchuang-top">
        <h3 style="text-align: center">{{ currentCity }}-未来七天温度变化</h3>
        <span @click="showModal = false">X</span>
      </div>
      <div class="chart1" ref="sevenDayChartRef"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive, watch, nextTick, onBeforeUnmount } from 'vue'
import * as echarts from 'echarts'
import axios from 'axios'

const currentCity = ref('青岛')
const searchCity = ref('青岛')
const currentDate = ref('')
const currentTime = ref('')
const showModal = ref(false)
const cityId = ref('')

// 天气数据
const weatherData = reactive({
  now: {},
})

const dailyWeather = ref([])
const hourlyWeather = ref([])
const sevenDayWeather = ref([])
const indicesData = reactive({
  daily: [],
})

// 图表引用
const chartRef = ref(null)
const sevenDayChartRef = ref(null)

// 初始化图表
let chartInstance = null
let sevenDayChartInstance = null

// 格式化日期和时间
const formatDateTime = () => {
  const date = new Date()
  // 格式化日期
  currentDate.value = `${date.getFullYear()}年${date.getMonth() + 1}月${date.getDate()}日 星期${
    ['日', '一', '二', '三', '四', '五', '六'][date.getDay()]
  }`

  // 格式化时间
  const h = date.getHours().toString().padStart(2, '0')
  const m = date.getMinutes().toString().padStart(2, '0')
  const s = date.getSeconds().toString().padStart(2, '0')
  currentTime.value = `${h}:${m}:${s}`
}

// 搜索城市处理
const handleSearch = () => {
  if (!searchCity.value.trim()) return

  currentCity.value = searchCity.value

  // 获取城市ID
  axios
    .get('https://geoapi.qweather.com/v2/city/lookup', {
      params: {
        location: searchCity.value,
        key: '374655a3a532453f92c6c61268cc9478',
      },
    })
    .then((res) => {
      if (res.data.location && res.data.location.length > 0) {
        cityId.value = res.data.location[0].id
        localStorage.setItem('city', cityId.value)
        fetchWeatherData()
      }
    })
    .catch((err) => {
      console.error('获取城市信息失败:', err)
    })
}

// 获取天气数据
const fetchWeatherData = () => {
  if (!cityId.value) return

  // 获取实时天气
  axios
    .get('https://devapi.qweather.com/v7/weather/now', {
      params: {
        location: cityId.value,
        key: '374655a3a532453f92c6c61268cc9478',
      },
    })
    .then((res) => {
      weatherData.now = res.data.now || {}
    })
    .catch((err) => {
      console.error('获取实时天气失败:', err)
    })

  // 获取3天预报
  axios
    .get('https://devapi.qweather.com/v7/weather/3d', {
      params: {
        location: cityId.value,
        key: '374655a3a532453f92c6c61268cc9478',
      },
    })
    .then((res) => {
      dailyWeather.value = res.data.daily || []
    })
    .catch((err) => {
      console.error('获取3天预报失败:', err)
    })

  // 获取24小时预报
  axios
    .get('https://devapi.qweather.com/v7/weather/24h', {
      params: {
        location: cityId.value,
        key: '374655a3a532453f92c6c61268cc9478',
      },
    })
    .then((res) => {
      hourlyWeather.value = res.data.hourly || []
      updateHourlyChart()
    })
    .catch((err) => {
      console.error('获取24小时预报失败:', err)
    })

  // 获取生活指数
  axios
    .get('https://devapi.qweather.com/v7/indices/1d', {
      params: {
        location: cityId.value,
        type: 3,
        key: '374655a3a532453f92c6c61268cc9478',
      },
    })
    .then((res) => {
      indicesData.daily = res.data.daily || []
    })
    .catch((err) => {
      console.error('获取生活指数失败:', err)
    })
}

// 更新小时图表
const updateHourlyChart = () => {
  if (!chartRef.value || hourlyWeather.value.length === 0) return

  const temps = hourlyWeather.value.map((item) => item.temp)
  const times = hourlyWeather.value.map((item) => item.fxTime.substring(11, 16))
  const windSpeeds = hourlyWeather.value.map((item) => item.windSpeed)

  if (!chartInstance) {
    chartInstance = echarts.init(chartRef.value)
  }

  const option = {
    title: {
      show: true,
      text: '24小时天气',
      left: 1,
      padding: 15,
      textStyle: { color: '#fff' },
    },
    color: ['#006886b5', '#8e40408c'],
    tooltip: {
      trigger: 'axis',
      formatter: `时间：{b0} <br>温度：{c0} ℃ <br>风速：{c1} km/h`,
      axisPointer: {
        type: 'cross',
        label: { backgroundColor: '#6a7985' },
      },
    },
    xAxis: {
      type: 'category',
      boundaryGap: false,
      data: times,
      axisLine: {
        lineStyle: {
          color: '#fff',
        },
      },
      axisLabel: { color: '#fff' },
    },
    yAxis: {
      type: 'value',
      axisLine: {
        lineStyle: {
          color: '#fff',
        },
      },
      axisLabel: { color: '#fff' },
    },
    series: [
      {
        data: temps,
        type: 'line',
        areaStyle: {},
      },
      {
        data: windSpeeds,
        type: 'line',
        areaStyle: {},
      },
    ],
  }

  chartInstance.setOption(option)
}

// 更新7天图表
const updateSevenDayChart = () => {
  if (!sevenDayChartRef.value || sevenDayWeather.value.length === 0) return

  const avgTemps = sevenDayWeather.value.map((item) =>
    Math.floor((parseInt(item.tempMax) + parseInt(item.tempMin)) / 2)
  )
  const dates = sevenDayWeather.value.map((item) => item.fxDate)

  // 如果图表实例已存在，先销毁再重新创建
  if (sevenDayChartInstance) {
    sevenDayChartInstance.dispose()
    sevenDayChartInstance = null
  }

  sevenDayChartInstance = echarts.init(sevenDayChartRef.value)

  const option = {
    tooltip: {
      trigger: 'axis',
      formatter: `日期：{b0} <br>温度：{c0} ℃ <br>`,
      axisPointer: {
        type: 'cross',
        label: {
          backgroundColor: '#6a7985',
        },
      },
    },
    xAxis: {
      type: 'category',
      data: dates,
      axisLine: {
        lineStyle: {
          color: '#333',
        },
      },
      axisLabel: {
        color: '#333',
      },
    },
    yAxis: {
      type: 'value',
      axisLine: {
        lineStyle: {
          color: '#333',
        },
      },
      axisLabel: {
        color: '#333',
      },
    },
    series: [
      {
        data: avgTemps,
        type: 'line',
        lineStyle: {
          color: '#00c5e6',
          width: 3,
        },
        itemStyle: {
          color: '#00c5e6',
        },
        symbol: 'circle',
        symbolSize: 8,
      },
    ],
  }

  sevenDayChartInstance.setOption(option)
}

// 监听弹窗显示，获取7天天气数据
watch(showModal, (newVal) => {
  if (newVal && cityId.value) {
    axios
      .get('https://devapi.qweather.com/v7/weather/7d', {
        params: {
          location: cityId.value,
          key: '374655a3a532453f92c6c61268cc9478',
        },
      })
      .then((res) => {
        sevenDayWeather.value = res.data.daily || []
        nextTick(() => {
          updateSevenDayChart()
        })
      })
      .catch((err) => {
        console.error('获取7天预报失败:', err)
      })
  } else {
    // 关闭弹窗时销毁图表实例
    if (sevenDayChartInstance) {
      sevenDayChartInstance.dispose()
      sevenDayChartInstance = null
    }
  }
})

// 初始化
onMounted(() => {
  // 设置定时器更新日期时间
  formatDateTime()
  setInterval(formatDateTime, 1000)

  // 从本地存储获取城市ID或初始化
  const savedCityId = localStorage.getItem('city')
  if (savedCityId) {
    cityId.value = savedCityId
  } else {
    // 初始获取青岛的ID
    axios
      .get('https://geoapi.qweather.com/v2/city/lookup', {
        params: {
          location: '青岛',
          key: '374655a3a532453f92c6c61268cc9478',
        },
      })
      .then((res) => {
        if (res.data.location && res.data.location.length > 0) {
          cityId.value = res.data.location[0].id
          localStorage.setItem('city', cityId.value)
          fetchWeatherData()
        }
      })
  }

  // 窗口大小变化时重绘图表
  window.addEventListener('resize', () => {
    if (chartInstance) chartInstance.resize()
    if (sevenDayChartInstance) sevenDayChartInstance.resize()
  })

  // 如果已有城市ID，获取天气数据
  if (cityId.value) {
    fetchWeatherData()
  }
})

// 组件卸载前销毁所有图表实例
onBeforeUnmount(() => {
  if (chartInstance) {
    chartInstance.dispose()
  }
  if (sevenDayChartInstance) {
    sevenDayChartInstance.dispose()
  }
})
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}

li {
  list-style: none;
}

a {
  text-decoration: none;
}

.clearfix::after {
  content: '';
  display: block;
  clear: both;
  height: 0;
  visibility: hidden;
}

video {
  position: fixed;
  width: 100%;
  inset: 0;
  z-index: -1;
}

.app-container {
  width: 100%;
}

.container {
  width: 1200px;
  margin: 0 auto;
}

.top {
  width: 100%;
  height: 70px;
  line-height: 70px;
  background-color: rgba(0, 0, 0, 0.5);
}

.top h2 {
  float: left;
  font-size: 20px;
  color: white;
  font-weight: 500;
}

.nav {
  float: left;
  margin-left: 150px;
}

.nav li {
  width: 90px;
  text-align: center;
  cursor: pointer;
  float: left;
  color: white;
}

.top-right {
  float: right;
}

.top-right img {
  width: 30px;
  height: 30px;
  vertical-align: middle;
}

.top-right span {
  color: white;
  vertical-align: middle;
}

.top-right input {
  height: 30px;
  padding-left: 10px;
  vertical-align: middle;
  outline: none;
}

.top-right button {
  height: 33px;
  background-color: #ccc;
  padding: 0 10px;
  border: none;
  outline: none;
  cursor: pointer;
}

.middle {
  width: 1200px;
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  color: #fff;
}

.weather {
  width: 400px;
  height: 400px;
  border-radius: 20px;
  background-color: rgba(255, 255, 255, 0.4);
  border: 1px solid #00c5e6;
  transition: all 0.5s;
  float: left;
}

.weather:hover {
  transform: scale(1.2);
  background-color: rgba(255, 255, 255, 0.2);
}

.chart:hover {
  transform: scale(1.2);
}

.weather-top {
  background-color: #00c5e6;
  text-align: center;
  height: 60px;
  line-height: 60px;
  border-radius: 20px 20px 0 0;
  font-size: 14px;
}

.weather-top span {
  font-size: 18px;
}

.weather-main .riqi {
  text-align: center;
  padding: 10px 0;
}

.weather-main .now {
  text-align: center;
}

.weather-main .now img {
  width: 60px;
  vertical-align: middle;
}

.wendu {
  text-align: center;
  padding: 10px 0;
}

.wendu span {
  font-weight: 600;
}

.aa1 {
  padding: 5px 20px;
}

.aa1-left {
  float: left;
}

.aa1 .aa1-right {
  float: right;
  width: 120px;
  text-align: left;
}

h3 {
  text-align: center;
}

.cyzs {
  padding: 10px;
  text-align: center;
}

.chart {
  width: 700px;
  height: 400px;
  background-color: rgba(255, 255, 255, 0.3);
  transition: all 0.5s;
  float: right;
}

.bottom {
  width: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  height: 61px;
  position: fixed;
  bottom: 0;
}

.copy {
  color: #fff;
  line-height: 61px;
}

.tanchuang {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 800px;
  height: 500px;
  border-radius: 5px;
  background-color: white;
  z-index: 9999;
  transition: all 0.5s;
}

.tanchuang-top {
  text-align: center;
  padding: 8px;
  position: relative;
}

.tanchuang-top span {
  position: absolute;
  top: 0;
  right: 10px;
  font-size: 30px;
  font-weight: bold;
  cursor: pointer;
}

.chart1 {
  width: 800px;
  height: 460px;
}
</style>
