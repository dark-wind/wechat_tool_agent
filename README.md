# 微信开发者工具agent超长bug复现

# a 使用本项目复现bug
## 1 创建数据库
创建数据库，并导入server目录下wechat_tool_test.sql文件
使用导入而不是初始化，是因为给普通用户追加了2个api权限
配置config.yaml
```
mysql:
  path: 127.0.0.1
  port: "3306"
  config: charset=utf8mb4&parseTime=True&loc=Local
  db-name: wechat_tool_test
  username: yourusername
  password: "yourpwd"
```
## 2 安装运行前后端代码
```
运行后端
cd server
直接运行编译包
./server.exe
(linux) ./server
或者执行main.go文件

运行前端
cd ../web
cnpm install
cnpm run serve
```

## 3 访问页码登录获取x-token值
http://localhost:8080

# 第四步，只需选择一种方式

## 4 直接导入微信小程序
导入微信小程序

目录选择
gin-vue-admin\wechat_tool_agent\unpackage\dist\dev\mp-weixin

使用测试号即可，无需appid

修改token值，模拟登录：
pages\index\index.js 156 行

      token: '新获取到的token'

## 4 （使用hbuilder）编译微信小程序
导入gin-vue-admin\wechat_tool_agent 文件夹

目录选择
gin-vue-admin\wechat_tool_agent\pages\index\index.vue 16行

      token: '新获取到的token'




## 5 编译运行小程序
无需操作，运行后，自动访问后端agent接口
## 4 查看后端日志可看到错误
```
2023/03/29 20:54:59 D:/项目代码/www/temp/gin-vue-admin/server/service/system/sys_operation_record.go:19 Error 1406: Data too long for column 'agent' at row 1
[86.453ms] [rows:0] INSERT INTO `sys_operation_records` (`created_at`,`updated_at`,`deleted_at`,`ip`,`method`,`path`,`status`,`latency`,`agent`,`error_message`,`body`,`resp`,`user_id`) VALUES ('2023-03-29 20:54:59.2','2023-03-29 20:54:59.2',NULL,'127.0.0.1','GET','/user/agent',200,0,'Mozilla/5.0 (iPhone; CPU iPhone OS 10_3_1 like Mac OS X) AppleWebKit/603.1.3 (KHTML, like Gecko) Version/10.0 Mobile/14E304 Safari/602.1 wechatdevtools/1.06.2303060 MicroMessenger/8.0.5 Language/zh_CN webview/','','{}','{"code":0,"data":{},"msg":"GetAgentTest"}',1)
[github.com/flipped-aurora/gin-vue-admin/server]2023/03/29 - 20:54:59.244	error	D:/项目代码/www/temp/gin-vue-admin/server/middleware/operation.go:122	create operation record error:	{"error": "Error 1406: Data too long for column 'agent' at row 1"}
[GIN] 2023/03/29 - 20:54:59 | 200 |    110.3421ms |       127.0.0.1 | GET      "/user/agent"
```

# ！！！复现成功则无需看下面内容-------------------------------------------

# b 项目构建流程

## 1 clone源码
```
git clone https://github.com/flipped-aurora/gin-vue-admin.git
```
使用最新的main分支进行复现（2023/3/29）

## 编写测试路由及代码
server\api\v1\system\sys_user.go 464行插入测试用接口
```
func (b *BaseApi) GetAgentTest(c *gin.Context) {

	response.OkWithMessage("GetAgentTest", c)
}

func (b *BaseApi) PostAgentTest(c *gin.Context) {

	response.OkWithMessage("PostAgentTest", c)
}
```
server\router\system\sys_user.go 25行插入路由，这2个测试路由使用操作日志中间件
```
userRouter.GET("agent", baseApi.GetAgentTest)           // get方法agent测试
userRouter.POST("agent", baseApi.PostAgentTest)           // post方法agent测试
```
## 2 前后端依赖安装
```
后端直接编译运行main.go即可
前端：
cd web 
cnpm install
cnpm run 
cd serve serve
```
## 3 创建数据库
数据库版本；
mysql5.7（使用的docker官方镜像对应版本）

创建mysql数据库：
wechat_tool_test

字符集：
utf8mb4
排序规则：
utf8mb4_general_ci

## 4 访问http://localhost:8080

## 5 初始化项目填入数据库信息

## 6 编写微信小程序
使用了uni-api，仅仅引入最简单的helloworld模板
在wechat_tool_agent\pages\index.vue中写入
```
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
```

token从页面访问，开发者工具获取x-token
## 7 编译运行到微信小程序

