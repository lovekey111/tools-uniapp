<template>
	<view class="content">
		
		<navigator url="/homepackage/human-curve/human-curve" hover-class="navigator-hover">
			<image class="logo" src="/static/health_icon.png"></image>
		</navigator>
		<text class="text-area">{{title}}</text>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '人体三曲线'
			}
		},
		onLoad() {

		},
		methods: {
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 100rpx;
		width: 100rpx;
		margin-top: 50rpx;
		margin-bottom: 10rpx;
		margin-left: auto;
		margin-right: auto;
	}

	.text-area {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
