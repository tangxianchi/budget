### laravelInit 只是为了方便以后使用才建立的项目

### 使用
 	1、git clone https://github.com/wangle201210/laravel.git wangle1
 	2、composer install
 	3、php artisan key:
 	4、php artisan migrate

### 引入的包及其功能
## dingo
- API文档
- 频率限制

### 引入apidocjs
- 引入 npm install apidoc -g
- 生成API文档 apidoc -i app/Http/Controllers/API -o public/api

### 常用命令
 	创建控制器
 	php artisan make:controller XxxController --resource # 创建控制器
 	php artisan make:model Xxx  # 创建模型
 	php artisan api:generate --routePrefix="v1" --router="Dingo" # refresh docs
 	php artisan make:request XxxRequest
 	php artisan make:migration create_users_table --create=users
 	php artisan make:migration add_votes_to_users_table --table=users

### JWT
 	用户验证

### laratrust
 	权限控制

###Header
 	Content-Type : application/json
 	Authorization : Bearer token

### filter and sort
 	使用方法
 	class SomeModel extends Model{
 		use FilterAndSorting;


 	public function index(Request $request){
 		return SomeModel::setFilterAndRelationsAndSort($request)->get()


 	/message?filter={"created_at":{"from":"2016-02-20","to":"2016-02-24 23:59:59"}, "id":{"operation":"not in", "value":[2,3,4]}}
 	/message?filter={"id":{"from":2,"to":5}}
 	/message?filter={"id":{"to":5}} or /message?filter={"id":{"operation":"<=","value":5}}
 	/message?filter={"updated_at":{"isNull":true}}
 	/message?filter={"answer":{"operation":"like","value":"Partial search string"}}
 	/message?filter={"answer":"Full search string"}
 	/message?filter={"user.name":"asd"} # 关联搜索
 	/message?filter={"id":1}

 	# 暂时只支持单字段排序
 	/message?sort=id
 	/message?sort=-id
 	/message?sort=user.name

 	/message?expand=user # 关联搜索
 	response: { "id": 1, "message": "some message", "user_id": 1, ... "user": { "id": 1, "name": "Some username", ... } }

### redis 基本使用
 	Redis::set('key','value'); //存入redis
 	Redis::setex('key','time','value'); //设置键值和过期时间
 	Redis::setnx('key','value'); //如果没有键为key的那么,设置key的值为value
 	Redis::get('key'); //获取Redis中的值
 	Redis::incr('key'); //若原来key的值为1那么incr之后就会加1变为2
 	Redis::decr('key'); //若原来key的值为2那么incr之后就会减1变为1
 	Redis::del('key'); //删除键值
 	Redis::lLen('key'); //队列的长度
 	Redis::rpop('key'); //右侧出队列
 	Redis::rpush('key','value'); //右侧存入队列
 	Redis::exists($key) //redis是否存在这个键

### other

 	artisan migrate
 	artisan db:seed

 	artisan serve --host=127.0.0.1
 		ngrok --subdomain=xxxxx 8000

 	curl -X POST -F "name=xxx" -F "password=password" "http://localhost:8000/api/auth/login"

### 数据的恢复
 	$res = TABLE::withTrashed()->whereId($request->id)->restore();
 	if ($res) {
 		return $this->response->error('success', 201);
 	} else {
 		return $this->response->error('未知错误!', 200);
 	}
## status_code 状态码说明 出现参数 均表示表示业务处理出现错误。
 	200 会在message中显示错误,
 	202 出现错误 一般是参数不足 或会给出错误提示
 	203 出现错误 一般是授权不通过
 	210 出现错误 目标不存在
 	211 出现错误 业务重复
 	其它看情况再说明
## HTTP 状态码说明 与业务完成度有一定关系 并不绝对。
 	200 成功,
 	201 成功-完成新增或修改
 	202 接受 但是出现错误 一般是参数不足 或会给出错误提示
 	203 接受 但是出现错误 一般是授权不通过
 	204 成功-一般用于删除操作,表示完成
 	其它看情况再说明
### 审核流程
<!--  	state 数据状态(默认为0 一次上报为1 一次审批为2 二次上报为3 二次审批为4 人大审批为5)
 	xmzt 项目状态(未上报为1 已上报为2 业务股已审批为3 预算单位已审批为4 人大审批为5)
 	bzxl 编制序列(一次上报为1 二次上报为2 人大为3)
 	type=1	初始生成                 {state:0,xmzt:1,bzxl:1} (初始数据)
 	type=2	一上提交结果为            {state:1,xmzt:2,bzxl:1}* 并生成 {state:2,xmzt:2,bzxl:1}
 	type=3	一下业务股审批提交结果为   {state:2,xmzt:3,bzxl:1}
 	type=4	一下预算股室审批提交结果为 {state:2,xmzt:4,bzxl:1}* 并生成 {state:3,xmzt:1,bzxl:2}
 	type=5	二上提交结果为            {state:3,xmzt:2,bzxl:2}* 并生成 {state:4,xmzt:2,bzxl:2}
 	type=6	二下业务股审批提交结果为   {state:4,xmzt:3,bzxl:2}
 	type=7	二下预算股室审批提交结果为 {state:4,xmzt:4,bzxl:2}* 并生成 {state:5,xmzt:4,bzxl:3}
 	type=8	人大审批提交结果为        {state:5,xmzt:5,bzxl:3}* -->

### 以后只有state
	state=1	初始生成 					该单位上报 	
	state=2	一上提交结果 					该股室审核 
	state=3	一下业务股审批提交结果		该预算审核	
	state=4	一下预算股室审批提交结果		该单位再次上报
	state=5	二上提交结果 					该股室审核 
	state=6	二下业务股审批提交结果 		该预算审核
	state=7	二下预算股室审批提交结果 		结束||添加人大审批
	state=8	人大审批提交 					... ...

### 注意事项
	1.在基础设置项里面的`在岗情况`里面必须有`在职`这一项，否则会导致无法计算日常公用经费
	2.功能科目需要有2010101项目


## 开发备注

### 设置
	在基础设置项里面的`在岗情况`里面必须有`在职`这一项，否则会导致无法计算日常公用经费
	功能科目需要有2010101项目，用于基本支出找不到对应功能科目时使用
	单位下可建部门，部门所用计算公式继承上级单位
	项目归类必须含有001-无归类项，否则项目将打不开

### 安装
	如果上传实在传不上去，php又没有报错 那就应该是apache的问题  FcgidMaxRequestLen 104857600 #100M

### 系统常量
行政人员   		<!-- array 基本信息处区分人员性质 -->
事业人员			<!-- array 基本信息处区分人员性质 -->
在职人员 		<!-- number 基本信息处区分人员性质 -->
指标文号 		<!-- string 生成指标时文号的前缀 -->
指标管理文号 		<!-- string 生成指标时管理文号的前缀 -->
预算方案 		<!-- string 有些不同县特殊处理的地方，均用此处区分 -->
单位名 			<!-- string 登录时显示名字 -->
在职人员身份 		<!-- array 在职人员实际对应id -->