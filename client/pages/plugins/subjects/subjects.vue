<template>
	<view class="content">
		<view class="zhuige-converge">
			<zhuige-scroll-ad v-if="recs" ext-ad-class="zhuige-scroll-coterie" title="话题推荐" :items="recs">
			</zhuige-scroll-ad>
		</view>

		<view class="zhuige-block">
			<view class="zhuige-block-head">
				<view>热门话题（TOP {{hots.length}}）</view>
			</view>
			<view class="zhuige-topic">
				<view v-for="(subject, index) in hots" :key="subject.id" @click="clickSubject(subject.id)"
					class="zhuige-topic-block">
					<view class="zhuige-topic-info">
						<view>
							<image mode="aspectFill" :src="subject.image"></image>
						</view>
						<view>
							<view>#{{subject.name}}</view>
							<view>{{subject.count}}个作品</view>
						</view>
					</view>
					<view class="zhuige-classify-join">
						<text>讨论</text>
					</view>
				</view>
			</view>
		</view>

		<view class="zhuige-block">
			<view class="zhuige-block-head">
				<view>话题（总计 {{alls.length}} 个话题）</view>
			</view>
			<view class="zhuige-topic">
				<view v-for="(subject, index) in alls" :key="subject.id" @click="clickSubject(subject.id)"
					class="zhuige-topic-block">
					<view class="zhuige-topic-info">
						<view>
							<image mode="aspectFill" :src="subject.image"></image>
						</view>
						<view>
							<view>#{{subject.name}}</view>
							<view>{{subject.count}}个作品</view>
						</view>
					</view>
					<view class="zhuige-classify-join">
						<text>讨论</text>
					</view>
				</view>
			</view>
		</view>

	</view>
</template>

<script>
	import Util from '@/utils/util';
	import Alert from '@/utils/alert';
	import Api from '@/utils/api';
	import Rest from '@/utils/rest';

	import ZhuigeScrollAd from "@/components/zhuige-scroll-ad";

	export default {
		data() {
			return {
				hots: [],
				recs: [],
				alls: [],
			}
		},

		components: {
			ZhuigeScrollAd,
		},

		onLoad(options) {
			Util.addShareScore(options.source);

			this.loadSetting();

			uni.showTabBarRedDot({
				index: 3
			})
		},

		onShareAppMessage() {
			return {
				title: '话题聚合-' + getApp().globalData.appName,
				path: Util.addShareSource('pages/plugins/subjects/subjects?n=n')
			};
		},

		// #ifdef MP-WEIXIN
		onShareTimeline() {
			return {
				title: '话题聚合-' + getApp().globalData.appName,
			};
		},
		// #endif

		methods: {
			/**
			 * 打开话题列表
			 */
			clickSubject(subject_id) {
				Util.openLink('/pages/bbs/list/list?subject_id=' + subject_id);
			},

			/**
			 * 加载配置
			 */
			loadSetting() {
				Rest.post(Api.URL('subject', 'setting')).then(res => {
					this.hots = res.data.hots;
					this.recs = res.data.recs;
					this.alls = res.data.alls;

					this.subjects.forEach((subject) => {
						this.hots.forEach((ele) => {
							if (ele.name == subject) {
								ele.class = 'active';
							}
						})

						this.recs.forEach((ele) => {
							if (ele.name == subject) {
								ele.class = 'active';
							}
						})

						this.alls.forEach((ele) => {
							if (ele.name == subject) {
								ele.class = 'active';
							}
						})
					})
				}, err => {
					console.log(err)
				});
			},
		}
	}
</script>

<style>
	.content {
		padding: 20rpx;
	}

	.content {
		padding: 20rpx;
	}

	.zhuige-converge {
		padding-bottom: 20rpx;
	}

	.zhuige-topic-info>view:nth-child(2) view:nth-child(1) {
		font-size: 28rpx;
	}

	.zhuige-classify-join text {
		font-size: 24rpx;
		height: 48rpx;
		line-height: 48rpx;
		padding: 0 28rpx;
		border-radius: 48rpx;
		font-weight: 400;
		background: #EEEEEE;
	}
</style>