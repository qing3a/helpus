<template>
	<view class="content">
		<!-- 背景图，后台控制 -->
		<view class="zhuige-ranking-box"
			:style="background ? 'background: url(' + background + ') no-repeat center; background-size: cover;' : ''">

			<view class="zhuige-ranking">
				<view class="zhuige-block">
					<view class="zhuige-block-head">
						<view>{{title}}</view>
					</view>
					<view class="zhuige-ranking-info">{{desc}}</view>
				</view>
				<view class="zhuige-block">

					<zhuige-tab type="simple" :tabs="tabs" :cur-tab="cur_tab" @clickTab="clickTab"></zhuige-tab>

					<!-- 用户列表 -->
					<view v-if="cur_tab==0 || cur_tab==1 || cur_tab==2" class="zhuige-tab-block zhuige-ranking-list">

						<!-- 用户卡列表 -->
						<view v-for="(user, index) in users" :key="index"
							@click="openLink('/pages/user/home/home?user_id=' + user.user_id)"
							class="zhuige-social-poster-blcok zhuige-friends-block">
							<view class="zhuige-ranking-num">
								<text>{{index + 1}}</text>
							</view>
							<view class="zhuige-social-poster">
								<view class="zhuige-social-poster-avatar">
									<image mode="aspectFill" :src="user.avatar"></image>
								</view>
								<view class="zhuige-social-poster-info">
									<view>
										<text>{{user.nickname}}</text>
										<!-- 图2 认证-->
										<image v-if="user.certify && user.certify.status==1" mode="aspectFill"
											:src="user.certify.icon"></image>
										<!-- 图1 vip-->
										<image v-if="user.vip && user.vip.status==1" class="zhuige-social-vip"
											mode="aspectFill" :src="user.vip.icon"></image>
									</view>
								</view>
							</view>
							<view class="zhuige-social-opt">
								<text v-if="cur_tab==0">发帖数 {{user.num}}</text>
								<text v-if="cur_tab==1">评论数 {{user.num}}</text>
								<text v-if="cur_tab==2">点赞数 {{user.num}}</text>
							</view>
						</view>

					</view>

					<!-- 帖子列表 -->
					<view v-if="cur_tab==3" class="zhuige-tab-block zhuige-ranking-list">
						<view v-for="(topic, index) in topics" :key="index"
							@click="openLink('/pages/bbs/detail/detail?topic_id=' + topic.id)"
							class="zhugie-info-block left-side">
							<view class="zhuige-ranking-num">
								<text>{{index + 1}}</text>
							</view>
							<view class="zhugie-info-image">
								<image mode="aspectFill" :src="topic.thumb"></image>
							</view>
							<view class="zhugie-info-text">
								<view class="zhugie-info-title">{{topic.title}}</view>
								<view class="zhuige-info-data">
									<text>点赞 {{topic.like_count}}</text>
									<text>{{topic.time}}</text>
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>

		</view>
	</view>
</template>

