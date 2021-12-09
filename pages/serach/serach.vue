<template>
	<view class="container" style="width: 750rpx; height: 100%">
		<template class="search">
			<uni-search-bar placeholder="输入要去的位置" @confirm="search" @cancel="cancel" cancel-text="取消" v-model="KeyWord">
				<uni-icons slot="searchIcon" color="#999999" size="18" type="home" />
			</uni-search-bar>
		</template>
		<map id="map" style="width: 100%; height: 800rpx;" show-location="true" :scale="scale" :latitude="latitude"
			:longitude="longitude" :enable-poi="true" enable-building="true" @tap="map" :markers="Markers"
			:circles="circle" :controls="controls" @callouttap="callouttap" @markertap="markersTap"
			:polyline="polyline">
			<cover-view style="position: absolute;top: 50upx;right: 30upx;width: 40upx;height: 40upx;"
				@tap="changeScale('+')">
				<cover-image src="../../static/jia.png"></cover-image>
			</cover-view>
			<cover-view style="position: absolute;top: 50upx;right: 60upx;width: 40upx;height: 40upx; margin: 0 20rpx;"
				@tap="changeScale('-')">
				<cover-image src="../../static/jian.png"></cover-image>
			</cover-view>
			<cover-view
				style="position: absolute;bottom: 40upx;right: 40upx;width: 50upx;height: 50upx; margin: 0 20rpx;"
				@tap="Reset">
				<cover-image src="../../static/dingwei.png"></cover-image>
			</cover-view>

			<cover-view
				style="position: absolute;bottom: 40upx;right: 100upx;width: 50upx;height: 50upx; margin: 0 20rpx;">
				<cover-image src="../../static/定位.png"></cover-image>
			</cover-view>
		</map>
		<!-- :include-points="includePoints" 需要准备一个数组,包含markers和当前位置的数组 -->
	</view>
</template>

