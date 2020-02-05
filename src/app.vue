<template>
  <div id="app">
    <!-- <div class="header">
      <h1>合肥新冠状病毒感染确诊病例</h1>
      <p>数据来源于 <a target="_blank" href="http://wjw.hefei.gov.cn/ztzl/xxgzbdgrdfyyqfk/xxfb/17723019.html">合肥卫健委</a> ，仅供参考，不喜勿喷</p>
    </div> -->
    <div class="fixed-link">
      <a @click="switchShowMode('marker')">
        <img src="./assets/icon-marker.png">
        <span>小区明细</span>
      </a>
      <a @click="switchShowMode('heat')">
        <img src="./assets/icon-heat.png">
        <span>热力图</span>
      </a>
      <a @click="getCurrentLocation" class="icon-location">
        <img src="./assets/icon-location.png">
        <span>我的位置</span>
      </a>
      <a href="https://github.com/twoer/2019-nCoV-HF">
        <img src="./assets/icon-github.png">
        <span>源码</span>
      </a>
    </div>
    <div class="fixed-info">数据来源于 <a target="_blank" href="http://wjw.hefei.gov.cn/ztzl/xxgzbdgrdfyyqfk/xxfb/17723019.html">合肥卫健委</a> ，更新时间：2020/02/04 20:00</div>
    <div id="container"></div>
  </div>
</template>

<script>
import dataCase from './data.js'
const BMap = window['BMap']
const BMapLib = window['BMapLib']
const isSupportCanvas = () => {
  let elem = document.createElement('canvas')
  return !!(elem.getContext && elem.getContext('2d'))
}
export default {
  name: 'app',
  data () {
    return {
      map: null,
      showMode: 'marker',
      currentPoint: null,
      markerList: [],
      heatmapOverlay: null
    }
  },
  mounted () {
    this.initMap()
    this.initMarker()
    this.initHeat()
  },
  methods: {
    initMap () {
      if (!BMap) {
        return
      }
      this.map = new BMap.Map('container')
      let point = new BMap.Point(117.233675, 31.827828)
      this.map.centerAndZoom(point, 12)
      this.map.enableScrollWheelZoom()
    },
    initMarker () {
      dataCase.forEach((area) => {
        area.list.forEach((item) => {
          let temp = item.split(',')
          let point = new BMap.Point(temp[0], temp[1])
          let label = new BMap.Label(temp[2], {
            offset: new BMap.Size(5, -20)
          })
          label.setStyle({ padding: '2px 3px', lineHeight: '1em', color: '#dc6450', fontSize: '11px', backgroundColor: 'rgba(255,255,255, 0.8)', borderColor: 'rgba(220,100,80,0.5)' })
          let icon = new BMap.Icon('./img/marker.png', new BMap.Size(20, 20), {
            imageSize: new BMap.Size(20, 20)
          })
          let marker = new BMap.Marker(point, {
            icon
          })
          marker.setLabel(label)
          this.map.addOverlay(marker)
          this.markerList.push(marker)
        })
      })
    },
    initHeat () {
      if (!isSupportCanvas()) {
        alert('热力图目前只支持有canvas支持的浏览器,您所使用的浏览器不能使用热力图功能~')
        return
      }
      let points = [
      ]
      dataCase.forEach((area) => {
        area.list.forEach((item) => {
          let temp = item.split(',')
          points.push({
            'lng': temp[0], 'lat': temp[1], 'count': 10
          })
        })
      })
      this.heatmapOverlay = new BMapLib.HeatmapOverlay({ 'radius': 20,
        gradient: {
          0.2: 'rgb(0, 255, 255)',
          0.5: 'rgb(0, 110, 255)',
          0.8: 'rgb(100, 0, 255)'
        } })
      this.map.addOverlay(this.heatmapOverlay)
      this.heatmapOverlay.setDataSet({ data: points, max: 13 })
      if (this.showMode !== 'heat') {
        this.heatmapOverlay.hide()
      }
    },
    switchShowMode (mode) {
      this.showMode = mode
      if (mode === 'heat') {
        this.heatmapOverlay.show()
        this.markerList.forEach((marker) => {
          marker.hide()
        })
      } else if (mode === 'marker') {
        this.heatmapOverlay.hide()
        this.markerList.forEach((marker) => {
          marker.show()
        })
      }
    },
    getCurrentLocation () {
      let geolocation = new BMap.Geolocation()
      geolocation.getCurrentPosition((r) => {
        if (!r.point) {
          alert('获取位置失败，请重试')
          return
        }
        if (this.currentPoint) {
          this.currentPoint.remove()
        }
        let icon = new BMap.Icon('./img/point.png', new BMap.Size(30, 30), {
          imageSize: new BMap.Size(30, 30)
        })
        this.currentPoint = new BMap.Marker(r.point, { icon })
        this.map.addOverlay(this.currentPoint)
        // this.map.panTo(r.point)
        this.map.centerAndZoom(r.point, 15)
      }, { enableHighAccuracy: true })
    }
  }
}
</script>

<style lang="scss">
html,body,#app,#container
{
  flex: 1;
  display: flex;
  flex-direction: column;
  margin: 0px;
  height: 100%;
}
.header
{
  flex: 0 0 50px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  h1,p
  {
    margin: 0px;
    line-height: 1.5em;
    text-align: center;
  }
  h1
  {
    font-size: 16px;
    color: #333;
  }
  p
  {
    font-size: 12px;
    color: #777;
    a
    {
      color: #576b95;
    }
  }
}
.fixed-link
{
  position: fixed;
  bottom: 8px;
  right: 8px;
  width: 50px;
  z-index: 9999;
  padding: 5px 0px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  background-color: #fff;
  border-radius: 4px;
  a
  {
    display: block;
    position: relative;
    padding: 5px 0px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-decoration: none;
    img
    {
      max-width: 28px;
      max-height: 28px;
      border-radius: 50%;
    }
    span
    {
      margin-top: 5px;
      font-size: 11px;
      color: #555;
      letter-spacing: -0.5px;
    }
    // &.icon-location
    // {
    //   img
    //   {
    //     max-width: 34px;
    //     max-height: 34px;
    //   }
    // }
  }
  a + a
  {
    &::before
    {
      content: '';
      position: absolute;
      top: 0px;
      left: 50%;
      width: 80%;
      height: 1px;
      background-color: rgba(#ddd, 0.7);
      transform: translateX(-50%) scaleY(0.5);
    }
  }
}
.fixed-info
{
  position: fixed;
  bottom: 8px;
  left: 8px;
  z-index: 9999;
  padding: 0.2em 0.4em;
  color: #666;
  background-color: #fff;
  border-radius: 4px;
  // transform: translateX(-50%);
  white-space: nowrap;
  font-size: 11px;
  a
  {
    color: #576b95;
  }
}
</style>
