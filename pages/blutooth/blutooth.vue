<template>
	<view>
		<view class="uni-padding-wrap uni-common-mt">
			<view class="">
				<p>当前蓝牙设备：{{device_num}}</p>
				<scroll-view class="scroll-Y" scroll-y="true">
					<view class="scroll-view-item uni-bg-red uni-common-mt" 
					v-for="(item,index) in devices" 
					style="width: 80%;" @click="test(item,index)">
						<p>设备名:{{item.name}}</p>
						<p>设备ID:{{item.id}}</p>
					</view>
				</scroll-view>
			</view>
			<view class="">
				<p>当前状态:{{status}}</p>
				<button type="primary" @click="scan" :disabled="flag[0]" style="width: 100%;background-color: #1E9FFF;">开始扫描</button>
				<!-- <button class="uni-common-mt" type="default" @click="sendC1" :disabled="flag[1]">发送C1</button>
				<button class="uni-common-mt" type="default" @click="sendC2" :disabled="flag[2]">发送C2</button> -->
			</view>
		</view>
	</view>
</template>

<script>
	import {uniBadge} from '@dcloudio/uni-ui';
	import uniCard from "@dcloudio/uni-ui/lib/uni-card/uni-card.vue"

	export default {
		components: {uniBadge,uniCard},
		
		data() {
			return {
				device_num:0,
				devices:[],
				status:'无',
				c0:'无数据',
				c1:'无数据',
				c2:'无数据',
				flag:[false,true,true],
				device_id:'',
				service_id:'',
				characteristic_id:''
			};
		},
		
		methods:{
			/**
			 * arraybuffer转字符串方法
			 */
			ab2hex(buffer){
				var hexArr=Array.prototype.map.call(
					new Uint8Array(buffer),
					function(bit){
						return (bit.toString(16).slice(-2))
					}
				)
				return hexArr.join(' ').toUpperCase();
			},
			
			/**
			 * 数组转arraybuffer方法
			 */
			arr2ab(arr){
				let arrayBuffer=new Uint8Array(arr).buffer
				return arrayBuffer;
			},
			
			/**
			 * 测试方法
			 */
			test(item,index){
				uni.showModal({
					title:'信息',
					content:index+'\n'+item.id+'\n'+item.name
				})
			},
			
			/**
			 * 扫描设备
			 */
			scan(){
				let _this=this;
				_this.flag[0]=true;
				_this.status='初始化蓝牙模块...'
				
				//打开蓝牙模块
				uni.openBluetoothAdapter({
					success: (res) => {
						_this.status='初始化成功！'
						
						//搜索蓝牙设备
						uni.startBluetoothDevicesDiscovery({
							services:[],
							success: (res) => {
								_this.status='开始搜索附近蓝牙设备...'
							},
							fail: (res) => {
								
							}
						})
						
						//监听搜索到的蓝牙设备
						uni.onBluetoothDeviceFound((res)=>{
							let devices=res.devices;
							for(let i=0;i<devices.length;i++){
								// console.log(devices[i]);
								_this.devices.push({name:devices[i].localName,id:devices[i].deviceId});
								_this.device_num++;
							}
						})
						
						//10秒后停止蓝牙搜索
						setTimeout(function(){
							uni.stopBluetoothDevicesDiscovery({
								success:(res)=>console.log(res),
								fail:(res)=>console.log(res)
							});
						},10000)
						
					},
					fail: (res) => {
						
					}
				})
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
															
															//允许发送C1、C2命令
															_this.flag[1]=false;
															_this.flag[2]=false;
															
															//监听特征值变化
															uni.onBLECharacteristicValueChange(function(res){
																let code=_this.ab2hex(res.value);
																console.log(code);
																switch(code.substring(0,2)){
																	case 'C0':
																	_this.c0=code;
																	break;
																	case 'C1':
																	_this.c1=code;
																	break;
																	case 'C2':
																	_this.c2=code;
																	break;
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
				let arrC1=['0xC1','00','00','00','00','00','00','00','00','00','00','00','00','00','00','00','00','00']
				
				//字符串转arraybuffer
				let ab=this.arr2ab(arrC1);
				
				let _this=this;
				
				//向蓝牙设备特征值中写入二进制数据
				uni.writeBLECharacteristicValue({
					deviceId:this.device_id,
					serviceId:this.service_id,
					characteristicId:this.characteristic_id,
					value:ab,
					success: (res) => {
						console.log(res);
					},
					fail: (res) => {
						console.log(res);
					}
				})
			},
			
			sendC2(){
				//设定要发送的数据
				let arrC2=['0xC2','00','00','00','00','00','00','00','00','00','00','00','00','00','00','00','00','00']
				
				//字符串转arraybuffer
				let ab=this.arr2ab(arrC2);
				
				let _this=this;
				
				//向蓝牙设备特征值中写入二进制数据
				uni.writeBLECharacteristicValue({
					deviceId:this.device_id,
					serviceId:this.service_id,
					characteristicId:this.characteristic_id,
					value:ab,
					success: (res) => {
						console.log(res);
					},
					fail: (res) => {
						console.log(res);
					}
				})
			}
		}
	}
</script>

<style>
	page{
		background-color: #F4F5F6;
	}
	view{
		display: flex;
		flex-direction: column;
	}
	.scroll-Y{
		height: 550upx;
		background-color: #FFFFFF;
	}
</style>
