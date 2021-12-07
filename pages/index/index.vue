<template>
	<view class="content">
		<map id="map" style="width: 750rpx;height: 100vh;" :latitude="latitude" :longitude="longitude" scale="17"
			@bindtap="map" show-location="true" :show-compass="true" enable-poi="true" :markers="markers"></map>
	</view>
</template>

<script>
	export default {
		data() {
			return {
                //https://apis.map.qq.com/ws/place/v1/search?keyword=酒店&boundary=nearby(24.47951,118.08948,1000)&key=SSDBZ-TGICF-Q6GJ3-NSZED-3Q4RV-N7BWS		
				longitude: 0, //经度
				latitude: 0, //纬度
				accuracy: 0, //定位精度
				markers: [ //渲染地图上的坐标
					// {
					// 	id:1,
					// 	width:40,
					// 	height:40,
					// 	iconPath:'../../static/地址.png',
					// 	latitude:0,
					// 	longitude:0,

					// },
					{
						id: 1,
						width: 40,
						height: 40,
						iconPath: '../../static/地址.png',
						latitude: 24.492835,
						longitude: 118.1783,
						callout: {
							content: '气泡3',
							color: '#444444',
							padding: 5,
							fontSize: 15,
							borderRadius: 5,
							display: 'ALWAYS',
							textAlign: 'center',
						}

					}, {
						id: 2,
						width: 40,
						height: 40,
						iconPath: '../../static/地址.png',
						latitude: 24.491318,
						longitude: 118.178143,
						callout: {
							content: '气泡2',
							color: '#444444',
							padding: 5,
							fontSize: 15,
							borderRadius: 5,
							display: 'ALWAYS',
							textAlign: 'center',
						}
					}, {
						id: 3,
						width: 40,
						height: 40,
						iconPath: '../../static/地址.png',
						latitude: 24.49121,
						longitude: 118.179734,
						callout: {
							content: '气泡1',
							color: '#444444',
							padding: 5,
							fontSize: 15,
							borderRadius: 5,
							display: 'ALWAYS',
							textAlign: 'center',
						}
					},
				]
			}
		},

		async onLoad() {


			// this.gitctiy()

			// const res2 = await uni.openLocation({
			// 	latitude:this.latitude,
			// 	longitude:this.longitude,
			// 	scale:14
			// })
			// console.log(res2)
			this.getMyPosition()
			// this.gitctiy()
			// this.getMapLocation()


		},
		methods: {
			async getMyPosition() { //获取当前位置经纬度方法
				const res = await uni.getLocation({
					type: 'gcj02'
				})
				const {
					latitude,
					longitude,
					accuracy
				} = res[1]
				this.latitude = latitude
				this.longitude = longitude
				this.accuracy = accuracy
				// this.markers[0].latitude = latitude
				// this.markers[0].longitude = longitude
				console.log(this.longitude, this.latitude)












				//定位的位置和搜素定位的位置显示在同一个屏幕内
				this.mapCtx = wx.createMapContext('map') //获取map组件的实例
				this.mapCtx.includePoints({
					padding: [100, 40, 40, 100],
					points: [{ //目前所在的位置经纬度
							latitude: this.latitude,
							longitude: this.longitude
						},
						{ //搜素地的经纬度
							latitude: this.markers[0].latitude,
							longitude: this.markers[0].longitude
						}
					]
				})



				// this.mapCtx.moveToLocation({  // 视野回到自己的位置
				// })

			},
			// gitctiy() {
			// 	uni.chooseLocation({
			// 		latitude: this.latitude,
			// 		longitude: this.longitude,
			// 		success: res => {
			// 			console.log(res)
			// 		}

			// 	})
			// },
			// getMapLocation(){
			// 	uni.chooseLocation({
			// 		success:(res)=> {
			// 			console.log(res);
			// 			// this.getRegionFn(res);
			// 		},
			// 		fail:()=>{
			// 			// 如果用uni.chooseLocation没有获取到地理位置，则需要获取当前的授权信息，判断是否有地理授权信息
			// 			uni.getSetting({
			// 				success: (res) => {
			// 					console.log(res);
			// 					var status = res.authSetting;
			// 					if(!status['scope.userLocation']){
			// 					// 如果授权信息中没有地理位置的授权，则需要弹窗提示用户需要授权地理信息
			// 						uni.showModal({
			// 							title:"是否授权当前位置",
			// 							content:"需要获取您的地理位置，请确认授权，否则地图功能将无法使用",
			// 							success:(tip)=>{
			// 								if(tip.confirm){
			// 								// 如果用户同意授权地理信息，则打开授权设置页面，判断用户的操作
			// 									uni.openSetting({
			// 										success:(data)=>{
			// 										// 如果用户授权了地理信息在，则提示授权成功
			// 											if(data.authSetting['scope.userLocation']===true){
			// 												uni.showToast({
			// 													title:"授权成功",
			// 													icon:"success",
			// 													duration:1000
			// 												})
			// 												// 授权成功后，然后再次chooseLocation获取信息
			// 												uni.chooseLocation({
			// 													success: (res) => {
			// 														console.log("详细地址",res);
			// 														// this.getRegionFn(res);
			// 													}
			// 												})
			// 											}else{
			// 												uni.showToast({
			// 													title:"授权失败",
			// 													icon:"none",
			// 													duration:1000
			// 												})
			// 											}
			// 										}
			// 									})
			// 								}
			// 							}
			// 						})
			// 					}
			// 				},
			// 				fail: (res) => {
			// 					uni.showToast({
			// 						title:"调用授权窗口失败",
			// 						icon:"none",
			// 						duration:1000
			// 					})
			// 				}
			// 			})
			// 		}
			// 	});
			// },

		},
	}
</script>

<style>
/* 	.content {
		width: 750rpx;
		height: 100%;
	} */
</style>
