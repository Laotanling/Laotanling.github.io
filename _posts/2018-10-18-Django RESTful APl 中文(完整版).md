# djangorestframework：
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

	###### 	restframework 将原来数字类型的状态码优化为可读类型的状态码HTTP_400_BAD_REQUEST、HTTP_404_NOT_FOUND这种，极大的提高可读性。

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
	
	#view.py文件
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
		        return Response(serializer.errors,status=status.HTTP_400_BAD_REQUEST)
		
		    elif request.method == 'DELETE':
		        snippet.delete()
		        return Response(status=status.HTTP_204_NO_CONTENT)
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


	###### 	重构snnipets/views.p文件基于类的视图把各种不同的HTTP请求分离开变成单个的方法，而不是if...elif...这样的结构


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
	        
	        
	class SnippetDetail(APIView):
	    """
	    检索查看、更新或者删除一个snippet
	    """
	    def get_object(self, pk):
	        try:
	            return Snippet.objects.get(pk=pk)
	        except Snippet.DoesNotExist:
	            raise Http404
	
	    def get(self, request, pk, format=None):
	        snippet = self.get_object(pk)
	        serializer = SnippetSerializer(snippet)
	        return Response(serializer.data)
	
	    def put(self, request, pk, format=None):
	        snippet = self.get_object(pk)
	        serializer = SnippetSerializer(snippet, data=request.data)
	        if serializer.is_valid():
	            serializer.save()
	            return Response(serializer.data)
	        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
	
	    def delete(self, request, pk, format=None):
	        snippet = self.get_object(pk)
	        snippet.delete()
	        return Response(status=status.HTTP_204_NO_CONTENT)
	```
###### 	更改urls.py文件


	```python
	from django.conf.urls import url
	from rest_framework.urlpatterns import format_suffix_patterns
	from snippets import views
	
	urlpatterns = [
	    url(r'^snippets/$', views.SnippetList.as_view()),
	    url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),
	]
	
	urlpatterns = format_suffix_patterns(urlpatterns
	```
* 	**基于继承mixins类的视图**


	使用类视图可以把各种HTTP请求分开，此外还能够提供封装性较好方法，mixxins类，它封装了很多操作，简化代码，使用也很简单，优化snippets/views.py视图函数：


	```python
	from snippets.models import Snippet
	from snippets.serializers import SnippetSerializer
	from rest_framework import mixins
	from rest_framework import generics
	
	class SnippetList(mixins.ListModelMixin,
	                  mixins.CreateModelMixin,
	                  generics.GenericAPIView):
	    queryset = Snippet.objects.all()
	    serializer_class = SnippetSerializer
	
	    def get(self, request, *args, **kwargs):
	        return self.list(request, *args, **kwargs)
	
	    def post(self, request, *args, **kwargs):
	        return self.create(request, *args, **kwargs)
	```
###### 	继承了 generic.GenericAPIView、mixins.ListModelMixin和mixins.CreatteModelMixin,mixins类为我们提供了list()和create()方法,使用这两个函数需要先设置queryset和serializer_class，同理另一个视图：
```python
class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
	    queryset = Snippet.objects.all()
	    serializer_class = SnippetSerializer
	
	    def get(self, request, *args, **kwargs):
	        return self.retrieve(request, *args, **kwargs)
	
	    def put(self, request, *args, **kwargs):
	        return self.update(request, *args, **kwargs)
	
	    def delete(self, request, *args, **kwargs):
	        return self.destroy(request, *args, **kwargs)
```
* **基于通用的视图类**


	进一步优化代码连mixins类都不用了，只使用generics就可以了，代码：


	```python
	from snippets.models import Snippet
	from snippets.serializers import SnippetSerializer
	from rest_framework import generics
	
	
	class SnippetList(generics.ListCreateAPIView):
	    queryset = Snippet.objects.all()
	    serializer_class = SnippetSerializer
	
	
	class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
	    queryset = Snippet.objects.all()
	    serializer_class = SnippetSerializer


## 三、RESTful API添加认证和权限
1. 	snippet与其创建者相互关联
1. 	只有经过身份验证（登录）的用户才可以创建snippets
1. 	只有创建该snippet的用户才可以对其进行更改或者删除
1. 	未经验证的用户只具有访问（只读）的功能

* 修改snippet模型
	snippets都和它们的创建用户关联起来，给Snippet模型添加一个owner字段


	```python
	owner = models.ForeignKey('auth.User', related_name='snippets', on_delete=models.CASCADE)
	highlighted = models.TextField()
	```
###### 	想要实现代码高亮，当然不是上面一行代码就搞定了，它现在还只是一个普通的字段而已。我们要做的是在保存的时候，也就是当执行save()时, 我们使用pygments生成高亮后的HTML，还是在model.py,首先导入相关的库:
	```python
	from pygments.lexers import get_lexer_by_name
	from pygments.formatters.html import HtmlFormatter
	from pygments import highlight
	```
	在Snippet类中添加save()方法:


	```python
	def save(self, *args, **kwargs):
	    """
	    使用pygments库来生成能使代码高亮的HTML代码
	    """
	    lexer = get_lexer_by_name(self.language)
	    linenos = self.linenos and 'table' or False
	    options = self.title and {'title': self.title} or {}
	    formatter = HtmlFormatter(style=self.style, linenos=linenos,
	                              full=True, **options)
	    self.highlighted = highlight(self.code, lexer, formatter)
	    super(Snippet, self).save(*args, **kwargs)
	```

* 	**给用户模型添加端点**


	```python
	from django.contrib.auth.models import User

	class UserSerializer(serializers.ModelSerializer):
	    snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())
	
	    class Meta:
	        model = User
	        fields = ('id', 'username', 'snippets')
	```
	`snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())`
###### 	snippets在User模型中是一个反向关系,使用ModelSerializer类时默认情况是不会包括这个关系,通过Snippet的owner能查询到User，而User这边查询不到一个用户创建的snippet，所以我们需要手动为用户序列添加这个字段。
User的序列化器添加之后，为了显示获取USER数据添加相关视图类，在views.py中添加：


	```python
	from django.contrib.auth.models import User
	from snippets.serializers import UserSerializer
	
	
	class UserList(generics.ListAPIView):
	    queryset = User.objects.all()
	    serializer_class = UserSerializer
	
	
	class UserDetail(generics.RetrieveAPIView):
	    queryset = User.objects.all()
	    serializer_class = UserSerializer
	```
	在snippets/urls.py文件中添加url:


	```python
	url(r'^users/$', views.UserList.as_view()),
	url(r'^users/(?P<pk>[0-9]+)/$', views.UserDetail.as_view()),
	```
* 	**把Snippets和Users关联起来**


	###### 	之前在Snippet模型中添加了一个owner外键字段，在views.py的SnippetList类视图中添加一个方法:


	```python
	class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly,)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
	```
	perform_create() 方法会在用户通过POST请求创建一个新的Snippet时，在保存新的Snippet数据的时候会把request中的user赋值给Snippet的owner.
* 	**serializer序列化过程获取owner字段数据**


	owner会在创建新的Snippet的时候拥有User的各个属性，那么在API中要让owner显示id还是用户名，为了提高可读性，答案当然是显示用户名了，所以我们在SnippetSerializer 下面增加一个字段：


	```python
	owner = serializers.ReadOnlyField(source='owner.username')
	```
	source参数就指定了哪个属性用于填充字段,为了在使用的时候显示owner，但是还要把它添加进Meta类里面，所以整个SnippetSerializer如下:


	```python
	class SnippetSerializer(serializers.ModelSerializer):
    # 这里可以使用也 CharField(read_only=True) 来替换
	    owner = serializers.ReadOnlyField(source='owner.username')
	
	    class Meta:
	        model = Snippet
	        fields = ('id', 'title', 'code', 'linenos', 'language', 'style','owner')
	```
* 	**添加权限**

	Snippet和User已经关联起来并且是可浏览的，下面实现权限问题：


	###### 	a、只有经过身份验证（登录）的用户才可以创建snippet
	
	###### 	b、只有创建该snippet的用户才可以对其进行更改或者删除
	
	###### 	c、未经验证的用户只具有访问（只读）的功能
	**首先，在views.py导入一个库：**


	`from rest_framework import permissions`


	**其次，为SnippetList 和 SnippetDetail添加权限判断，在这两个视图类中都加入：**

	
	```python
	permission_classes = (permissions.IsAuthenticatedOrReadOnly,)
	#特别注意:逗号一定要加上去，不然就会报错
	```
	**这行代码的作用就是判断当前用户是否为该Snippet的创建者，而其他用户只有只读属性，就是只能查看。**
	
* 	**为可浏览的API添加登录功能**


	###### 	刚才添加了权限判断，如果没有登录用户，那就相当于游客啦，什么功能都没有只能看，所以在浏览器浏览API的时候就需要登录 功能。在这里，强	大的django-rest-framework又为我们做了很多事情，想要在添加登录按钮和页面，只需要修改一个rest_tutorial/urls.py，添加一个URL匹配：

	```python
	urlpatterns += [
    url(r'^api-auth/', include('rest_framework.urls',
                               namespace='rest_framework')),
]
	```
	这里的r'^api-auth/'你可以设置成任意你喜欢的，但是命名空间一定要相同，就是namespace='rest_framework'。

	好了，现在打开浏览器，就可以看到在我们的API页面的右上角有一个登录的按钮，点击之后就可以使用之前创建的用户登录了。

	这个时候访问单个用户的详情，就可以看到该用户创建的所有Snippet的id值（需要先创建好几个Snippet，可以按照本系列第一篇文章中在shell模式中的方法来创建）。比如访问：
	`http://127.0.0.1:8000/users/2/`
* 	**添加对象权限**

	**接着我们要实现的是让所有的Snippet可以被所有人访问到，但是每个Snippet只有其创建者才可以对其进行更改、删除等操作**

	因此，我们需要设置一下自定义权限，使每个Snippet只允许其创建者编辑它。在snippets目录下新建一个permissions.py


	```python
	from rest_framework import permissions

	class IsOwnerOrReadOnly(permissions.BasePermission):
	    """
	    使每个Snippet只允许其创建者编辑它
	    """
	
	        def has_object_permission(self, request, view, obj):
	        # 任何用户或者游客都可以访问任何Snippet，所以当请求动作在安全范围内，
	        # 也就是GET，HEAD，OPTIONS请求时，都会被允许
	        if request.method in permissions.SAFE_METHODS:
	            return True
	
	        # 而当请求不是上面的安全模式的话，那就需要判断一下当前的用户
	        # 如果Snippet所有者和当前的用户一致，那就允许，否则返回错误信息
	        return obj.owner == request.user
	```
	代码的逻辑已在注释中，简单说就是提供判断功能，然后我们要把它运用起来，在view.py中的SnippetDetail 修改一下：


	```python
	class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
	    queryset = Snippet.objects.all()
	    serializer_class = SnippetSerializer
	    permission_classes = (permissions.IsAuthenticatedOrReadOnly,
	                          IsOwnerOrReadOnly,)
	#添加IsOwnerOrReadOnly
	```
	注意要导入IsOwnerOrReadOnly类:
	`from snippets.permissions import IsOwnerOrReadOnly`
	现在用浏览器打开单个Snippet详情页，如果你当前登录的用户是这个Snippet的创建者，那你会发现多了DELETE和PUT两个操作，比如访问http://127.0.0.1:8000/snippets/2/，效果如下：
	![详情](http://ww1.sinaimg.cn/large/8d7120c5gy1fj27nqepi5j20fi0h3gme.jpg)
	
	


* 	**用API授权**


	###### 	由于现在我们还没使用authentication 类，所以项目目前还是使用默认的SessionAuthentication 和 BasicAuthentication.
	###### 	在使用浏览器访问API的时候，浏览器会帮我们保存会话信息，所以当权限满足时就可以对一个Snippet进行删除或者更改，或者是创建一个新的Snippet。
	
	###### 	当如果是通过命令行来操作API，我们就必须在每次发送请求的时候添加授权信息，也就是用户名和密码，没有的话就会报错，比如：
	```python
	http POST http://127.0.0.1:8000/snippets/ code="print 123"

	{
	    "detail": "Authentication credentials were not provided."
	}
	```
	**正确的操作：**

	```python
	http -a username1:password POST http://127.0.0.1:8000/snippets/ code="print 789"

	{
	    "id": 1,
	    "owner": "username1",
	    "title": "",
	    "code": "print 789",
	    "linenos": false,
	    "language": "python",
	    "style": "friendly"
	}
	```
	我们可以看出owner就是提交过来的用户名，这就是上面代码的功能体现：


	```python
	def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
	```
	owner会在一个用户创建Snippet时得到该用户的信息就是这么来的