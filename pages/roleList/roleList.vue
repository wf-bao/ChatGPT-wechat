<template>
	<view class="container">
		<view class="select_btn">
			<button type="primary" :plain="numId == 0 ? false : true" size="mini" @click="changeType(0)">全部</button>
			<button v-for="(item, index) of dateInfo" :key="index" type="primary"
				:plain="item.id == numId ? false : true" size="mini"
				@click="changeType(item.id)">{{item.btnTitle}}</button>
		</view>
		<view v-if="viewAllFlag" class="role_list">
			<view class="title">
				<text>{{dateInfo[numId - 1].btnTitle}}</text>
			</view>
			<view class="care_list">
				<uni-card v-for="(item, index) of dateInfo[numId].careList" :key="index" :title=item.careTitle
					extra="复制">
					<text class="uni-body">{{item.careContent}}</text>
					<span class="copy" @click="copyMassage(item.careContent)"></span>
				</uni-card>
			</view>
		</view>
		<view v-else class="role_list select_all">
			<view v-for="(item, index) of dateInfo" :key="index">
				<text class="title">{{item.btnTitle}}</text>
				<uni-card v-for="(article, i) of item.careList" :key="i" :title=article.careTitle extra="复制">
					<text class="uni-body">{{article.careContent}}</text>
					<span class="copy" @click="copyMassage(article.careContent)"></span>
				</uni-card>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				viewAllFlag: false,
				numId: 0,
				dateInfo: [{
					id: 1,
					btnTitle: '写报告',
					careList: [{
						careTitle: '报告开头',
						careContent: '我现在正在 [报告的情境与目的]。我的简报主题是 [主题]，请提供 [数字] 种开头方式，要简单到 [目标] 能听懂，同时要足够能吸引人，让他们愿意专心听下去'
					}, {
						careTitle: '研究报告',
						careContent: '写出一篇有关 [知识] 的 [数字] 字研究报告，报告中需引述最新的研究，并引用专家观点'
					}, {
						careTitle: '提出反对观点',
						careContent: '你是 [某个主题] 的专家，请针对以下论述 [附上论述]，提出 [数字] 个反驳的论点，每个论点都要有佐证'
					}, {
						careTitle: '报告总结',
						careContent: '你是 [某个主题] 的专家，请总结以下内容，并针对以下内容提出未来能进一步研究的方向 [附上内容]'
					}]
				}, {
					id: 2,
					btnTitle: '程式',
					careList: [{
						careTitle: '写程式',
						careContent: '你现在是一个 [程式语言] 专家，请帮我用 [程式语言] 写一个函式，它需要做到 [某个功能]'
					}, {
						careTitle: '解读程式码',
						careContent: '你现在是一个 [程式语言] 专家，请告诉我以下的程式码在做什么。[附上程式码]'
					}, {
						careTitle: '重构程式码',
						careContent: '你现在是一个 Clean Code 专家，我有以下的程式码，请用更干净简洁的方式改写，让我的同事们可以更容易维护程式码。另外，也解释为什么你要这样重构，让我能把重构的方式的说明加到 Pull Request 当中。[附上程式码]'
					}, {
						careTitle: '写测试',
						careContent: '你现在是一个 [程式语言] 专家，我有一段程式码 [附上程式码]，请帮我写一个测试，请至少提供五个测试案例，同时要包含到极端的状况，让我能够确定这段程式码的输出是正确的。'
					}]
				}, {
					id: 3,
					btnTitle: '准备面试',
					careList: [{
						careTitle: '汇整面试题目',
						careContent: '你现在是 [公司] 的 [职位] 面试官，请分享在 [职位] 面试时最常会问的 [数字] 个问题。 '
					}, {
						careTitle: '给予回馈',
						careContent: '我针对 [问题] 的回答，有哪些可以改进的地方? [附上回答]'
					}, {
						careTitle: '提供追问的问题',
						careContent: '针对 [问题] 这个面试问题，请提供一些常见的追问面试题。 '
					}, {
						careTitle: '用 STAR 原则回答面试问题',
						careContent: '我在准备 [问题] 这个面试问题，请用 STAR 原则帮我回答这个问题。针对这个问题，我有的经历如下 [附上经历]。'
					}]
				}, {
					id: 4,
					btnTitle: '资料整理',
					careList: [{
						careTitle: '搜集资料',
						careContent: '给我 [数字] 篇，有关 [领域] 的文章。'
					}, {
						careTitle: '内容总结',
						careContent: '用列点的方式总结出这篇文章的 [数字] 个重点：[附上文章内容/附上文章网址]。'
					}, {
						careTitle: '摘录某领域重点',
						careContent: '用列点的方式总结出 [数字] 个 [领域] 知识重点'
					}]
				}, {
					id: 5,
					btnTitle: '知识学习',
					careList: [{
						careTitle: '概念解说',
						careContent: '详细的说明 [填入想了解的知识]'
					}, {
						careTitle: '简易教学',
						careContent: '你扮演 [科目老师] 的角色， 我需要理解 [理论]。请用 [方式] 方式描述。'
					}, {
						careTitle: '教学与测验',
						careContent: '教我 [二次方程式]，最后给我一个测验'
					}]
				}, {
					id: 6,
					btnTitle: '英语学习',
					careList: [{
						careTitle: '背单字法宝',
						careContent: '用 [中文/英文] 解释以下英文单字：[填入一个或多个单字]。请用表格的方式呈现，并且表格内须包含单字、词性、解释与例句。'
					}, {
						careTitle: '英语单字学习',
						careContent: '解释英文单字 [英文单字]，并且给我 [数字] 个常用句子。'
					}, {
						careTitle: '英语对话',
						careContent: 'Can we have a conversation about [话题]?'
					}, {
						careTitle: '校阅英文文法',
						careContent: 'Can we have a conversation about [话题]?'
					}, {
						careTitle: '英文作文修改与解释',
						careContent: '校阅以下英文文章，并用表格的方式呈现，要有三个栏位，分别是原文、修正后的版本，以及用中文详解为什么要这样修改：[附上英文文章]'
					}, {
						careTitle: '纠正文法和拼字错误',
						careContent: 'Please correct my grammar and spelling mistakes in the text above: [附上英文文字]'
					}]
				}, {
					id: 7,
					btnTitle: '写作帮手',
					careList: [{
						careTitle: '撰写标题',
						careContent: '写出 [数字] 个有关 [主题] 的 [社群平台] 风格标题，要遵守以下规则：[规则 1]、[规则 2]、[其他规则]。'
					}, {
						careTitle: '撰写文章大纲',
						careContent: '提供 [某主题] 主题的文章大纲'
					}, {
						careTitle: '文章撰写',
						careContent: '针对 [主题] 这个主题生成一篇文章'
					}, {
						careTitle: '产品文案',
						careContent: '将以下产品关键字生成 [数字] 句的产品文案。产品关键字：[附上关键字...]'
					}]
				}, {
					id: 8,
					btnTitle: '日常生活',
					careList: [{
						careTitle: '活动计划清单',
						careContent: '你扮演一位专业的活动企划，请生成 [活动] 活动计划清单，包括重要任务和截止日期。'
					}, {
						careTitle: '旅游计划',
						careContent: '生成一份 [数字] 天的 [地点] 旅游计画，交通工具是 [交通工具...]。要遵守以下规则：[填入规则]'
					}]
				}, {
					id: 9,
					btnTitle: '有趣好玩',
					careList: [{
						careTitle: '写歌词',
						careContent: '大家都说我写的歌词像 [人名]，但我最近有点没灵感，请帮我用 [人名] 的风格写一首歌。歌中包含的元素要 [关键字...]。'
					}, {
						careTitle: '写故事',
						careContent: '写出一篇有关 [故事想法]，拥有 [风格] 风格的短篇故事'
					}, {
						careTitle: '写Rap',
						careContent: '你是现在最红的饶舌歌手，请创作一首 Rap，主题是 [附上主题]。'
					}]
				}, {
					id: 10,
					btnTitle: '角色扮演',
					careList: [{
						careTitle: '综合情景',
						careContent: '你现在是一名 [角色]，你要针对我提出的问题提供建议。我的问题是：[附上问题]。'
					}, {
						careTitle: '面试官',
						careContent: '你现在是一个 [职位] 面试官，而我是要应征 [职位] 的面试者。你需要遵守以下规则：1. 你只能问我有关 [职位] 的面试问题。2. 不需要写解释。3. 你需要向面试官一样等我回答问题，再提问下一个问题。我的第一句话是，你好。'
					}, {
						careTitle: '担任导游',
						careContent: '你是一位导游，我会把我旅游的位置给你，你要推荐一个靠近我位置的地方。在某些情况下，我还会告诉您我想旅游地点的类型。你还会向我推荐靠近我的第一个位置的类似类型的地方。我的第一个需求是[填入需求]'
					}]
				}]
			};
		},
		methods: {
			changeType(id) { // 切换列表
				this.numId = id
				if (id == 0) {
					this.viewAllFlag = false
				} else {
					this.viewAllFlag = true
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
		},
	}
</script>

<style lang="less">
	.container {
		padding: 0 5%;
		margin-bottom: 3rem;

		.select_btn {
			margin-top: 1rem;

			button {
				padding: 6px 14px;
				line-height: normal;
				font-size: 16px;
				margin-right: 22rpx;
			}
		}

		.role_list {
			margin-top: 1rem;

			.title {
				text-align: center;
				position: relative;

				&::after {
					position: absolute;
					bottom: -4px;
					left: 50%;
					display: block;
					content: "";
					border-bottom: 1px solid #48BB78;
					width: 50px;
					transform: translateX(-50%);
				}
			}

			.care_list {
				position: relative;

				.copy {
					position: absolute;
					display: block;
					width: 70rpx;
					height: 48rpx;
					background-color: #ccc;
					right: 28rpx;
					top: 16rpx;
					opacity: 0;
				}
			}
		}

		.select_all {
			text-align: center;

			uni-card {
				text-align: initial;
				position: relative;
				
				.copy {
					position: absolute;
					display: block;
					width: 70rpx;
					height: 48rpx;
					background-color: #ccc;
					right: 28rpx;
					top: 16rpx;
					opacity: 0;
				}
			}
		}
	}
</style>
