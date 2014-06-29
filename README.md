#Django Render#
=============

##0. 开始

一个非常轻量Django URL 的装饰器

	from django_render.annotations import *
	
	@url(r'^/index$', method=M.POST)
	@post(text=str)
	def index(request, text):
		return {'hello': text}

##1. 简介

基于Python 2.7 上重新定义 Django url 的用法，使用@语法可以轻松定义出 RESTful 风格的 url，也可以用于声明 HTTP GET/POST 参数。也可以用于用户认证拦截。自动识别返回类型 HttpResponse, JSON, google-protobuf

##2. 安装
使用 pip:

	sudo pip install git+http://xxxx
	
使用 git submodule:

	git submoudle add https://github.com/wangtai/django_render.git

##3. 用法

###3.1. 定义url
在urls.py中的定义:

	from django.conf.urls import patterns, include, url

	urlpatterns = patterns('',
	    url(r'^', include('mysite.views')),
	)
	
在views.py中声明: 

	@url(ur'^/hello$')
	def link(request):
		...
		return True
		
定义 GET|POST /index:

	@url(r'^/index$')
	
定义 GET /index:

	@url(r'^/index$', method=M.GET)
	
定义 POST|PUT /name:

	@url(r'^/name$', method=[M.POST, M.PUT])
	
需要分别处理的 POST /name , PUT /name:

	@url(r'^/name$', method=M.POST)
	def post_name(request):
		...
		
	@url(r'^/name$', method=M.PUT)
	def put_name(request):
		...
		
RESTful Style URL:

	@url(r'^/blog/$', method=M.GET)
	def get_blog_list(request):
		...
		
	@url(r'^/blog/(?P<id>\d+)$', method=M.GET)
	def get_specific_blog(request, id):
		...
		
	@url(r'^/blog/$', method=M.POST)
	@post(text=str)
	def post_a_blog(request, text):
		...
		
	@url(r'^/blog/(?P<id>\d+)$', method=M.PUT)
	@post(text=str)
	def change_a_blog_text(request, id, text):
		...
		
###3.2. 声明HTTP参数

GET/POST 参数:

	@get(id=int)
	@post(name=str)
	
在方法上接收:
	
	def hello(request, id, name):
		...

更灵活的使用方法, 以GET为例:

    @get('param1', 'param2') 
	''' 
	HTTP参数: param1, param2
	方法实参: param1, param2
	类型: str
	默认值: 无
	'''
	
    @get(param1=int)
    ''' 
	HTTP参数: param1
	方法实参: param1
	类型: int
	默认值: 无
	'''
	
    @get(param1=(int, 0))    
    ''' 
	HTTP参数: param1
	方法实参: param1
	类型: int
	默认值: 0
	'''
	
    @get(param1=('param_name', int, 0))
    ''' 
	HTTP参数: param_name
	方法实参: param1
	类型: int
	默认值: 0
	'''
	
    
语义化的用法:

    @get(param1={'name':'parameter_name', 'type':int, 'default':0})
    @get(param1={'type':int, 'default':0})
    @get(param1={'type':int })
    
参数类型除了可以转换成:

	str, int, bool
	
还支持array:

	Type.int_list, Type.str_list, Type.json
	
方法如下:

	@get(ids=Type.int_list, names=Type.str_list, extrs=Type.json)
	
其中 Type.int_list 和 Type.str_list, Value应构造成 ids=1,2,3 和 name=Bob,Johns,Peter

###3.3. 返回类型

如果 return HttpResponse 或其子类，则直接返回到客户端

如果希望返回Json 数据类型

	...
	return True
	# {'rt':true, 'message':''}
	
	...
	return False
	# {'rt':false, 'message':''}
	
	...
	return True, {'data': ...}
	# {'rt':true, 'data': ...}
	
	...
	return True, 'message content'
	# {'rt':true, 'message':'message content'}
	
	...
	return {'data':'xxx', 'num':1, 'other':[{...},...]}
	# {'data':'xxx', 'num':1, 'other':[{...},...]}

	...
	return []
	# []
	
	...
	return # direct return
	# {}
	
	...
	return 'message content'
	# {'message':'message content'}
	
###3.4. 用户认证

	# /hello?access_secret_key=The_Key_Only_You_Know
	@login_required(access_secret_key='The_Key_Only_You_Know', login_page='/login.html')
	
全局定义参数:

	## in settings.py
	from django_render
	...
	django_render.global_access_secret_key = 'The_Key_Only_You_Know'
	django_render.global_login_page = '/login.html'
	
	## in views.py
	@login_required
	
复杂的登录拦截器:

	def read_user_interceptor(request):
		...
		if success:
			return user
		else:
			return None
			
	...
	
	@url(r'^/hello$')
	@login_required(read_user_interceptor=read_user_interceptor)
	def hello(request, user):
		'''
		@param request: MUST BE EXIST!
		@param user: MUST BE EXIST!
		'''
		return True, {'user': user.name}
	
全局定义 read_user_interceptor:

	#in settings.py
	from django_render
	
	def read_user_interceptor(request):
		...
		if success:
			return user
		else:
			return None
			
	django_render.global_read_user_interceptor = read_user_interceptor
	
	## in views.py
	@login_required
	
如果是Ajax类型的请求
	
	@login_required(is_ajax=True)
	#如果登录失败，不会跳转到 login page, 而是返回 {'rt':false, 'message':'login first'}
	

##4. 注意

> @url() 并不是真正 django 的 url 方法，但因为是在基础上的封装，所以 django url 的其他用法 @url() 也同样支持

##5. 交流

* mail list: xxxx

##6. 联系

* email: i@wangtai.me
* twitter: wang_tai

##7. 待续

1. 自动识别google-protobuf
2. 支持 Form, Ajax 使用 PUT/DELETE/...
3. Type.int_list, Type.str_list 识别 id=1&id=2&id=3