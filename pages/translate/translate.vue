<template>
	<view class="container">
		<image src="/static/noRecords.png" mode="aspectFill" class="img"></image>
		<view class="tips">
			<i class="iconfont">&#xe630;</i>将文章修改完善，更有逻辑性。
		</view>
		<view class="tip_botton">
			<button v-for="(item, index) of caseInfo" :key="index" type="primary" plain="true"
				@click="caseBtn(item.case)">{{item.title}}</button>
		</view>
		<view class="inp">
			<uni-easyinput @input="sumFontnum()" type="textarea" v-model="beforeData" maxlength=500
				placeholder="请输入文章内容" />
			<text class="font_num">{{fontNum}}/500</text>
		</view>
		<button type="primary" class="btn" @click="submit()">{{createText}}</button>
		<view class="form_essay" v-if="inpFlag">
			<view class="inp inp_btn">
				<uni-easyinput type="textarea" v-model="afterData" :maxlength='-1' />
			</view>
			<button type="primary" class="btn" @click="copyMassage()">复制</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				createText: '生成',
				beforeData: '',
				afterData: '',
				fontNum: 0,
				inpFlag: false,
				btnFlag: true,
				msgContent: [{
					role: "system",
					content: "润色此句"
				}],
				caseInfo: [{
						title: '示例1',
						case: '今天早上我七点钟起床，八点钟吃早餐，吃了两个包子一杯豆浆。九点上课，十二点下课吃午饭，然后睡觉到下午两点上课，六点钟下课。回宿舍玩游戏到晚上十一点，睡觉。'
					},
					{
						title: '示例2',
						case: '关于这个事，我简单说两句，你明白就行，总而言之，这个事呢，现在就是这个情况，具体的呢，大家也都看得到，也得出来说那么几句，可能，你听的不是很明白，但是意思就是那么个意思，不知道的你也不用去猜，这种事情见得多了，我只想说懂得都懂，不懂的我也不多解释，毕竟自己知道就好，细细品吧。'
					},
					{
						title: '示例3',
						case: '你是个明白人，我明白你明白的意思。我也是明白人，明白人就应该明白我明白你明白的意思。只要大家都明白明白人应该明白我明白你明白的。'
					}
				]
			};
		},
		onLoad() {},
		onShow() {
			console.log('转发成功!')
		},
		onShareAppMessage: (res) => {
			console.log('触发转发')
		},
		methods: {
			caseBtn(res) {
				this.beforeData = res
				this.fontNum = res.length
			},
			sumFontnum(e) {
				this.fontNum = e.length
			},
			copyMassage() { // 点击复制
				wx.setClipboardData({
					data: this.afterData,
					success(res) {
						wx.showToast({
							title: '复制成功',
							icon: "none",
							mask: "true" //是否设置点击蒙版，防止点击穿透
						})
					}
				})
			},
			// 发送
			submit() {
				if (this.btnFlag) {
					this.btnFlag = false
					this.createText = '生成中...'
					this.msgContent.push({
						"role": "user",
						"content": this.beforeData
					})
					uni.request({
						url: 'https://ryukrobot.ren/api/v1/chat/completions',
						method: 'POST',
						header: {
							'Content-Type': 'application/json',
							'Authorization': 'Bearer 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'
						},
						data: {
							// 问答
							model: 'gpt-3.5-turbo', // 模型
							temperature: 0, // 回答相似度
							messages: this.msgContent,
							presence_penalty: 1,
							top_p: 1,
							frequency_penalty: 1,
							max_tokens: 2048
						},
						success: (res) => {
							this.btnFlag = true
							this.msgContent = [{
								role: "system",
								content: "润色此句"
							}]
							this.createText = '生成'
							if (res.statusCode == 200) {
								this.afterData = res.data.choices[0].message.content
								this.inpFlag = true
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
							this.msgContent = [{
								role: "system",
								content: "润色此句"
							}]
							if (res.errMsg == 'request:fail timeout') {
								this.createText = '生成'
								this.btnFlag = true
								wx.showToast({
									title: '发送失败请重试~',
									icon: "none",
									mask: "true" //是否设置点击蒙版，防止点击穿透
								})
							}
						}
					})
				} else {
					wx.showToast({
						title: '生成中，请稍后~',
						icon: "none",
						mask: "true" //是否设置点击蒙版，防止点击穿透
					})
				}
			}
		},
	}
</script>
<style lang="less">
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;

		.img {
			width: 120px;
			height: 120px;
		}


		.tips {
			display: flex;
			opacity: 0.5;

			.iconfont {
				opacity: 1;
				font-size: 40rpx !important;
				margin-right: 10rpx;
				color: rgb(255, 111, 2);
			}
		}

		.tip_botton {
			display: flex;
			width: 100%;
			margin-top: 40rpx;

			button {
				font-size: 26rpx;
				width: 30%;
				color: orangered;
				border: 1px solid orangered;
			}

			&>button:nth-child(2) {
				color: royalblue;
				border: 1px solid royalblue;
			}

			&>button:nth-child(3) {
				color: limegreen;
				border: 1px solid limegreen;
			}
		}

		.inp {
			margin-top: 40rpx;
			width: 100%;
			position: relative;

			.font_num {
				position: absolute;
				right: 10rpx;
				bottom: 10rpx;
				opacity: 0.5;
			}
		}

		.btn {
			margin-top: 40rpx;
			border-radius: 0;
			width: 90%;
			background-color: #007aff;
		}

		.is-input-border {
			border-radius: 0;
		}

		.uni-easyinput__content-textarea {
			min-height: 140px;
		}

		.form_essay {
			width: 100%;

			.inp_btn {
				margin-top: 40rpx;
			}
		}
	}

	page {
		background-color: #ececec;
		height: 100%;
	}
</style>
