<template>
	<view view class="content">

		<view class="zhuige-identity-box">
			<view class="zhuige-identity-block">
				<!-- 头部大图 后台控制 -->
				<view v-if="background" class="zhuige-identity-img">
					<image mode="aspectFill" :src="background"></image>
				</view>

				<!-- 认证信息 -->
				<view v-if="certify" class="zhuige-block zhuige-identity-info">
					<view class="zhuige-block-head">
						<view>认证信息</view>
					</view>
					<view class="zhuige-identity-info-line">
						<view>
							<text>认证状态：</text>
							<text>{{certify.status_desc}}</text>
							<template v-if="certify.status==2">
								<image :src="certify.certify_icon"></image>
								<text>{{certify.certify_name}}</text>
							</template>
							<text v-else>{{certify.remark}}</text>
						</view>
						<view v-if="certify.status==2">
							<text>有效时间：</text>
							<text>{{certify.certify_start}} - {{certify.certify_end}}</text>
						</view>
					</view>
				</view>

				<view v-if="icons && icons.length>0">
					<zhuige-icons type="scroll" :items="icons"></zhuige-icons>
				</view>
			</view>

			<view class="zhuige-block">

				<!-- 认证表单 -->
				<view class="zhuige-info-form">
					<view class="zhuige-info-line">
						<view>怎么称呼</view>
						<view>
							<input type="text" placeholder="请输入称呼" v-model="certify.name" />
						</view>
					</view>
					<view class="zhuige-info-line">
						<view>怎么联系</view>
						<view>
							<input type="text" placeholder="怎么才能联系您" v-model="certify.contact" />
						</view>
					</view>
					<view class="zhuige-info-line">
						<view>认证类型</view>
						<picker @change="bindPickerChange" :value="cert_index" :range="cert_names">
							<template v-if="cert_index > -1">
								<view>{{cert_names[cert_index]}}</view>
							</template>
							<template v-else>
								<view class="tips">选择您的认证类型</view>
							</template>
						</picker>
					</view>
					<view class="zhuige-info-line">
						<view>描述信息</view>
						<view>
							<input type="text" placeholder="一句话介绍一下自己吧" v-model="certify.brief" />
						</view>
					</view>
					<view class="zhuige-info-line">
						<view>图片信息</view>
						<view @click="clickCertificate()">
							<template v-if="certify.certificate">
								<view class="zhugie-info-image-upload">
									<image mode="aspectFill" :src="certify.certificate"></image>
									<view @click.stop="clickDelete">
										<uni-icons type="clear" color="#010101" size="22"></uni-icons>
									</view>
								</view>
							</template>
							<template v-else>
								<view class="zhugie-info-image-upload">
									<uni-icons type="clear" size="24"></uni-icons>
									<uni-icons type="plusempty" color="#999999" size="40"></uni-icons>
								</view>
							</template>
						</view>
					</view>
				</view>

			</view>
		</view>

		<!-- 底部大按钮 -->
		<view @click="clickSubmit()" class="zhuige-base-button">
			<view>提交</view>
		</view>

	</view>
</template>

