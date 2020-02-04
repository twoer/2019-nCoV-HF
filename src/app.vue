<template>
  <div id="app">
    <div class="header">
      <h1>合肥新冠状病毒感染确诊病例</h1>
      <p>所有数据来源于 <a target="_blank" href="https://mp.weixin.qq.com/s/Kl_LM0j-cEWklZAh0l8W-g">新安晚报</a> ，仅供参考，不喜勿喷</p>
    </div>
    <div class="fixed-link">
      <!-- <a v-if="showMode === 'distribute'" @click="switchShowMode('heat')"  class="icon-mode">
        <img src="./assets/icon-distribute.jpg">
      </a>
      <a v-if="showMode === 'heat'" @click="switchShowMode('distribute')" class="icon-mode">
        <img src="./assets/icon-heat.jpg">
      </a> -->
      <a @click="getCurrentLocation" class="icon-location">
        <img src="./assets/icon-location.jpg">
      </a>
      <a href="https://github.com/twoer/2019-nCoV-HF">
        <img src="./assets/icon-github.jpg">
      </a>
    </div>
    <div class="fixed-info">数据更新于 2020/02/04 20:00 共计(60)个小区</div>
    <div id="container"></div>
  </div>
</template>

<script>
const BMap = window['BMap']
const BMapLib = window['BMapLib']
const CASE_DATA = [
  {
    key: '肥东县',
    list: ['117.427506,31.895833,陈大郢社区紫玉华府',
      '117.494224,31.883583,店埠镇定光社区新农村小区',
      '117.458584,31.896521,店埠镇光大居委会光大新村',
      '117.466085,31.896163,店埠镇和平社区汇景新城',
      '117.459392,31.874001,店埠镇新城社区新城家园',
      '117.478499,31.889884,店埠镇镇南社区',
      '117.477684,31.890811,店埠镇青春社区育红小学宿舍楼',
      '117.494224,31.883583,店埠镇定光社区传桥组',
      '117.84373,32.077629,马湖乡马湖社区',
      '117.852802,32.054828,马湖乡沙河村',
      '117.684292,32.145053,八斗镇王城社区',
      '117.574411,31.905293,石塘镇龙城社区',
      '117.284078,31.862031,石塘镇供销社宿舍',
      '117.487648,31.95986,牌坊回族满族乡民新村']
  },
  {
    key: '肥西县',
    list: [
      '117.201326,31.69008,上派镇韩圩村',
      '117.170393,31.732746,上派镇华邦万派城',
      '117.179595,31.789791,桃花镇东冠繁华逸城'
    ]
  },
  {
    key: '长丰县',
    list: [
      '117.19813,31.952126,岗集镇玉成明珠苑',
      '117.264533,31.957851,阿奎利亚城品'
    ]
  },
  {
    key: '庐江县',
    list: [
      '117.301121,31.263588,庐城镇翠绿园',
      '117.399161,30.988277,罗河镇鲍店村',
      '117.438949,31.071198,矾山镇新中村',
      '117.19249,31.195572,柯坦镇城池村',
      '117.340362,31.275924,冶父山镇冶父山社区',
      '117.176552,31.38256,金牛镇铺岗村',
      '117.298829,31.482542,同大镇施丰村',
      '117.265351,31.084395,罗河镇吉桥村'
    ]
  },
  {
    key: '巢湖市',
    list: [
      '117.884472,31.622419,凤凰山街道城市之光C区',
      '117.620552,31.422466,槐林镇武山居委会'
    ]
  },
  {
    key: '瑶海区',
    list: [
      '117.309993,31.892159,临泉花苑小区',
      '117.307049,31.887935,怡馨苑',
      '117.38432,31.871841,华都城市花园',
      '117.297459,31.894408,元一名城',
      '117.328291,31.879792,凤翔家园',
      '117.304492,31.882357,恒大中央广场',
      '117.316008,31.880239,温莎国际广场',
      '117.33319,31.873209,长江东路五里井17栋',
      '117.330442,31.877155,铜陵路工业锅炉厂宿舍',
      '117.374088,31.880635,斯瑞新景苑',
      '117.330529,31.875132,恒丰嘉园'
    ]
  },
  {
    key: '庐阳区',
    list: [
      '117.291757,31.909907,探矿厂宿舍',
      '117.278023,31.882968,柏景湾青云轩',
      '117.262834,31.880638,古城新村'
    ]
  },
  {
    key: '蜀山区',
    list: [
      '117.232031,31.836028,当代花园',
      '117.243143,31.873707,中央美域A区',
      '117.217465,31.814236,天和园',
      '117.220322,31.82185,内森庄园',
      '117.250595,31.876861,中铁青秀城',
      '117.266745,31.874394,和谐家园（长丰南路）',
      '117.248554,31.863732,颐和阳光100小区',
      '117.248281,31.871345,龙居山庄天龙居'
    ]
  },
  {
    key: '包河区',
    list: [
      '117.282056,31.830598,曙宏南苑',
      '117.28879,31.738861,滨湖明珠西区',
      '117.284451,31.713328,豪门金地小区',
      '117.312875,31.855629,金湾嘉园'
    ]
  },
  {
    key: '高新区',
    list: [
      '117.156127,31.828954,岭湖墅'
    ]
  },
  {
    key: '新站区',
    list: [
      '117.379427,31.910114,合郢花园',
      '117.327947,31.902247,家天下3期',
      '117.312057,31.904238,昊天园小区',
      '117.349143,31.905215,新海尚宸家园'
    ]
  }
]
export default {
  name: 'app',
  data () {
    return {
      showMode: 'heat',
      map: null,
      currentPoint: null,
      heatmapOverlay: null
    }
  },
  mounted () {
    const isSupportCanvas = () => {
      let elem = document.createElement('canvas')
      return !!(elem.getContext && elem.getContext('2d'))
    }
    if (!BMap) {
      return
    }
    this.map = new BMap.Map('container')
    let point = new BMap.Point(117.233675, 31.827828)
    this.map.centerAndZoom(point, 12)
    this.map.enableScrollWheelZoom()

    let points = [
    ]
    CASE_DATA.forEach((area) => {
      area.list.forEach((item) => {
        let temp = item.split(',')
        points.push({
          'lng': temp[0], 'lat': temp[1], 'count': 10
        })
      })
    })
    if (!isSupportCanvas()) {
      alert('热力图目前只支持有canvas支持的浏览器,您所使用的浏览器不能使用热力图功能~')
      return
    }
    this.heatmapOverlay = new BMapLib.HeatmapOverlay({ 'radius': 20,
      gradient: {
        0.2: 'rgb(0, 255, 255)',
        0.5: 'rgb(0, 110, 255)',
        0.8: 'rgb(100, 0, 255)'
      } })
    this.map.addOverlay(this.heatmapOverlay)
    this.heatmapOverlay.setDataSet({ data: points, max: 13 })
    this.heatmapOverlay.show()
  },
  methods: {
    switchShowMode (mode) {
      this.showMode = mode
      if (mode === 'heat') {
        this.heatmapOverlay.show()
      } else if (mode === 'distribute') {
        this.heatmapOverlay.hide()
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
  width: 40px;
  height: 200px;
  z-index: 9999;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  a
  {
    display: block;
    position: relative;
    margin-top: 8px;
    width: 40px;
    height: 40px;
    background-color: #fff;
    border-radius: 50%;
    img
    {
      position: absolute;
      top: 50%;
      left: 50%;
      max-width: 32px;
      max-height: 32px;
      border-radius: 50%;
      transform: translate(-50%, -50%);
    }
    &.icon-location
    {
      img
      {
        max-width: 34px;
        max-height: 34px;
      }
    }
  }
}
.fixed-info
{
  position: fixed;
  bottom: 5px;
  left: 50%;
  z-index: 9999;
  padding: 0.2em 0.4em;
  color: #666;
  background-color: #fff;
  border-radius: 4px;
  transform: translateX(-50%);
  white-space: nowrap;
  font-size: 10px;
}
</style>
