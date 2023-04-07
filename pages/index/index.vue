<template>
	<view class="content">
		<!-- 自定义nav -->
		<uni-nav-bar class="custom_nav" dark :fixed="true" shadow background-color="#fff" color="black" status-bar
			left-icon="left" :title="defaultData.navTitle" @clickLeft="back" />
		<!-- 聊天界面 -->
		<view class="list" id="chatWindow">
			<block v-for="(item,index) in list" :key="index">
				<view v-if="item.f" class="item-f">
					<i v-if="defaultData.aiImg == ''" class="iconfont" @click="openPopTop()">&#xe6c3;</i>
					<image v-else :src="defaultData.aiImg" mode="aspectFill" @click="open()"></image>
					<view class="mark_down" @click="copyMassage(item.text)">
						<zero-markdown-view :themeColor="themeColor" :markdown="item.text"></zero-markdown-view>
					</view>
				</view>
				<view v-else class="item-r">
					<view @click="copyMassage(item.text)">{{item.text}}</view>
					<image :src="defaultData.userImg == '' ? '/static/img.jpg' : defaultData.userImg" mode="aspectFill">
					</image>
				</view>
			</block>
		</view>
		<!-- 普通弹窗 -->
		<uni-popup ref="popup" background-color="#fff" class="pop">
			<view class="popup-content">
				<uni-forms ref="baseForm" :modelValue="defaultData" :label-position="alignment">
					<uni-forms-item label="Ai名称">
						<uni-easyinput v-model="defaultData.navTitle" />
					</uni-forms-item>
					<uni-forms-item label="Ai头像">
						<i v-if="defaultData.aiImg == ''" class="iconfont setIcon" @click="getToBase64(0)">&#xe6c3;</i>
						<image v-else :src="defaultData.aiImg" mode="aspectFill" class="setImg" @click="getToBase64(0)">
						</image>
					</uni-forms-item>
					<uni-forms-item label="用户头像">
						<image :src="defaultData.userImg == '' ? '/static/img.jpg' : defaultData.userImg"
							mode="aspectFill" class="setImg" @click="getToBase64(1)"></image>
					</uni-forms-item>
				</uni-forms>
				<view class="button-group">
					<button type="primary" size="mini" @click="reset()">重置</button>
					<button type="primary" size="mini"
						@click="$refs.popup1.open('left'), preserveRecords()">聊天记录</button>
					<b<button type="primary" size="mini" @click="closePopTop()">完成</button>
				</view>
			</view>
		</uni-popup>
		<!-- 聊天记录弹窗 -->
		<uni-popup ref="popup1" background-color="#fff">
			<view class="pop_left">
				<button type="primary" plain style="color: #fff;border-color: #fff;" @click="newRecordsBtn()">+ New
					chat</button>
				<view v-for="(item, index) of chatRecords" :key="index" class="toke_list" @click="changeRecord(index)">
					<i class="iconfont icon2">&#xe635;</i>
					<view class="title">
						{{item.msgContent[0].content}}
					</view>
				</view>
				<view class="clare_record" @click="clearRecord()">
					<i class="iconfont icon2">&#xe646;</i>清空聊天记录
				</view>
			</view>
		</uni-popup>
		<!-- 发送输入框 -->
		<view class="input" :style="{position: 'fixed',bottom:inputHeight+'px'}">
			<textarea @blur="closeFingerboard" @focus="openFingerboard" v-model="text" :fixed='true' :auto-height='true'
				maxlength=150 :adjust-position='false' :auto-blur='true' :show-confirm-bar="false"></textarea>
			<view @click="submit">发送</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				apiUrl: '', // 必填 后端转发地址
				apiKey: '', // 必填 你的apikey
				startChatFlag: false,
				changeFlag: false,
				newRecords: {},
				chatRecords: [],
				inputHeight: 0, // 软键盘的高度
				defaultData: { // 初始配置
					navTitle: '若菈',
					aiImg: '',
					userImg: ''
				},
				titleFlag: true,
				list: [{
					f: true,
					text: '您好，我是若菈，你可以问我任何问题。点击头像可配置相关信息'
				}],
				text: '',
				msgContent: []
			}
		},
		onLoad() {
			if (wx.getStorageSync("imgUrl")) {
				this.defaultData.aiImg = wx.getStorageSync("imgUrl")
			}
			if (wx.getStorageSync("userImg")) {
				this.defaultData.userImg = wx.getStorageSync("userImg")
			}
			if (wx.getStorageSync("chatRecords")) {
				this.chatRecords = wx.getStorageSync("chatRecords")
			}
			wx.showShareMenu({
				withShareTicket: true,
				menus: ['shareAppMessage', 'shareTimeline'],
			})
		},
		onUnload() {
			this.preserveRecords()
		},
		onHide() {
			this.preserveRecords()
		},
		methods: {
			newRecordsBtn() { // 新聊天记录
				this.$refs.popup.close()
				this.$refs.popup1.close()

				this.list = []
				this.msgContent = []
			},
			changeRecord(i) { // 切换聊天记录
				this.$refs.popup.close()
				this.$refs.popup1.close()

				this.newRecords = {}
				this.list = this.chatRecords[i].list
				this.msgContent = this.chatRecords[i].msgContent

				this.newRecords.list = this.chatRecords[i].list
				this.newRecords.msgContent = this.chatRecords[i].msgContent
				this.chatRecords.unshift(this.newRecords) // 将此聊天记录放入第一位

				this.chatRecords.splice(i + 1, 1)
				this.changeFlag = true
				this.moveTo()

				if (this.startChatFlag) {
					wx.setStorage({
						key: "chatRecords",
						data: this.chatRecords
					})
					this.startChatFlag = false
				}
			},
			preserveRecords() { // 保存聊天记录
				if (this.startChatFlag) {
					let records = {}
					records.list = this.list
					records.msgContent = this.msgContent
					if (this.changeFlag) {
						this.changeFlag = false
					} else {
						this.chatRecords.unshift(records) // 将此聊天记录放入第一位
					}
					if (this.chatRecords.length > 3) { // 长度大于3时删除
						this.chatRecords.pop()
					}
					wx.setStorage({
						key: "chatRecords",
						data: this.chatRecords
					})
					this.startChatFlag = false
				}
			},
			clearRecord() { // 清空聊天记录
				this.list = []
				this.newRecords = {}
				this.chatRecords = []
				uni.removeStorageSync("chatRecords")
				this.$refs.popup.close()
				this.$refs.popup1.close()
			},
			moveTo() {
				setTimeout(() => {
					uni.createSelectorQuery().in(this).select('#chatWindow').fields({
						size: true
					}, data => {
						uni.pageScrollTo({
							scrollTop: data.height,
							duration: 300
						});
					}).exec()
				}, 100)
			},
			closeFingerboard() {
				this.inputHeight = 0
			},
			openFingerboard(e) {
				if (e.detail.height) {
					this.inputHeight = e.detail.height //这个高度就是软键盘的高度
				}
			},
			copyMassage(msg) { // 点击复制
				wx.setClipboardData({
					data: msg,
					success(res) {
						wx.showToast({
							title: '复制成功',
							icon: "none",
							mask: "true" //是否设置点击蒙版，防止点击穿透
						})
					}
				})
			},
			reset() { // 重置修改
				this.defaultData = {
					navTitle: '若菈',
					aiImg: '',
					userImg: ''
				}
				uni.removeStorageSync("imgUrl")
				uni.removeStorageSync("userImg")
			},
			closePopTop() { // 弹窗
				this.$refs.popup.close()
			},
			openPopTop() {
				// 通过组件定义的ref调用uni-popup方法 ,如果传入参数 ，type 属性将失效 ，仅支持 ['top','left','bottom','right','center']
				this.$refs.popup.open('top')
			},
			getToBase64(type) { // 图片转base64
				wx.chooseImage({
					count: '1', // 最多可以选择的图片张数
					sizeType: ['original', 'compressed'], // ['原图','压缩图']
					sourceType: ['album', 'camera'], // ['从相册选图','使用相机']
					success: res => {
						wx.getFileSystemManager().readFile({
							filePath: res.tempFilePaths[0], //选择图片返回的相对路径
							encoding: 'base64', //编码格式
							success: res => { //成功的回调
								if (type) { // 判断是用户还是ai头像
									this.defaultData.userImg = 'data:image/png;base64,' + res.data
									wx.setStorage({
										key: "userImg",
										data: this.defaultData.userImg
									})
								} else {
									this.defaultData.aiImg = 'data:image/png;base64,' + res.data
									wx.setStorage({
										key: "imgUrl",
										data: this.defaultData.aiImg
									})
								}
							}
						})
					}
				})
			},
			back() { // 返回
				uni.navigateBack({
					delta: 0
				})
			},
			// 发送
			submit() {
				if (this.titleFlag) {
					let obj = {
						f: false,
						text: this.text
					}
					this.list.push(obj)
					this.msgContent.push({
						"role": "user",
						"content": this.text
					})
					this.defaultData.navTitle = '对方正在输入中...'
					this.titleFlag = false
					this.text = ''
					this.moveTo()
					uni.request({
						url: this.apiUrl,
						method: 'POST',
						header: {
							'Content-Type': 'application/json',
							'Authorization': 'Bearer ' + this.apiKey
						},
						data: {
							// 问答
							model: 'gpt-3.5-turbo', // 模型
							temperature: 0.9, // 回答相似度
							messages: this.msgContent,
							presence_penalty: 1,
							stream: false,
							temperature: 0.5,
							top_p: 0.8,
							frequency_penalty: 0,
							max_tokens: 2000
						},
						success: (res) => {
							console.log('success', res);
							let str = res.data.choices[0].message
							str.content = str.content.replace(/chatgpt|OpenAI/gi, "人工智能")
							str.content = str.content.replace(/GPT/gi, "AI")
							this.titleFlag = true
							if (res.statusCode == 200) {
								this.msgContent.push(str)
								let obj = {
									f: true,
									text: str.content
								}
								this.list.push(obj)
								this.moveTo()
							} else {
								setTimeout(() => {
									uni.showToast({
										title: '发送失败请重试~'
									})
								}, 300)
							}
						},
						complete: (res) => {
							console.log('complete', res)
							this.startChatFlag = true
							if (res.errMsg == 'request:fail timeout') {
								this.titleFlag = true
								wx.showToast({
									title: '发送失败请重试~',
									icon: "none",
									mask: "true" //是否设置点击蒙版，防止点击穿透
								})
							}
							this.defaultData.navTitle = 'ChatAI'
						}
					})
				} else {
					wx.showToast({
						title: '对方正在输入',
						icon: "none",
						mask: "true" //是否设置点击蒙版，防止点击穿透
					})
				}
			}
		}
	}