<script>
	import Util from '@/utils/util';
	import Api from '@/utils/api';
	import Rest from '@/utils/rest';

	import ZhuigeTab from "@/components/zhuige-tab";

	export default {
		data() {
			return {
				background: '',
				title: '',
				desc: '',

				cur_tab: 0,
				tabs: [{
					id: 0,
					title: '发帖榜'
				}, {
					id: 1,
					title: '评论榜'
				}, {
					id: 2,
					title: '捧场榜'
				}, {
					id: 3,
					title: '热帖榜'
				}],

				topics: [],

				users: [],
			};
		},

		components: {
			ZhuigeTab,
		},

		onLoad(options) {
			Util.addShareScore(options.source);

			Rest.post(Api.URL('rank', 'setting'), {}).then(res => {
				this.background = res.data.background;
				this.title = res.data.title;
				this.desc = res.data.desc;
			}, err => {
				console.log(err)
			});

			this.loadData();
		},

		onShow() {
			// #ifdef MP-BAIDU
			swan.setPageInfo({
				title: '排行榜_' + getApp().globalData.appName,
				keywords: getApp().globalData.appKeywords,
				description: getApp().globalData.appDesc,
			});
			// #endif
		},

		onShareAppMessage() {
			return {
				title: '排行榜-' + getApp().globalData.appName,
				path: Util.addShareSource('pages/plugins/ranking/ranking?n=n')
			};
		},

		// #ifdef MP-WEIXIN
		onShareTimeline() {
			return {
				title: '排行榜-' + getApp().globalData.appName,
			};
		},
		// #endif

		methods: {
			/**
			 * 打开链接
			 */
			openLink(link) {
				Util.openLink(link)
			},

			/**
			 * 切换tab
			 */
			clickTab(tab) {
				this.cur_tab = tab.id;
				this.loadData();
			},

			/**
			 * 加载数据
			 */
			loadData() {
				if (this.cur_tab == 3) {
					this.loadTopics();
				} else {
					this.loadUsers();
				}
			},

			/**
			 * 加载帖子
			 */
			loadTopics() {
				if (this.topics.length > 0) {
					return;
				}

				Rest.post(Api.URL('rank', 'topic')).then(res => {
					this.topics = res.data.topics;
				}, err => {
					console.log(err)
				});
			},

			/**
			 * 加载用户
			 */
			loadUsers() {
				let params = {};
				if (this.cur_tab == 0) {
					params.order = 'chzdr';
				} else if (this.cur_tab == 1) {
					params.order = 'hdnsh';
				} else if (this.cur_tab == 2) {
					params.order = 'pchw';
				}

				Rest.post(Api.URL('rank', 'user'), params).then(res => {
					this.users = res.data.users;
				}, err => {
					console.log(err)
				});
			}
		}
	}
</script>

<style>
	.zhuige-ranking-box {
		position: fixed;
		height: 100%;
		width: 100%;
		overflow-y: scroll;
	}

	.zhuige-ranking {
		padding: 400rpx 20rpx 80rpx;
	}

	.zhuige-ranking-list .zhugie-info-image {
		flex: 0 0 180rpx;
		width: 180rpx;
		height: 140rpx;
	}

	.zhuige-ranking-list .zhugie-info-block {
		padding: 20rpx 0;
		border-bottom: 1rpx solid #EEEEEE;
	}

	.zhuige-ranking-list .zhugie-info-block:last-of-type {
		border: none;
	}

	.zhuige-ranking-num {
		width: 60rpx;
		text-align: center;
		padding-right: 20rpx;
	}

	.zhuige-ranking-num text {
		font-size: 36rpx;
		font-weight: 600;
		color: #999999;
	}

	.zhuige-ranking-list .zhugie-info-block:nth-child(1) .zhuige-ranking-num text,
	.zhuige-ranking-list .zhuige-friends-block:nth-child(1) .zhuige-ranking-num text {
		color: #FF6146;
	}

	.zhuige-ranking-list .zhugie-info-block:nth-child(2) .zhuige-ranking-num text,
	.zhuige-ranking-list .zhuige-friends-block:nth-child(2) .zhuige-ranking-num text {
		color: #1D9FFC;
	}

	.zhuige-ranking-list .zhugie-info-block:nth-child(3) .zhuige-ranking-num text,
	.zhuige-ranking-list .zhuige-friends-block:nth-child(3) .zhuige-ranking-num text {
		color: #F5AD1A;
	}

	.zhuige-social-poster {
		width: 70%;
	}

	.zhuige-social-opt {
		width: 30%;
	}

	.left-side .zhugie-info-text {
		width: 360rpx;
	}

	.zhugie-info-title {
		font-size: 28rpx;
	}

	.zhuige-info-data text {
		margin: 0 20rpx 0 -2rpx;
	}

	.zhuige-social-poster-blcok {
		padding: 20rpx 0;
		margin-bottom: 0;
	}

	.zhuige-friends-block .zhuige-social-opt text {
		font-weight: 400;
		padding-right: 20rpx;
	}
</style>