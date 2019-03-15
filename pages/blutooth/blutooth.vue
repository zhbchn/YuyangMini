<template>
	<view class="content">
		<view class="uni-text">
			<p>C0信息:{{code}}</p>
			<p>当前状态:{{status}}</p>
		</view>
		<view class="uni-text">
			<button type="primary" @click="connect" :disabled="flag[0]">连接蓝牙</button>
		</view>
		<view class="uni-text">
			<button type="default" @click="sendC1" :disabled="flag[1]">发送C1</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				status:'',
				code:'无数据',
				flag:[false,true,true],
				device_id:'',
				service_id:'',
				characteristic_id:''
			};
		},
		methods:{
			arrayBufferToHexString(buffer){
				let bufferType = Object.prototype.toString.call(buffer)
				  if (buffer != '[object ArrayBuffer]') {
					return
				  }
				  let dataView = new DataView(buffer)

				  var hexStr = '';
				  for (var i = 0; i < dataView.byteLength; i++) {
					var str = dataView.getUint8(i);
					var hex = (str & 0xff).toString(16);
					hex = (hex.length === 1) ? ',' + hex : hex;
					hexStr += hex;
				  }
				  return hexStr.toUpperCase();
			},
			ab2hex(buffer){
				var hexArr=Array.prototype.map.call(
					new Uint8Array(buffer),
					function(bit){
						return (bit.toString(16).slice(-2))
					}
				)
				return hexArr.join(' ').toUpperCase();
			},
			connect(){
				let _this=this;
				_this.flag[0]=true;
				_this.status='初始化蓝牙模块...'
				//打开蓝牙模块
				uni.openBluetoothAdapter({
					success:(res)=>{
						console.log(res);
						_this.status='初始化成功！'
						
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
								_this.status='开始搜索附近蓝牙设备...'
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
									_this.status='发现目标设备'
									_this.device_id=devices[i].deviceId;
									
									//连接指定设备
									uni.createBLEConnection({
										deviceId:_this.device_id,
										success: (res) => {
											_this.status='成功连接到设备！'
											console.log(res);
											
											//获取设备所有服务
											uni.getBLEDeviceServices({
												deviceId:_this.device_id,
												success: (res) => {
													console.log(res);
													_this.service_id=res.services[0].uuid
													
													//获取服务特征值
													let service_id=_this.service_id
													uni.getBLEDeviceCharacteristics({
														deviceId:_this.device_id,
														serviceId:_this.service_id,
														success: (res) => {
															console.log(res);
															_this.characteristic_id=res.characteristics[0].uuid
															
															//订阅指定特征值
															uni.notifyBLECharacteristicValueChange({
																deviceId:_this.device_id,
																serviceId:_this.service_id,
																characteristicId:_this.characteristic_id,
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
															
															//监听特征值变化
															uni.onBLECharacteristicValueChange(function(res){
																console.log(_this.ab2hex(res.value));
																_this.code=_this.ab2hex(res.value);
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
					},
					fail:(res)=>{
						console.log(res);
						_this.flag[0]=false;
					},
					complete:(res)=>{
						
					}
				})
			},
			
			sendC1(){
				//设定要发送的数据
				let code='C1 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00';
				
				//向蓝牙设备特征值中写入二进制数据
				uni.writeBLECharacteristicValue({
					deviceId:_this.device_id,
					serviceId:_this.service_id,
					characteristicId:_this.characteristic_id
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
