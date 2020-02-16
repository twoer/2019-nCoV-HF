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
        <a class="item item-more" @click="showDetail">
          <cite>查看更多</cite>
          <span>{{DATA_STATISTICS.update.date}}</span>
        </a>
      </template>
    </div>
    <div class="fixed-link">
      <a @click="switchShowMode('marker')">
        <img src="./assets/img/icon-marker.png">
        <span>小区分布</span>
      </a>
      <a @click="switchShowMode('heat')">
        <img src="./assets/img/icon-heat.png">
        <span>热力图</span>
      </a>
      <a @click="getCurrentLocation" class="icon-location">
        <img src="./assets/img/icon-location.png">
        <span>我的位置</span>
      </a>
      <a @click="goGithub" target="_blank" href="https://github.com/twoer/2019-nCoV-HF">
        <img src="./assets/img/icon-github.png">
        <span>源码</span>
      </a>
    </div>
    <modal ref="modalDetail" title="详细数据" class="modal-detail">
      <div v-if="!currentPoint" class="tip">如您想查看与确诊小区的距离，请先点击 <a @click="getCurrentLocation" >[获取当前位置]</a> </div>
      <div class="area-group">
        <div class="slot-wrapper">
          <dl v-for="area in dataCase" :key="area.key" :class="['area-list']">
            <dt class="name">
              {{area.key}}<span>（确诊小区<b>{{area.count}}</b>个，确诊<b>{{area.peopleCount}}</b>例，出院<b>{{area.peopleDischarge}}</b>例）</span>
            </dt>
            <ul class="community-list">
              <li v-for="(item, index) in area.list" :key="index" class="item">
                <label>{{item.name}}<span>（确诊<b>{{item.count}}</b>例，出院<b>{{item.discharge}}</b>例）</span></label>
                <span v-if="item.distance" >距您 {{item.distance}} km</span>
              </li>
            </ul>
          </dl>
          <div class="source-info">
            数据来源于 <a target="_blank" :href="DATA_STATISTICS.update.url">{{DATA_STATISTICS.update.author}}</a>
          </div>
        </div>
      </div>
    </modal>
    <!-- <div v-if="DATA_STATISTICS && DATA_STATISTICS.update" class="fixed-info">
        数据来源于 <a target="_blank" :href="DATA_STATISTICS.update.url">{{DATA_STATISTICS.update.author}}</a>
        &nbsp;&nbsp;更新时间：{{DATA_STATISTICS.update.date}}
    </div> -->
    <div id="container"></div>
  </div>
</template>

