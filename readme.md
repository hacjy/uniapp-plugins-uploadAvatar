## hacjy-uploadAvatar -- 一键上传头像组件
1. 支持设置头像大小size
2. 支持设置上传地址action
3. 支持拍照和选择相册两种方式
4. 更多字段如下：
```
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
			sourceType: { //选择照片来源
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
```
<br/>