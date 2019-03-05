<template>
	<view class="content">
		<view class="title">{{title}}</view>
		<view class="content">
			<rich-text class="richText" :nodes="strings"></rich-text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title:'',
				strings:''
			};
		},
		onLoad:function(e){
			uni.showLoading({
				title: '加载中...',
				mask: false
			});
			uni.request({
				url: 'https://unidemo.dcloud.net.cn/api/news/36kr/'+e.newsid,
				method: 'GET',
				data: {},
				success: res => {
					this.title=res.data.title;
					this.strings=res.data.content;
					uni.hideLoading(); 
				},
				fail: () => {},
				complete: () => {}
			});
		}
	}
</script>

<style>
.content{
	padding: 20upx 2%;
	width: 96%;
	flex-wrap: wrap;
}
.title{
	line-height: 2em;
	font-weight: 700;
	font-size: 38upx;
}
</style>
