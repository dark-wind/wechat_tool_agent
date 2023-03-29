<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				baseUrl:'http://localhost:8080/',
				token:'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVVUlEIjoiZDdkYmVjYmQtY2E3NC00NjkwLTliNGUtODU4NzE1MjljNmI1IiwiSUQiOjEsIlVzZXJuYW1lIjoiYWRtaW4iLCJOaWNrTmFtZSI6Ik1yLuWlh-a3vCIsIkF1dGhvcml0eUlkIjo4ODgsIkJ1ZmZlclRpbWUiOjg2NDAwLCJleHAiOjE2ODA2OTgwMzcsImlzcyI6InFtUGx1cyIsIm5iZiI6MTY4MDA5MjIzN30.rMOjnv73H9d0PmzwMR_XYpzokUga6ztes5v0NV5NdZs'
			}
		},
		onLoad() {
		},
		onShow(){
			// console.log($tool.baseUrl)
			this.testAgent()
		},
		methods: {
			testAgent() {
				const token = uni.getStorageSync('token');
				const userInfo = uni.getStorageSync('setUserData');
				uni.request({
					url: this.baseUrl+'api/user/agent',
					method: "GET",
					data: {},
					header: {
						'x-token': this.token //自定义请求头信息
					},
					success: (res) => {
						console.log(res)
					}
				});

			},
		}
	}
</script>

<style>

</style>