<script>
	import Util from '@/utils/util';
	import Alert from '@/utils/alert';
	import Api from '@/utils/api';
	import Rest from '@/utils/rest';

	import ZhuigeIcons from "@/components/zhuige-icons";

	export default {
		data() {
			return {
				background: '',

				cert_icons: [],
				cert_names: [],
				cert_index: -1,

				icons: [],
				certify: undefined,
			};
		},

		components: {
			ZhuigeIcons
		},

		onLoad(options) {
			Util.addShareScore(options.source);

			this.loadSetting();
		},

		onShow() {
			// #ifdef MP-BAIDU
			swan.setPageInfo({
				title: '认证_' + getApp().globalData.appName,
				keywords: getApp().globalData.appKeywords,
				description: getApp().globalData.appDesc,
			});
			// #endif
		},

		onShareAppMessage() {
			return {
				title: '认证-' + getApp().globalData.appName,
				path: Util.addShareSource('pages/pllugins/certify/certify?n=n')
			};
		},

		// #ifdef MP-WEIXIN
		onShareTimeline() {
			return {
				title: '认证-' + getApp().globalData.appName,
			};
		},
		// #endif

		methods: {
			/**
			 * 选择认证种类
			 */
			bindPickerChange(e) {
				this.cert_index = e.detail.value
			},

			/**
			 * 上传图片
			 */
			clickCertificate() {
				uni.chooseImage({
					count: 1,
					sizeType: ['compressed'],
					success: (res) => {
						let path = res.tempFilePaths[0];
						Rest.upload(Api.URL('other', 'upload'), path).then(oo => {
							this.certify.certificate = oo.data.src;
						}, err => {
							Alert.error(err);
						});
					}
				});
			},

			/**
			 * 删除图片
			 */
			clickDelete() {
				this.certify.certificate = '';
			},

			/**
			 * 提交认证
			 */
			clickSubmit() {
				Rest.post(Api.URL('certify', 'apply_certify'), {
					name: this.certify.name,
					contact: this.certify.contact,
					certify_icon: this.cert_icons[this.cert_index].cid,
					certify_name: this.cert_icons[this.cert_index].name,
					brief: this.certify.brief,
					certificate: this.certify.certificate,
				}).then(res => {
					if (res.code == 0) {
						Alert.success(res.message);
						setTimeout(() => {
							Util.navigateBack();
						}, 1500)
					} else {
						Alert.error(res.message);
					}
				}, err => {
					console.log(err)
				});
			},

			/**
			 * 加载配置
			 */
			loadSetting() {
				Rest.post(Api.URL('certify', 'setting'), {}).then(res => {
					this.background = res.data.background;
					this.cert_icons = res.data.cert_icons;
					for (let i = 0; i < this.cert_icons.length; i++) {
						this.cert_names.push(this.cert_icons[i].name)
					}
					this.icons = res.data.icons;

					this.loadInfo();
				}, err => {
					console.log(err)
				});
			},

			/**
			 * 加载我的认证信息
			 */
			loadInfo() {
				Rest.post(Api.URL('certify', 'my_certify'), {}).then(res => {
					this.certify = res.data;
					for (let i = 0; i < this.cert_icons.length; i++) {
						if (this.cert_icons[i].cid == this.certify.certify_id) {
							this.cert_index = i;
						}
					}
				}, err => {
					console.log(err)
				});
			}
		}
	}
</script>

<style>
	.content {
		padding-bottom: 160rpx;
	}

	.zhuige-identity-box {
		padding: 0 20rpx;
	}

	.zhuige-identity-block {
		background: #FFFFFF;
		border-radius: 12rpx;
		margin-bottom: 20rpx;
	}

	.zhuige-identity-img image {
		width: 100%;
		height: 480rpx;
		border-radius: 12rpx 12rpx 0 0;
	}

	.zhuige-wrap-icon {
		padding: 20rpx 20rpx;
	}

	/* 控制图标数量对应宽度，建议宽度和数量
		6个 16.666666%
		5个 20%
		4个 25%
	 */
	.zhuige-icon view {
		width: 16.666666%;
	}

	.zhuige-identity-info-line {
		padding-bottom: 40rpx;
		border-bottom: 1rpx solid #EEEEEE;
	}

	.zhuige-identity-info-line view {
		display: flex;
		align-items: center;
	}

	.zhuige-identity-info-line view text {
		font-size: 26rpx;
		font-weight: 300;
		line-height: 1.8em;
		margin-right: 8rpx;
		color: #999999;
	}

	.zhuige-identity-info-line image {
		height: 28rpx;
		width: 28rpx;
		margin: 0 8rpx 0 4rpx;
	}

	.zhuige-identity-block .zhuige-scroll-icon {
		padding-bottom: 20rpx !important;
	}

	.zhuige-identity-block .zhuige-icon .text {
		color: #666666 !important;
	}

	.zhuige-info-form {
		padding: 0 20rpx;
	}

	.zhuige-info-line {
		padding: 30rpx 0;
		border-top: 1rpx solid #EEEEEE;
	}

	.zhuige-info-line:nth-child(1) {
		border: none;
	}

	.zhuige-info-line>view {
		font-size: 28rpx;
		font-weight: 300;
	}

	.zhuige-info-line>view:nth-child(1) {
		font-size: 30rpx;
		font-weight: 600;
		margin-bottom: 20rpx;
	}

	.zhuige-info-line>view:nth-child(2) input {
		width: 100%;
		font-size: 28rpx;
		font-weight: 300;
	}

	.zhuige-info-line .tips {
		color: #aaaaaa;
	}

	.zhugie-info-image-upload {
		height: 240rpx;
		width: 240rpx;
		text-align: center;
		border-radius: 12rpx;
		border: 1rpx solid #DDDDDD;
		position: relative;
	}

	.zhugie-info-image-upload uni-icons:nth-child(1) {
		position: absolute;
		z-index: 3;
		right: -20rpx;
		top: -50rpx;
	}

	.zhugie-info-image-upload uni-icons:nth-child(2) {
		line-height: 240rpx;
	}

	.zhugie-info-image-upload image {
		height: 100%;
		width: 100%;
		border-radius: 12rpx;
	}
</style>