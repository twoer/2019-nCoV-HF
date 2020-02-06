<template>
  <div id="app">
    <div class="fixed-header">
      <template v-if="DATA_STATISTICS && DATA_STATISTICS.update && DATA_STATISTICS.count">
        <div class="item">
          <span class="last"><label>较昨日</label><b>+{{DATA_STATISTICS.count.newCommunity}}</b></span>
          <cite class="current"><label>确诊小区</label><b>{{DATA_STATISTICS.count.community}}</b></cite>
        </div>
        <div class="item">
          <span class="last"><label>较昨日</label><b>+{{DATA_STATISTICS.count.newPeople}}</b></span>
          <cite class="current"><label>确诊人数</label><b>{{DATA_STATISTICS.count.people}}</b></cite>
        </div>
        <a class="item item-more" target="_blank" :href="DATA_STATISTICS.update.url" @click="showDetail">
          <cite>查看详细</cite>
          <span>{{DATA_STATISTICS.update.date}}</span>
        </a>
      </template>
    </div>
    <div class="fixed-link">
      <a @click="switchShowMode('marker')">
        <img src="./assets/icon-marker.png">
        <span>小区分布</span>
      </a>
      <a @click="switchShowMode('heat')">
        <img src="./assets/icon-heat.png">
        <span>热力图</span>
      </a>
      <a @click="getCurrentLocation" class="icon-location">
        <img src="./assets/icon-location.png">
        <span>我的位置</span>
      </a>
      <a @click="goGithub" target="_blank" href="https://github.com/twoer/2019-nCoV-HF">
        <img src="./assets/icon-github.png">
        <span>源码</span>
      </a>
    </div>
    <!-- <div v-if="DATA_STATISTICS && DATA_STATISTICS.update" class="fixed-info">
        数据来源于 <a target="_blank" :href="DATA_STATISTICS.update.url">{{DATA_STATISTICS.update.author}}</a>
        &nbsp;&nbsp;更新时间：{{DATA_STATISTICS.update.date}}
    </div> -->
    <div id="container"></div>
  </div>
</template>

<script>
import { DATA_CASE, DATA_STATISTICS } from './data.js'
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
      heatmapOverlay: null,
      DATA_STATISTICS: DATA_STATISTICS
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
      DATA_CASE.forEach((area) => {
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
      DATA_CASE.forEach((area) => {
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
      window['_hmt'] && window['_hmt'].push(['_trackEvent', '模式切换', mode])
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
        this.map.centerAndZoom(r.point, 15)
      }, { enableHighAccuracy: true })
      window['_hmt'] && window['_hmt'].push(['_trackEvent', '我的位置', ''])
    },
    goGithub () {
      window['_hmt'] && window['_hmt'].push(['_trackEvent', '查看源码', ''])
    },
    showDetail () {
      window['_hmt'] && window['_hmt'].push(['_trackEvent', '查看详细', ''])
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
.fixed-header
{
  position: fixed;
  top: 8px;
  left: 8px;
  right: 8px;
  z-index: 999;
  height: 50px;
  background-color: #fff;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  box-shadow: 1px 1px 10px rgba(0, 0,0,.1), 1px 1px 2px rgba(0, 0,0,.1);
  .item
  {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    height: 100%;
    align-items: center;
    position: relative;
    .last
    {
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      height: 16px;
      font-size: 10px;
      label
      {
        margin-right: 4px;
        color: #ddd;
      }
      b
      {
        color: rgb(247, 76, 49);
      }
    }
    .current
    {
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      height: 20px;
      font-weight: normal;
      font-style: normal;
      font-size: 12px;
      label
      {
        margin-right: 4px;
        font-size: 10px;
        color: #888;
      }
      b
      {
        line-height: 18px;
        color: rgb(247, 76, 49);
        font-size: 18px;
      }
    }
  }
  .item:first-child + .item
  {
    &::before
    {
      content: '';
      position: absolute;
      top: 15%;
      left: 0px;
      width: 1px;
      height: 70%;
      background-color: rgba(#ddd, 0.7);
    }
  }
  .item-more
  {
    flex: 0 0 35%;
    margin: 0px;
    padding: 0px;
    background-color: #dc6450;
    text-decoration: none;
    span
    {
      font-size: 10px;
      color: #eeb4aa;
    }
    cite
    {
      font-size: 14px;
      font-style: normal;
      color: #fff;
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
  box-shadow: 1px 1px 10px rgba(0, 0,0,.1), 1px 1px 2px rgba(0, 0,0,.1);
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
    transition: all 0.3s;
    &:active
    {
      background-color: rgba(#999, 0.2);
    }
    img
    {
      max-width: 28px;
      max-height: 28px;
    }
    span
    {
      margin-top: 2px;
      font-size: 11px;
      color: #666;
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