<script>
	export default {
		data() {
			return {
				KeyWord: '', //搜索内容
				latitude: 0,
				longitude: 0,
				scale: 16,
				searchPosition: [],
				searchMarkers: [],
				MarkersId: [],
				includePoints: [],
				locationVision: [],
				AddressLocation: [],
				key: 'SSDBZ-TGICF-Q6GJ3-NSZED-3Q4RV-N7BWS',
				// polyline:{
				// 	points:[],
				// 	color:'#0000AA',
				// 	width:10,
				// 	borderWidth:2,
				// },
				polyline: [],
				circle: [{ //控制地图的园
					latitude: 0,
					longitude: 0,
					color: '#ffffff',
					fillColor: '#7cb5ec88',
					radius: 300,
					strokeWidth: 1,
				}, ],
				controls: [{
						id: 11,
						iconPath: '../../static/jia.png',
						position: {
							left: 320,
							top: 200,
							width: 200,
							height: 200
						},
						clickable: true,

					},
					{
						id: 12,
						iconPath: '../../static/jian.png',
						position: {
							left: 340,
							top: 200,
							width: 200,
							height: 200
						},
						clickable: true,

					},
				],

			}
		},
		onLoad() {
			this.getPosition()
		},
		methods: {
			async search(e) {
				this.searchMarkers = [];
				this.polyline = [];
				this.locationVision = [];
				this.mapCtx = wx.createMapContext('map') //获取map组件的实例
				this.mapCtx.removeMarkers({
					markerIds: this.MarkersId
				})

				const res = await uni.request({ //搜索
					url: `https://apis.map.qq.com/ws/place/v1/search?keyword=${this.KeyWord}&boundary=nearby(${this.latitude},${this.longitude},1000,0)&orderby=_distance&key=${this.key}`,
					// https://apis.map.qq.com/ws/place/v1/search?keyword=酒店&boundary=nearby(24.47951,118.08948,1000,1)&orderby=_distance&key=SSDBZ-TGICF-Q6GJ3-NSZED-3Q4RV-N7BWS
					// &page_size=20
				})

				const {
					data
				} = res[1]
				console.log(data.data)
				if (!data.data.length) return console.log('data.data不存在数据')
				this.searchPosition = data.data ? data.data : false;
				console.log(this.searchPosition, '加工的数据在这里')


				this.searchPosition.map((item, index, Arr) => { //准备添加的Markers数据


					const data = {
						id: Number(item.id),
						title: item.title, //有callout会忽略
						width: 30,
						height: 30,
						iconPath: '../../static/图钉.png',
						latitude: item.location.lat,
						longitude: item.location.lng,
						clusterId:Number(item.id),
						joinCluster:true,
						callout: {
							content: item.title,
							color: '#444444',
							padding: 5,
							fontSize: 12,
							borderRadius: 5,
							display: 'BYCLICK',
							textAlign: 'center',
							bgColor: '#cccccc'
						},
						label: {
							content: index + 1,
							color: '#FFFFFF',
							fontSize: 12,
							anchorX: '-8rpx',
							anchorY: '-58rpx',
							joinCluster:true,
						},

					}; //ALWAYS


					console.log(this.searchPosition, 'searchP')
					const myPosition = {
						latitude: item.location.lat,
						longitude: item.location.lng,
					}
					this.searchMarkers.push(data)
					this.locationVision.push(myPosition)
					// this.includePoints.push(data)
					// this.includePoints.push(myPosition)
					this.MarkersId.push(item.id)
				})

				this.locationVision.push({
					latitude: this.latitude,
					longitude: this.longitude
				})


				console.log(this.searchMarkers)
				this.mapCtx.addMarkers({
					markers: this.searchMarkers,
					clear: true,
					success: () => {},
					fail: (e) => {
						console.log(e)
					}
				})





				this.mapCtx.includePoints({
					points: this.locationVision,
					padding: [50, 50, 50, 50]
				})
				console.log(this.locationVision)
			},
			cancel() { //取消

			},

			async getPosition() { //获得目前所在地的定位经纬度
				const res = await uni.getLocation({
					type: 'gcj02',
					geocode: true
				})
				const {
					latitude,
					longitude
				} = res[1]
				this.latitude = latitude
				this.longitude = longitude

				this.circle[0].latitude = latitude
				this.circle[0].longitude = longitude
				console.log(res[1], '-----------------------------------------------------')
			},
			map(e) {
				console.log(e)
			},
			changeScale(scale) { //控制 Scale
				// if(this.scale == 1 || this.scale == 20) return
				this.scale <= 3 ? this.scale = 3 : this.scale;
				this.scale >= 20 ? this.scale = 20 : this.scale;




				scale == '+' ? this.scale++ : this.scale--;
				console.log(this, 'ChangeScale')

			},
			Reset() { //回到当前坐标位置 

				this.mapCtx = wx.createMapContext('map') //获取map组件的实例
				this.mapCtx.moveToLocation()
				// this.scale !== 16 ? this.scale = 16 : this.scale;
				// this.$set(this.scale,16)
				console.log(this.scale)
			},
			callouttap(e) { //点击气泡触发的函数
				console.log('触发了气泡', e)
			},
			async markersTap(e) {
				const Address = await uni.request({
					url: `https://apis.map.qq.com/ws/geocoder/v1/?location=${this.latitude},${this.longitude}&get_poi=0&key=${this.key}`
				})
				const {
					result
				} = Address[1].data
				const {
					address,
					formatted_addresses
				} = result
				this.AddressLocation.push({
					address,
					recommend_Address: formatted_addresses.recommend
				})
				const resOne = this.searchPosition.find((item, index) => {
					return item.id == e.detail.markerId;
				})
				this.AddressLocation.push({
					address: resOne.address,
					recommend_Address: resOne.title
				})

				console.log(this.AddressLocation, 'this.AddressLocation')

				this.polyline = []
				this.mapCtx = wx.createMapContext('map') //获取map组件的实例
				const res = this.searchMarkers.find((item, index) => { //获取当前点击的标记点
					return item.id == e.detail.markerId;
				})
				const {
					callout,
					latitude,
					longitude
				} = res

				const polyline = await uni.request({
					url: `https://apis.map.qq.com/ws/direction/v1/driving/?from=${this.latitude},${this.longitude}&to=${latitude},${longitude}&output=json&callback=cb&key=${this.key}`
				})
				var coors = polyline[1].data.result.routes[0].polyline
				for (var i = 2; i < coors.length; i++) {
					coors[i] = coors[i - 2] + coors[i] / 1000000
				}
				console.log(polyline, '路线')
				console.log(res, '地点')
				const data = {
					points: [],
					color: '#3469aa',
					width: 4,
					borderWidth: 2,
					borderColor: '#5375aa',
					colorList: [
						'#DDDDDD', '#FFB7DD', '#FFCCCC', '#FFC8B4', '#FFDDAA', '#FFEE99', '#FFFFBB', '#EEFFBB',
						'#CCFF99',
						'#99FF99', '#BBFFEE', '#AAFFEE', '#99FFFF', '#CCEEFF', '#CCDDFF', '#CCCCFF', '#CCBBFF',
						'#D1BBFF',
						'#E8CCFF', '#F0BBFF', '#E38EFF', '#9955FF',
					]
				}
				coors.map((item, index, arr) => {
					// index % 2 === 0 ? data.latitude = arr[index]data.longitude = arr[index+1]: '' ;
					if (index % 2 === 0) {
						const Position = {
							latitude: arr[index],
							longitude: arr[index + 1]
						}

						// this.polyline.points.push(data)
						data.points.push(Position)
					}
				})
				this.polyline.push(data)
				console.log(this.polyline)
				this.mapCtx.addMarkers({
					clear: true,
					markers: [{
						id: 12345678,
						width: 35,
						height: 35,
						iconPath: '../../static/起.png',
						latitude: this.latitude,
						longitude: this.longitude,
						label: {
							content: '起',
							color: '#FFFFFF',
							fontSize: 14,
							anchorX: '-15rpx',
							anchorY: '-60rpx',

						},
					}, {
						id: 123456789,
						// title: item.title, //有callout会忽略
						width: 35,
						height: 35,
						iconPath: '../../static/终.png',
						latitude: latitude,
						longitude: longitude,
						label: {
							content: '终',
							color: '#FFFFFF',
							fontSize: 14,

							anchorX: '-15rpx',
							anchorY: '-60rpx',
						},
					}, ],

				})

				this.mapCtx.includePoints({
					points: data.points,
					padding: [50, 50, 50, 50],

				})
			}
		}
	}
</script>

<style lang="scss">
	.container {

		.search {
			display: flex;
			justify-content: center;
			align-items: center;
		}
	}
</style>
