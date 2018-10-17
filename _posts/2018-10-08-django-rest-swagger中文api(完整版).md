# django-rest-swagger：
## 一、django rest frame work
**概念：**
API(接口）：开发人员提供编程接口被其他人调用，他们调用之后会返回数据供其使用。
API的类型有多种，但是想在使用最多的是REST API


#### **RESTFUL API概念总结：**
**1.每一个URL代表一种资源**

**2.客户端和服务器之间，传递这种资源的某种表现层**

**3.客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。具体为：
GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。** 

**序列化是为了返回json格式的数据给客户端查看和使用数据，那么当客户端需要修改、增加或者删除数据时，就要把过程反过来了，也就是反序列化，把客户端提交的json格式的数据反序列化。**

函数视图：
@csrf_exempt注解来标识一个视图可以被跨域访问


类视图：
`url(r'^myview/$', csrf_exempt(views.MyView.as_view()), name='myview')`
## 二、djangorestframework中的请求和响应

* **Request对象**


```python
request.POST  # 只能处理表单数据.只能处理POST请求
request.data  # 能处理各种数据。  可以处理'POST', 'PUT' 和 'PATCH'模式的请求
```
* **请求响应状态码**

restframework 将原来数字类型的状态码优化为可读类型的状态码HTTP_400_BAD_REQUEST、HTTP_404_NOT_FOUND这种，极大的提高可读性。

* **装饰API视图**

	rest框架还提供了一个装饰器和一个类来包装视图函数，可以使用它们来写API视图

	###### 1、@api_view装饰器用在基于视图的方法上

	###### 2、APIView类用在基于视图的类上
	这两个功能能够在视图中收到Request对象或者在你的Response对象中添加上下文，此外装饰器在
	接受到输入错误的request.data时抛出ParseError或者在适当的时候返回405Method Not Allowed状态码。

	原本的视图函数snippet_detail中，处理'PUT'请求的时候，需要先解析json格式的数据再进一步处理：

	```python
	data = JSONParser().parse(request)
	serializer = SnippetSerializer(snippet, data=data)
	#替换为
	serializer = SnippetSerializer(data=request.data)
	
	return JsonResponse(serializer.data, status=201)
	#替换为
	return Response(serializer.data,status=status.HTTP_201_CREATED)
	```

	```python
	from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
@api_view(['GET', 'POST'])
def snippet_list(request):
    """
    列出所有已经存在的snippet或者创建一个新的snippet
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        
        
	@api_view(['GET', 'PUT', 'DELETE'])
	def snippet_detail(request, pk):
	    """
	    Retrieve, update or delete a snippet instance.
	    """
	    try:
	        snippet = Snippet.objects.get(pk=pk)
	    except Snippet.DoesNotExist:
	        return Response(status=status.HTTP_404_NOT_FOUND)
	
	    if request.method == 'GET':
	        serializer = SnippetSerializer(snippet)
	        return Response(serializer.data)
	
	    elif request.method == 'PUT':
	        serializer = SnippetSerializer(snippet, data=request.data)
	        if serializer.is_valid():
	            serializer.save()
	            return Response(serializer.data)
	        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
	
	    elif request.method == 'DELETE':
	        snippet.delete()
	        return Response(status=status.HTTP_204_NO_CONTENT)
        ```
        ```python
        
        ```
        

* **给请求url添加可选的格式后缀例如**


	`http://127.0.0.1:8000/snippets.json`

	根据请求url返回指定的json格式的Response，需要给视图函数添加关键字参数format

	```python
	#views.py文件
	def snippet_list(request, format=None):
	
	#urls.py文件导入format_suffix_patterns(格式后缀模式)
	from django.conf.urls import url
	from rest_framework.urlpatterns import format_suffix_patterns
	from snippets import views
	
	urlpatterns = [
	    url(r'^snippets/$', views.snippet_list),
	    url(r'^snippets/(?P<pk>[0-9]+)$', views.snippet_detail),
	]
	
	urlpatterns = format_suffix_patterns(urlpatterns)
	```
	`@api_view(['GET','POST'])`

	`@api_view(['GET','PUT','DELETE'])`


	@api_view和APIView装饰的视图函数在接受到请求动作范围之外就会报出405 Method Not Allowed
	
	
* **继承APIView的类视图**


	重构snnipets/views.p文件


	```python
	from snippets.models import Snippet
	from snippets.serializers import SnippetSerializer
	from django.http import Http404
	from rest_framework.views import APIView
	from rest_framework.response import Response
	from rest_framework import status
	
	
	class SnippetList(APIView):
	    """
	    列出所有已经存在的snippet或者创建一个新的snippet
	    """
	    def get(self, request, format=None):
	        snippets = Snippet.objects.all()
	        serializer = SnippetSerializer(snippets, many=True)
	        return Response(serializer.data)
	
	    def post(self, request, format=None):
	        serializer = SnippetSerializer(data=request.data)
	        if serializer.is_valid():
	            serializer.save()
	            return Response(serializer.data, status=status.HTTP_201_CREATED)
	        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
	```

**1.get_schema\_view()添加一个模式函数解析**


REST框架包含一个功能自动生成一个schema，或者允许你指定一个。有许多不同的方式来为你添加一个schema


```python
from rest_framework.schemas import get_schema_view
# 导入get_schema_view method
schema_url_patterns = [url(r'^api/', include('myproject.api.urls')),]
#patterns参数
schema_view = get_schema_view(title="Server Monitoring API"
urlpatterns = [
    url('^$', schema_view),
    ...
]
#get_schema_view(title="Server Monitoring API",
					url="https://www.example.org/api/",
					url_conf='myproject.urls',
					renderer_classes=[OpenAPIRenderer, SwaggerUIRenderer],
					patterns=schema_url_patterns
					generator_class=“”)
#title:描述模式标题
#url:可用于传递模式的规范URL
#urlconf:一个字符串表示您想要生成一个API模式的URL conf的导入路径。默认值为django的 ROOT_URLCONF设置
#renderer_classes：用来传递API根端点的渲染器类
#patterns：用于限制内联的url列表，如果你仅想暴露myproject.api的urls
#generator_class：指定一个SchemaGenerator子类，用于传递SchemaView
```



**2.指定一个SchemaView**

如果需要比get_schema_view()更多的控制权限，你需要使用SchemaGenerator类自动创建Document的实例，然后从视图中返回。 
这个选项给您提供了您想要的任何行为灵活设置模式端点。例如，您可以将不同的权限、节流或身份验证策略应用到模式端点。


```python
from rest_framework.decorators import api_view, renderer_classes
from rest_framework import renderers, response, schemas

generator = schemas.SchemaGenerator(title='Bookings API')

@api_view()
@renderer_classes([renderers.CoreJSONRenderer])
def schema_view(request):
    schema = generator.get_schema(request)
    return response.Response(schema)

# urls.py
urlpatterns = [
    url('/', schema_view),
    ...
]

```
*a、您还可以为不同的用户提供不同的模式，这取决于他们所拥有的权限。此方法可用于确保未经身份验证的请求与经过身份验证的请求不同，或确保不同的用户根据角色的不同判断用户是否可见。*
ppt自带了常用的1-48种模板通过index选择对应的模板


```python
@api_view()
@renderer_classes([renderers.CoreJSONRenderer])
def schema_view(request):
    generator = schemas.SchemaGenerator(title='Bookings API')
    return response.Response(generator.get_schema(request=request))

```



**3.显示模式定义**

```python
import coreapi
from rest_framework.decorators import api_view, renderer_classes
from rest_framework import renderers, response

schema = coreapi.Document(
    title='Bookings API',
    content={
        ...
    }
)

@api_view()
@renderer_classes([renderers.CoreJSONRenderer])
def schema_view(request):
    return response.Response(schema)
```
**4.静态模式文件**

将您的API模式编写为静态文件，使用可用的一种格式，比如Core JSON或Open API
##三、django-rest-swagger
**1.#  swagger 配置项**

*a、settings.py文件中配置swagger*
将rest_framework,rest_framework_swagger加入到app应用中
此外配置swagger_settings


```python
SWAGGER_SETTINGS = {
    # 基础样式
    'SECURITY_DEFINITIONS': {
        "basic":{
            'type': 'basic'
        }
    },
    # 如果需要登录才能够查看接口文档, 登录的链接使用restframework自带的.
    'LOGIN_URL': 'rest_framework:login',
    'LOGOUT_URL': 'rest_framework:logout',
    # 'DOC_EXPANSION': None,
    # 'SHOW_REQUEST_HEADERS':True,
    # 'USE_SESSION_AUTH': True,
    # 'DOC_EXPANSION': 'list',
    # 接口文档中方法列表以首字母升序排列
    'APIS_SORTER': 'alpha',
    # 如果支持json提交, 则接口文档中包含json输入框
    'JSON_EDITOR': True,
    # 方法列表字母排序
    'OPERATIONS_SORTER': 'alpha',
    'VALIDATOR_URL': None,
}

```



*b、django rest framework 序列化模型类*

```python
# 序列化
from django.contrib.auth.models import User,Group
from  rest_framework import serializers

class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = "__all__"
class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model =Group
        fields = "__all__"


```


*c、配置views.py


```python
from  django.contrib.auth.models import User,Group
from rest_framework import viewsets
from  api.serializers import UserSerializer,GroupSerializer

# Create your views here.

class UserViewSet(viewsets.ModelViewSet):
    '''查看，编辑用户的界面'''
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer

class GroupViewSet(viewsets.ModelViewSet):
    '''查看，编辑组的界面'''
    queryset = Group
    serializer_class = GroupSerializer
```

*d、配置urls.py*



```python
from django.conf.urls import url,include
from django.contrib import admin
from  rest_framework import routers
from  api import views

# 路由
router = routers.DefaultRouter()
router.register(r'users',views.UserViewSet,base_name='user')
router.register(r'groups',views.GroupViewSet,base_name='group')


# 重要的是如下三行
from rest_framework.schemas import get_schema_view
from rest_framework_swagger.renderers import SwaggerUIRenderer, OpenAPIRenderer
schema_view = get_schema_view(title='Users API', renderer_classes=[OpenAPIRenderer, SwaggerUIRenderer])



urlpatterns = [
    # swagger接口文档路由
    url(r'^docs/', schema_view, name="docs"),
    url(r'^admin/', admin.site.urls),
    url(r'^',include(router.urls)),
    # drf登录
    url(r'^api-auth/',include('rest_framework.urls',namespace='rest_framework'))

]

```
