<template>
	<view class="content">
		<!-- <view class="uni-list">
			<view class="uni-list-cell" hover-class="uni-list-cell-hover" v-for="(item,index) in news" :key="index" 
			@tap="openinfo" :data-newsid="item.post_id">
				<view class="uni-media-list">
					<image class="uni-media-list-logo" :src="item.author_avatar"></image>
					<view class="uni-media-list-body">
						<view class="uni-media-list-text-top">{{item.title}}</view>
						<view class="uni-media-list-text-bottom uni-ellipsis">{{item.created_at}}</view>
					</view>
				</view>
			</view>
		</view> -->
		<view class="text-box">
			<text>名称：</text>
			<text>{{name}}</text>
		</view>
		<view class="text-box">
			<text>型号：</text>
			<text>{{model}}</text>
		</view>
		<view class="text-box">
			<text>序列号：</text>
			<text>{{serial}}</text>
		</view>
		<view class="uni-text">
			<button type="primary" @click="scan">扫码获取产品信息</button>
		</view>
	</view>
</template>

<script>
	export default{
		data() {
			return {
				// news:[],
				name:'',
				model:'',
				serial:''
			}
		},
		onLoad:function(e){
// 			uni.request({
// 				url: 'https://unidemo.dcloud.net.cn/api/news',
// 				method: 'GET',
// 				data: {},
// 				success: res => {
// 					this.news=res.data;
// 				},
// 				fail: () => {},
// 				complete: () => {}
			// });
		},
		methods: {
// 			openinfo(e) {
// 				var newsid=e.currentTarget.dataset.newsid;
// 				console.log(newsid);
// 				uni.navigateTo({
// 					url: '../info/info?newsid='+newsid
// 				});
// 			},
			scan:
				function() {
					//将this保存在变量中
					let _this=this;
					
					uni.scanCode({
						success: function (res) {
							console.log('条码类型：' + res.scanType);
							console.log('条码内容：' + res.result);
							_this.name=res.result
						}
					});
				}
		},
	}
</script>
	
<style>
	.uni-media-list-body{
		height: auto;
	}
	.uni-media-list-text-top{
		line-height: 1.6em;
	}
	.uni-text{
		padding: 10upx 2%;
	}
	.text-box{
		padding: 5upx 0 5upx 50upx;
	}
</style>
