<template>
	<view class="content">
		<hacjy-uploadAvatar ref="uploadAvatar" :url="userImage" @uploadSuccess="uploadSuccess"
			@uploadFail="uploadFail"></hacjy-uploadAvatar>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				userImage: ''
			}
		},
		onLoad() {

		},
		methods: {
			uploadSuccess(res) { //上传成功
				console.log('uploadSuccess', res)
				if (res) {
					var obj = JSON.parse(res.data)
					if (obj && obj.code === 200) {
						var result = obj.data
						if (result) {
							this.userImage = result.file_url
						}
					} else {
						uni.$u.toast(res.data.message)
					}
				}
			},
			uploadFail(err) { //上传失败
				console.log('uploadFail', err)
			},
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
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>