</script>

<style lang="scss" scoped>
	.list {
		width: 710rpx;
		padding: 20rpx;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		margin-bottom: 4.5rem;

		.item-f {
			display: flex;
			margin-bottom: 20rpx;

			image {
				height: 88rpx;
				width: 88rpx;
				border-radius: 8rpx;
				margin-right: 20rpx;
			}

			.mark_down {
				zero-markdown-view {
					background-color: #fff;
					font-size: 34rpx;
					color: #333;
					border-radius: 12rpx;
					display: flex;
					align-items: center;
					justify-content: center;
					max-width: 450rpx;

					/deep/ .mark-content {
						padding: 0;
						max-width: 450rpx;
					}
				}
			}


			view {
				background-color: #fff;
				padding: 10rpx 20rpx;
				font-size: 34rpx;
				color: #333;
				border-radius: 12rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				max-width: 450rpx;
			}
		}

		.item-r {
			display: flex;
			justify-content: flex-end;
			margin-left: 108rpx;
			margin-bottom: 20rpx;

			image {
				height: 88rpx;
				width: 88rpx;
				border-radius: 8rpx;
				margin-left: 20rpx;
			}

			view {
				background-color: #95EC69;
				padding: 10rpx 20rpx;
				font-size: 34rpx;
				color: #333;
				border-radius: 12rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				max-width: 455rpx;
			}
		}
	}

	.input {
		background-color: #f6f6f6;
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
		padding-bottom: 40rpx;
		display: flex;
		align-items: center;
		border-top: 2rpx solid #ECECEC;
		padding-top: 20rpx;

		textarea {
			width: 450rpx;
			background-color: #FEFEFE;
			font-size: 28rpx;
			color: #666;
			padding: 20rpx 10rpx;
			border-radius: 12rpx;
			margin: 0 40rpx;
		}

		view {
			background-color: #E9E9E9;
			font-size: 32rpx;
			font-weight: bold;
			padding: 14rpx 60rpx;
			color: #07C160;
			border-radius: 12rpx;
		}

		view:active {
			background-color: #D2D2D2;
		}
	}

	// 弹窗
	.popup-content {
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
		padding: 15px;
		height: 50vh;
		background-color: #fff;

		.setImg {
			height: 88rpx;
			width: 88rpx;
			border-radius: 8rpx;
			margin-left: 20rpx;
		}

		.text {
			font-size: 12px;
			color: #333;
		}

		.button-group {
			margin-top: 15px;
			display: flex;
			width: 80vw;
			justify-content: space-evenly;
		}
	}

	.pop_left {
		width: 48vw;
		margin-top: 92px;
		padding-top: 6px;
		background-color: #202123;
		height: 100%;
		color: #fff;
		position: relative;

		button {
			margin: 6px 8px 8px 8px;
		}

		.toke_list {
			display: flex;
			margin-top: 10px;
			line-height: 35px;

			&:hover {
				background-color: rgba(52, 53, 65, 1);
			}

			.title {
				white-space: nowrap;
				overflow: hidden;
				text-overflow: ellipsis;
			}
		}

		.clare_record {
			display: flex;
			position: absolute;
			bottom: 16rem;
		}
	}


	// icon
	.iconfont {
		background-image: linear-gradient(90deg, #e9737e 0, #ec5765);
		font-size: 52rpx !important;
		color: #fff !important;
		width: 48rpx;
		height: 68rpx;
	}

	.setIcon {
		background-image: linear-gradient(90deg, #e9737e 0, #ec5765);
		font-size: 52rpx !important;
		color: #fff !important;
		width: 70rpx;
		height: 70rpx;
	}

	.icon2 {
		background-image: initial;
		color: #fff;
	}
</style>
<style>
	page {
		background-color: #EDEDED;
	}
</style>
