<template>
  <div>
    <div id="container-map" />
  </div>
</template>

<script>
import AMapLoader from '@amap/amap-jsapi-loader'
export default {
  name: 'GaoDeMap',
  props: {
    positions: {
      type: Array,
      default() {
        return [
          // {
          //   data: [116.39, 39.9], // 经纬度
          //   title: '北京' // 名称
          // }
        ]
      }
    },
    keyword: { // 行政区域查询关键词
      type: String,
      default: ''
    },
    level: { // 行政区域查询等级
      type: String,
      default: 'city'
    }
  },
  methods: {
    init() {
      AMapLoader.load({
        'key': '', // 申请好的Web端开发者Key，首次调用 load 时必填
        'version': '2.0', // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        'plugins': ['AMap.Marker', 'AMap.ToolBar', 'AMap.Scale', 'AMap.DistrictSearch', 'AMap.Polygon', 'AMap.InfoWindow'], // 需要使用的的插件列表，如比例尺'AMap.Scale'等
        'AMapUI': { // 是否加载 AMapUI，缺省不加载
          'version': '1.1', // AMapUI 缺省 1.1
          'plugins': [] // 需要加载的 AMapUI ui插件
        }
      }).then((AMap) => {
        console.log('Canvas:' + AMap.Browser.isCanvas)
        console.log('WebGL:' + AMap.Browser.isWebGL)
        const map = new AMap.Map('container-map')

        map.addControl(new AMap.ToolBar()) // 缩放工具
        map.addControl(new AMap.Scale()) // 比例尺工具

        // 信息窗口
        const infoWindow = new AMap.InfoWindow({
          content: '',
          anchor: 'top-left'
        })

        // 标记点
        const markers = []
        this.positions.forEach((item) => {
          const marker = new AMap.Marker({
            position: item.data, // new AMap.LngLat(116.39, 39.9)经纬度对象，也可以是经纬度构成的一维数组[116.39, 39.9]
            title: item.title
          })
          const content = `<div style="">定位地址：</div>
                           <div style="">${item.title}</div>`
          marker.on('click', () => {
            infoWindow.open(map, marker.getPosition())
            infoWindow.setContent(content)
          })
          markers.push(marker)
        })
        // map.remove(markers) // 移除已创建的marker
        map.add(markers)

        // 行政区域查询
        if (this.keyword) {
          const district = new AMap.DistrictSearch({
            subdistrict: 0, // 获取边界不需要返回下级行政区
            extensions: 'all', // 返回行政区边界坐标组等具体信息
            level: this.level // 查询行政级别为district,city,province
          })
          district.search(this.keyword, (status, result) => {
            console.log('DistrictSearch:' + status)
            // console.log(result)
            // 获取边界信息
            const bounds = result.districtList[0].boundaries
            const center = result.districtList[0].center
            const polygons = []
            if (bounds) {
              for (let i = 0, l = bounds.length; i < l; i++) {
                // 生成行政区划polygon
                const polygon = new AMap.Polygon({
                  strokeWeight: 1,
                  path: bounds[i],
                  fillOpacity: 0.4,
                  fillColor: '#CCF3FF',
                  strokeColor: '#CC66CC',
                  cursor: 'pointer'
                })
                const content = `<div style="">定位地址：</div>
                                 <div style="">${this.keyword}</div>`
                polygon.on('click', () => {
                  infoWindow.open(map, [center.lng, center.lat])
                  infoWindow.setContent(content)
                })
                polygons.push(polygon)
              }
              // map.remove(polygons)// 清除上次结果
              map.add(polygons)
            }
          })
        }

        // 地图自适应
        map.setFitView()
      }).catch(e => {
        console.log('地图加载出错' + e)
      })
    }
  },
  beforeDestroy() {
    // map.clearMap() // 清空地图
    // map.destroy() // 销毁地图，释放内存
  }
}
</script>

<style lang="scss" scoped>
#container-map {
  width: 100%;
  height: 0;
  padding-top: 50%;
}
</style>