<script>
import { DATA_CASE, DATA_STATISTICS } from './data.js'
import modal from './components/modal'
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
      DATA_STATISTICS,
      dataCase: [],
      maxCount: 0,
      map: null,
      showMode: 'marker',
      cityPoint: null,
      currentPoint: null,
      currentMarker: null,
      markerList: [],
      heatmapOverlay: null
    }
  },
  components: { modal },
  mounted () {
    this.parseData()
    this.initMap()
    this.initMarker()
    this.initHeat()
  },
  methods: {
    parseData () {
      this.dataCase = (() => {
        let areaList = []
        DATA_CASE.forEach((area) => {
          let tempArea = {
            key: area.key,
            list: [],
            count: area.list.length || 0,
            peopleCount: 0,
            peopleDischarge: 0
          }
          area.list.forEach((item) => {
            item.distance = null
            tempArea.peopleCount += item.count
            tempArea.peopleDischarge += item.discharge
            tempArea.list.push(item)
            this.maxCount = Math.max(this.maxCount, item.count)
          })
          areaList.push(tempArea)
        })
        return areaList
      })()
    },
    initMap () {
      if (!BMap) {
        return
      }
      this.map = new BMap.Map('container')
      this.cityPoint = new BMap.Point(117.233675, 31.827828) // 合肥
      this.map.centerAndZoom(this.cityPoint, 12)
      this.map.enableScrollWheelZoom()
    },
    initMarker () {
      this.dataCase.forEach((area) => {
        area.list.forEach((item) => {
          let point = new BMap.Point(item.lng, item.lat)
          let label = new BMap.Label(`${item.name}&nbsp;<span style="color:#888">确诊<b style="margin-left: 1px;color:#dc6450; font-weight: normal">${item.count}</b>&nbsp;出院<b style="margin-left: 1px;color:#dc6450; font-weight: normal">${item.discharge}</b></span>`, {
            offset: new BMap.Size(5, -20)
          })
          label.setStyle({ padding: '3px 3px', lineHeight: '1em', color: '#dc6450', fontSize: '11px', backgroundColor: 'rgba(255,255,255, 0.8)', borderColor: 'rgba(220,100,80,0.5)' })
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
      this.dataCase.forEach((area) => {
        area.list.forEach((item) => {
          points.push({
            'lng': item.lng, 'lat': item.lat, 'count': item.count
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
      this.heatmapOverlay.setDataSet({ data: points, max: this.maxCount })
      if (this.showMode !== 'heat') {
        this.heatmapOverlay.hide()
      }
    },
    switchShowMode (mode) {
      this.showMode = mode
      // this.map.setZoom(12)
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
        this.currentPoint = r.point
        this.getDistance()
        if (this.currentMarker) {
          this.currentMarker.remove()
        }
        let icon = new BMap.Icon('./img/point.png', new BMap.Size(30, 30), {
          imageSize: new BMap.Size(30, 30)
        })
        this.currentMarker = new BMap.Marker(r.point, { icon })
        this.map.addOverlay(this.currentMarker)
        this.map.centerAndZoom(r.point, 15)
      }, { enableHighAccuracy: true })
      window['_hmt'] && window['_hmt'].push(['_trackEvent', '我的位置', ''])
    },
    getDistance () {
      this.dataCase.forEach((area) => {
        area.list.forEach((item) => {
          let point = new BMap.Point(item.lng, item.lat)
          let distance = Math.ceil(this.map.getDistance(this.currentPoint, point))
          item.distance = (distance / 1000).toFixed(2)
        })
        // 区域内排序
        area.list.sort((a, b) => {
          if (a.distance && b.distance) {
            return a.distance - b.distance
          }
          return -1
        })
      })
      // 区域排序
      this.dataCase.sort((a, b) => {
        let aTemp = a.list[0]
        let bTemp = b.list[0]
        if (aTemp && aTemp.distance && bTemp && bTemp.distance) {
          return aTemp.distance - bTemp.distance
        }
        return -1
      })
    },
    goGithub () {
      window['_hmt'] && window['_hmt'].push(['_trackEvent', '查看源码', ''])
    },
    showDetail () {
      this.$refs.modalDetail.toggle()
      window['_hmt'] && window['_hmt'].push(['_trackEvent', '查看更多', ''])
    }
  }
}
</script>

<style lang="scss" scoped>
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
  bottom: 15px;
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
.modal-detail
{
  .tip
  {
    // margin-top: 5px;
    padding: 8px 0px;
    color: #666;
    font-size: 11px;
    text-align: center;
    // border-top: solid 1px rgba(#ddd, 0.4);
    a
    {
      font-size: 12px;
      color: #dc6450;
    }
  }
  .area-group
  {
    flex: 1;
    margin: 5px 5px 2px 5px;
    border: solid 1px rgba(#e4e4e4, 0.6);
    overflow: hidden;
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
    .slot-wrapper
    {
      height: calc(100% + 1px); /*no*/
    }
  }
  .area-list
  {
    background-color: #F7F7F7;
    overflow: hidden;
    transition: all 0.3s;
    .name
    {
      line-height: 34px;
      text-indent: 5px;
      font-size: 15px;
      color: #333;
      span
      {
        margin-left: 5px;
        font-size: 13px;
        color: #666;
        b
        {
          margin: 0px 2px;
          color: #dc6450;
          font-weight: normal;
          font-size: 14px;
        }
      }
    }
    .community-list
    {
      padding: 3px 10px 0px 8px;
      background-color: #fff;
      .item
      {
        position: relative;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        height: 32px;
        font-size: 13px;
        color: #444;
        label
        {
          flex: 1;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
          b
          {
            margin: 0px 1px;
            color: #dc6450;
            font-weight: normal;
          }
        }
        span
        {
          flex: 0 0 100px;
          color: #666;
          font-size: 12px;
          text-align: right;
        }
      }
      .item + .item
      {
        &::before
        {
          content: '';
          position: absolute;
          top: 0px;
          left: 0px;
          width: 100%;
          height: 1px;
          background-color: rgba(#ddd, 0.7);
          transform: scaleY(0.5);
        }
      }
    }
  }
  .source-info
  {
    padding: 10px 0px;
    text-align: center;
    color: #666;
    font-size: 13px;
    border-top: solid 1px #f7f7f7;
    a
    {
      color: #576b95;
    }
  }
}

</style>
