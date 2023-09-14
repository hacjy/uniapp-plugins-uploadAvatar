<template>
	<view>
		<u-action-sheet :actions="optionList" :show="show" cancelText="取消" round="16" :closeOnClickOverlay="true"
			@close="close" @select="selectClick"></u-action-sheet>
		<view class="container flex text-flex-center" @click="uploadAvatar">
			<u-avatar class="avatar" defaultUrl="/static/hacjy-uploadAvatar/default_avatar.png" :src="imageUrl" :size="size"></u-avatar>
			<image v-if="showIcon" class="icon-photo" :style="iconStyle" :src="icon"></image>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			url: {
				type: String,
				default: ""
			},
			size: {
				type: Number,
				default: 78
			},
			showIcon: {
				type: Boolean,
				default: true
			},
			icon: {
				type: String,
				default: "/static/hacjy-uploadAvatar/icon_photo.png"
			},
			iconStyle: {
				type: String,
				default: ""
			},
			sourceType: { //选择照片来源 【ps：H5就别费劲了，设置了也没用。不是我说的，官方文档就这样！！！】
				type: Array,
				default: () => ['album', 'camera'],
			},
			action: { //上传地址
				type: String,
				default: '',
			},
			compress: { //是否需要压缩
				type: Boolean,
				default: true,
			},
			quality: { //压缩质量，范围0～100
				type: Number,
				default: 80,
			},
			name: { //发到后台的文件参数名
				type: String,
				default: 'file',
			},
			headers: { //上传的请求头部
				type: Object,
				default: () => {},
			},
			formData: { //HTTP 请求中其他额外的 form data
				type: Object,
				default: () => {},
			},
		},
		data() {
			return {
				imageUrl: "",
				show: false,
				optionList: [{
						name: '拍摄'
					},
					{
						name: '从手机相册选择'
					}
				],
			}
		},
		created() {
			this.imageUrl = this.url
		},
		methods: {
			uploadAvatar() {
				this.imgAdd()
			},
			selectClick(e) {
				console.log(e, 'selectClick')
				this.show = false
				if (e.name == '拍摄') {
					this.appCamera();
				} else {
					this.appGallery(1);
				}
			},
			close() {
				this.show = false
			},
			imgAdd() {
				console.log('uploadAvatar imgAdd')
				let thisNum = 1
				// #ifdef APP-PLUS
				if (this.sourceType.length > 1) {
					this.show = true
				}
				if (this.sourceType.length == 1 && this.sourceType.indexOf('album') > -1) {
					this.appGallery(thisNum);
				}

				if (this.sourceType.length == 1 && this.sourceType.indexOf('camera') > -1) {
					this.appCamera();
				}
				// #endif
				//#ifndef APP-PLUS
				uni.chooseImage({
					count: thisNum,
					sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
					sourceType: this.sourceType,
					success: (res) => {
						console.log('chooseImage', res)
						this.chooseSuccessMethod(res.tempFilePaths, 0)
						//console.log('tempFiles', res)
						// if (this.action == '') { //未配置上传路径
						// 	this.$emit("chooseSuccess", res.tempFilePaths);
						// } else {
						// 	if (this.compress && (res.tempFiles[0].size / 1024 > 1025)) { //设置了需要压缩 并且 文件大于1M，进行压缩上传
						// 		this.imgCompress(res.tempFilePaths);
						// 	} else {
						// 		this.imgUpload(res.tempFilePaths);
						// 	}
						// }
					}
				});
				// #endif
			},
			appCamera() {
				var cmr = plus.camera.getCamera();
				var res = cmr.supportedImageResolutions[0];
				var fmt = cmr.supportedImageFormats[0];
				//console.log("Resolution: " + res + ", Format: " + fmt);
				cmr.captureImage((path) => {
						//alert("Capture image success: " + path);
						this.chooseSuccessMethod([path], 0)
					},
					(error) => {
						//alert("Capture image failed: " + error.message);
						console.log("Capture image failed: " + error.message)
					}, {
						resolution: res,
						format: fmt
					}
				);
			},
			appGallery(maxNum) {
				plus.gallery.pick((res) => {
					this.chooseSuccessMethod(res.files, 0)
				}, function(e) {
					//console.log("取消选择图片");
				}, {
					filter: "image",
					multiple: true,
					maximum: maxNum
				});
			},
			chooseSuccessMethod(filePaths, type) {
				console.log("action=" + this.action)
				if (this.action == '') { //未配置上传路径
					this.$emit("chooseSuccess", filePaths, type); //filePaths 路径 type 0 为图片 1为视频
				} else {
					if (type == 1) {
						this.imgUpload(filePaths);
					} else {
						if (this.compress) { //设置了需要压缩
							this.imgCompress(filePaths);
						} else {
							this.imgUpload(filePaths);
						}
					}

				}
			},
			imgCompress(tempFilePaths) {
				uni.showLoading({
					title: '压缩中...'
				});
				let compressImgs = [];
				let results = [];
				tempFilePaths.forEach((item, index) => {
					compressImgs.push(new Promise((resolve, reject) => {
						// #ifndef H5
						uni.compressImage({
							src: item,
							quality: this.quality,
							success: res => {
								console.log('compressImage', res.tempFilePath)
								results.push(res.tempFilePath);
								resolve(res.tempFilePath);
							},
							fail: (err) => {
								//console.log(err.errMsg);
								reject(err);
							},
							complete: () => {
								//uni.hideLoading();
							}
						})
						// #endif
						// #ifdef H5
						this.canvasDataURL(item, {
							quality: this.quality / 100
						}, (base64Codes) => {
							//this.imgUpload(base64Codes);
							results.push(base64Codes);
							resolve(base64Codes);
						})
						// #endif
					}))
				})
				Promise.all(compressImgs) //执行所有需请求的接口
					.then((results) => {
						uni.hideLoading();
						console.log('imgUpload', results)
						this.imgUpload(results);
					})
					.catch((res, object) => {
						uni.hideLoading();
					});
			},
			imgUpload(tempFilePaths) {
				// if (this.action == '') {
				// 	uni.showToast({
				// 		title: '未配置上传地址',
				// 		icon: 'none',
				// 		duration: 2000
				// 	});
				// 	return false;
				// }
				uni.showLoading({
					title: '上传中'
				});
				console.log('uploadAvatar', tempFilePaths)
				let uploadImgs = [];
				tempFilePaths.forEach((item, index) => {
					uploadImgs.push(new Promise((resolve, reject) => {
						console.log(index, item)
						var headers = this.headers
						const uploadTask = uni.uploadFile({
							url: this.action,
							filePath: item,
							name: this.name,
							fileType: 'image',
							formData: this.formData,
							header: headers,
							success: (uploadFileRes) => {
								//uni.hideLoading();
								//console.log(typeof this.uploadSuccess)
								//console.log('')
								if (typeof this.uploadSuccess == 'function') {
									if (this.uploadSuccess(uploadFileRes).success) {
										this.value.push(this.uploadSuccess(uploadFileRes)
											.url)
										this.$emit("input", this.uploadLists);
									}
								}
								resolve(uploadFileRes);
								this.$emit("uploadSuccess", uploadFileRes);
							},
							fail: (err) => {
								console.log(err);
								//uni.hideLoading();
								reject(err);
								this.$emit("uploadFail", err);
							},
							complete: () => {
								//uni.hideLoading();
							}
						});
					}))
				})
				Promise.all(uploadImgs) //执行所有需请求的接口
					.then((results) => {
						uni.hideLoading();
					})
					.catch((res, object) => {
						uni.hideLoading();
						this.$emit("uploadFail", res);
					});
			},
			canvasDataURL(path, obj, callback) {
				var img = new Image();
				img.src = path;
				img.onload = function() {
					var that = this;
					// 默认按比例压缩
					var w = that.width,
						h = that.height,
						scale = w / h;
					w = obj.width || w;
					h = obj.height || (w / scale);
					var quality = 0.8; // 默认图片质量为0.8
					//生成canvas
					var canvas = document.createElement('canvas');
					var ctx = canvas.getContext('2d');
					// 创建属性节点
					var anw = document.createAttribute("width");
					anw.nodeValue = w;
					var anh = document.createAttribute("height");
					anh.nodeValue = h;
					canvas.setAttributeNode(anw);
					canvas.setAttributeNode(anh);
					ctx.drawImage(that, 0, 0, w, h);
					// 图像质量
					if (obj.quality && obj.quality <= 1 && obj.quality > 0) {
						quality = obj.quality;
					}
					// quality值越小，所绘制出的图像越模糊
					var base64 = canvas.toDataURL('image/jpeg', quality);
					// 回调函数返回base64的值
					callback(base64);
				}
			},
		}
	}
</script>

<style lang="scss" scoped>
	.container {
		position: relative;
	}

	.avatar {
	}

	.icon-photo {
		position: absolute;
		left: 54%;
		bottom: 4rpx;
		z-index: 100;

		width: 48rpx;
		height: 48rpx;
		border-radius: 50%;
		background: rgba(0, 0, 0, 0.5);
	}
</style>