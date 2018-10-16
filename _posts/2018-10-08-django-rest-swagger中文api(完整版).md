# django-rest-swagger：
## 一、django rest frame work


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
##二、django-rest-swagger
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
