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
						uni.startBluetoothDevicesDiscovery({
							services:[],
							success(res){
								console.log(res);
							},
							fail(res) {
								console.log(res);
							}
						})
						
						//监听搜索到的设备
						// let mac="90:70:65:FC:6C:7A";
						let mac="01:02:03:04:06:99";
						uni.onBluetoothDeviceFound((res)=>{
							let devices=res.devices;
							for(let i=0;i<devices.length;i++){
								if(devices[i].deviceId==mac){
									console.log(devices[i]);
									
									//连接指定设备
									uni.createBLEConnection({
										deviceId:devices[i].deviceId,
										success: (res) => {
											console.log(res);
											
											//获取设备所有服务
											let device_id=devices[i].deviceId;
											uni.getBLEDeviceServices({
												deviceId:device_id,
												success: (res) => {
													console.log(res);
													
													//获取服务特征值
													let service_id=res.services[0].uuid
													uni.getBLEDeviceCharacteristics({
														deviceId:device_id,
														serviceId:service_id,
														success: (res) => {
															console.log(res);
															
															//订阅指定特征值
															let notify_id=res.characteristics[0].uuid
															uni.notifyBLECharacteristicValueChange({
																deviceId:device_id,
																serviceId:service_id,
																characteristicId:notify_id,
																state:true,
																success: (res) => { 
																	console.log(res);
																},
																fail: (res) => {
																	console.log(res);
																},
																complete: (res) => {
																	
																}
															});
															
															//类型转换函数
															function ab2hex (buffer) {
																var hexArr=Array.prototype.map.call(
																	new Uint8Array(buffer),
																	function(bit){
																		return ('00'+bit.toString(16).slice(-2))
																	}
																)
																return hexArr.join('');
															}
															
															//监听特征值变化
															uni.onBLECharacteristicValueChange(function(res){
																console.log(res);
															})
														},
														fail: (res) => {
															console.log(res);
														},
														complete: (res) => {
															
														}
													})
												},
												fail: (res) => {
													console.log(res);
												},
												complete: (res) => {
													
												}
											})
										},
										fail: (res) => {
											console.log(res);
										},
										complete: (res) => {
											
										}
									})
									
									
									//停止蓝牙搜索
									uni.stopBluetoothDevicesDiscovery({
										success:(res)=>console.log(res),
										fail:(res)=>console.log(res)
									});
								}
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
