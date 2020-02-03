<template>
  <div id="app">
    <div class="header">
      <h1>合肥新冠状病毒感染确诊病例</h1>
      <p>所有数据来源于网络，仅供参考，不喜勿喷</p>
    </div>
    <div class="fixed-link">
      <a href="https://github.com/twoer/2019-nCoV-HF">
      <img src="data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QMuaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0OCA3OS4xNjQwMzYsIDIwMTkvMDgvMTMtMDE6MDY6NTcgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLyIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCAyMS4wIChNYWNpbnRvc2gpIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjc4QTJCQUY5M0U1RTExRUE4RTgwQjUyODYyQTg4RjY4IiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjc4QTJCQUZBM0U1RTExRUE4RTgwQjUyODYyQTg4RjY4Ij4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6NzhBMkJBRjczRTVFMTFFQThFODBCNTI4NjJBODhGNjgiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6NzhBMkJBRjgzRTVFMTFFQThFODBCNTI4NjJBODhGNjgiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEB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">
      </a>
    </div>
    <div id="container"></div>
  </div>
</template>

<script>
export default {
  name: 'app',
  mounted () {
    const BMap = window['BMap']
    const BMapLib = window['BMapLib']
    if (!BMap) {
      return
    }
    let map = new BMap.Map('container')
    let point = new BMap.Point(117.233675, 31.827828)
    map.centerAndZoom(point, 12)
    map.enableScrollWheelZoom()

    let points = [
      { 'lng': 117.24743, 'lat': 31.87303, 'count': 10 }, // 中铁青秀城
      { 'lng': 117.242822, 'lat': 31.874133, 'count': 10 }, // 中央美域
      { 'lng': 117.230494, 'lat': 31.835576, 'count': 10 }, // 当代花园
      { 'lng': 117.220322, 'lat': 31.82185, 'count': 10 }, // 内森
      { 'lng': 117.218562, 'lat': 31.815099, 'count': 10 }, // 天和苑
      { 'lng': 117.22591, 'lat': 31.760895, 'count': 5 }, // 名邦景秀年华 疑似
      { 'lng': 117.291492, 'lat': 31.90401, 'count': 10 }, // 探矿
      { 'lng': 117.276182, 'lat': 31.882694, 'count': 10 }, // 柏景湾
      { 'lng': 117.180201, 'lat': 31.82937, 'count': 10 }, // 乔治庄园
      { 'lng': 117.156127, 'lat': 31.828954, 'count': 10 }, // 岭湖墅
      { 'lng': 117.290324, 'lat': 31.738736, 'count': 10 }, // 滨湖明珠
      { 'lng': 117.38432, 'lat': 31.871841, 'count': 10 }, // 华都城市花园
      { 'lng': 117.307049, 'lat': 31.887935, 'count': 10 }, // 怡馨苑
      { 'lng': 117.379427, 'lat': 31.910114, 'count': 10 }, // 合郢花园
      { 'lng': 117.349143, 'lat': 31.905215, 'count': 10 }, // 新海尚宸家园
      { 'lng': 117.327947, 'lat': 31.902247, 'count': 10 }, // 家天下三期
      { 'lng': 117.494224, 'lat': 31.883583, 'count': 10 }, // 定光社区
      { 'lng': 117.466085, 'lat': 31.896163, 'count': 10 }, // 汇景新城
      { 'lng': 117.474501, 'lat': 31.891716, 'count': 10 }, // 肥东老街
      { 'lng': 117.45986, 'lat': 31.896704, 'count': 10 }, // 光大新村
      { 'lng': 117.196832, 'lat': 31.952793, 'count': 10 }, // 玉成明珠苑
      { 'lng': 117.884278, 'lat': 31.621156, 'count': 10 } // 巢湖城市之光
    ]
    if (!this.isSupportCanvas()) {
      alert('热力图目前只支持有canvas支持的浏览器,您所使用的浏览器不能使用热力图功能~')
      return
    }
    let heatmapOverlay = new BMapLib.HeatmapOverlay({ 'radius': 20,
      gradient: {
        0.2: 'rgb(0, 255, 255)',
        0.5: 'rgb(0, 110, 255)',
        0.8: 'rgb(100, 0, 255)'
      } })
    map.addOverlay(heatmapOverlay)
    heatmapOverlay.setDataSet({ data: points, max: 10 })
    heatmapOverlay.show()
  },
  created () {

  },
  methods: {
    isSupportCanvas () {
      var elem = document.createElement('canvas')
      return !!(elem.getContext && elem.getContext('2d'))
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
  }
}
.fixed-link
{
  position: fixed;
  top: 50px;
  right: 0px;
  z-index: 9999;
  a
  {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    width: 40px;
    height: 40px;
    img
    {
      max-width: 36px;
      max-height: 36px;
      border-radius: 50%;
    }
  }
}
@media all and (min-width: 768px) {
  .fixed-link
  {
    top: 5px;
    right: 10px;
  }
}
</style>
