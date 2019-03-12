<template>
	<view class="content">
		<view class="uni-text">
			<button type="default" @click="connect">蓝牙</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				d:[]
			};
		},
		methods:{
			connect(){
				let _this=this;
				//打开蓝牙模块
				uni.openBluetoothAdapter({
					success:(res)=>{
						console.log(res);
					},
					fail:(res)=>{
						console.log(res);
					},
					complete:(res)=>{
						uni.onBluetoothAdapterStateChange(function(res){
// 							if(res.available){
// 							  setTimeout(function(){
// 								_this.connect();
// 							  },2000);
// 							}
						})
						
						//开始搜索蓝牙设备
						let mac="90:70:65:FC:6C:7A";
						uni.startBluetoothDevicesDiscovery({
							services:[],
							success(res){
								uni.onBluetoothDeviceFound((res)=>{
									let devices=res.devices;
									for(let i=0;i<devices.length;i++){
										if(devices[i].deviceId==mac){
											console.log("find");
											console.log(devices[i]);
											uni.stopBluetoothDevicesDiscovery({
												success:(res)=>console.log(res),
												fail:(res)=>console.log(res)
											})
										}
									}
								})
							},
							fail(res) {
								console.log(res);
							}
						})
					}
				})
			}
		}
	}
</script>

<style>
.uni-text{
		padding: 10upx 2%;
	}
</style>
