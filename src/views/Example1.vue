<template>
	<div class="mainDiv">
		<div id="mapDiv"></div>
	</div>
</template>

<script>
import 'leaflet/dist/leaflet.css'
import $L from 'leaflet'
import * as esri from 'esri-leaflet'
import 'esri-leaflet-renderers'
import province from '../static/province.json'

export default {
	data() {
		return {
			map: null,
			baseLayer: {},
			overLayer: null,
			urls: {
				base: 'http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineCommunity/MapServer',
				hydro: 'http://thematic.geoq.cn/arcgis/rest/services/ThematicMaps/WorldHydroMap/MapServer',
				pointFeature: 'http://sampleserver6.arcgisonline.com/arcgis/rest/services/Hurricanes/MapServer/0/',
				lineFeature: 'http://sampleserver6.arcgisonline.com/arcgis/rest/services/Hurricanes/MapServer/1/'
			}
		}
	},
	mounted(){
		this.map = $L.map('mapDiv', {
			maxZoom: 20,
			minZoom: 2,
			zoom: 3,
			center: [40, 80]
		})
		this.addBaseMap()
		this.addJsonLayer()
		this.addFeatureLayer()
	},
	methods: {
		// 加载底图
		async addBaseMap(){
			const base = await esri.tiledMapLayer({      // 加载ArcGIS MapServer
				url: this.urls.base
			}).addTo(this.map)
			const hydro = await esri.tiledMapLayer({
				url: this.urls.hydro
			})
			const gray = esri.basemapLayer('DarkGray')  // Esri提供的地图数据
			this.baseLayer = {
				'基础地图': base,
				'水系地图': hydro,
				'暗色地图': gray
			}
			$L.control.layers(this.baseLayer, this.overLayer).addTo(this.map)
		},
		// 加载要素图层
		addFeatureLayer() {
			const point = esri.featureLayer({
				url: this.urls.pointFeature
			})
			const line = esri.featureLayer({
				url: this.urls.lineFeature
			})
			point.bindPopup( lyr => {
				return $L.Util.template(`<h3>点要素属性</h3>
					<p>TIME: {TIME}</p>
					<p>DAY: {DAY}</p>
					<p>LatLng: {LAT}, {LONG}</p>
					<p>PRESSURE: {PRESSURE}</p>
					<p>WINDSPEED: {WINDSPEED}</p>
					<p>STAGE: {STAGE}</p>`, lyr.feature.properties)
			})
			this.overLayer['点要素'] = point
			this.overLayer['线要素'] = line
		},
		// 加载geoJson数据
		addJsonLayer() {
			const style = {
				'color': 'blue',
        'weight': 0.5,
        'opacity': 0.5
			}
			const jsonLayer = $L.geoJSON(province, {style: style})
			jsonLayer.bindPopup( lyr => {
				return `<h3>弹窗信息</h3>
					<p>省名：${lyr.feature.properties.name}</p>
					<p>行政编码：${lyr.feature.properties.code}</p>`
			}).addTo(this.map)
			this.overLayer = {
				'省级行政区划': jsonLayer
			}
		}
	}
}
</script>

<style scoped>
	.mainDiv {
		width: 100%;
		height: calc(100% - 18px);
	}
	#mapDiv {
		width: 100%;
		height: 100%;
		/* height: calc(100% - 18px); */
	}
</style>