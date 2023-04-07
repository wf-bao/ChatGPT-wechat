<template>
	<view class="home">
		<ul>
			<li @click="goChat()">
				<view class="list">
					<i class="iconfont">&#xe6c3;</i>
					<text>聊天</text>
				</view>
			</li>
			<li @click="goTranslate()">
				<view class="list">
					<i class="iconfont set_icon">&#xe610;</i>
					<text>文章润色</text>
				</view>
			</li>
			<li @click="getMoreInfo()">
				<view class="list">
					<i class="iconfont">&#xe73a;</i>
					<text>更多</text>
				</view>
			</li>
		</ul>
		<view @click="goFingerpost()" class="fingerpost">
			指令大全
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {

			};
		},
		onLoad() {
			},
		onLaunch: function() {
			// 设置分享时带上分享信息
			wx.showShareMenu({
				withShareTicket: true
			})
		},
		onShareAppMessage(res) {
			return {
				title: '问答小程序',
				path: '/pages/hoem1/hoem1', // 转发的页面路径
				success: res => {},
				fail: () => {
					console.log('--- 转发失败 ---', path);
				}
			}
		},
		methods: {
			goChat() {
				uni.navigateTo({
					url: '/pages/index/index'
				})
			},
			goTranslate() {
				uni.navigateTo({
					url: '/pages/translate/translate'
				})
			},
			goFingerpost() {
				uni.navigateTo({
					url: '/pages/roleList/roleList'
				})
			},
			getMoreInfo() {
				wx.showToast({
					title: '更多功能正在开发中~',
					icon: "none",
					mask: "true" //是否设置点击蒙版，防止点击穿透
				})
			}
		},
	}
</script>

<style lang="less">
	.home {
		background-color: #1d2340;
		height: 100%;

		ul {
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: flex-start;
			padding-top: 6rem;

			li {
				width: 60vw;
				height: 20vw;
				background-image: linear-gradient(90deg, #e9737e 0, #ec5765);
				border-radius: 20rpx;
				text-align: center;
				line-height: 20vw;
				margin: 1rem;

				.list {
					display: flex;
					justify-content: center;
					color: #fff;
					font-size: 1.5rem;
				}
			}

			&>li:nth-child(2) {
				background-image: linear-gradient(90deg, #946de1 0, #7c48e7);
			}

			&>li:nth-child(3) {
				background-image: linear-gradient(90deg, #6a9cff 0, #2a6ff9);
			}
		}

		.fingerpost {
			position: absolute;
			color: #ccc;
			top: 1rem;
			right: .5rem;
		}
	}

	.set_icon {
		font-size: 2rem;
	}

	page {
		background-color: #f8f8f8;
		height: 100%;
	}
</style>
