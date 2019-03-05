<template>
	<view class="content">
		<view class="uni-text">
			<button type="default" @click="blue">蓝牙</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			};
		},
		methods:{
			blue(){
				uni.openBluetoothAdapter({
				  success(res) {
					// console.log(res)
					uni.showToast({
						title:'成功调用',
						icon:'success'
					})
				  },
				  fail() {
				  	uni.showToast({
				  		title:'调用失败，',
						icon:'none'
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
