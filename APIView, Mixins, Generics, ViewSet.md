# APIView, Mixins, Generics, ViewSet

생성일: 2021년 2월 6일 오전 12:24

장고는 기본적으로 view를 통해 HTTP 요청을 처리한다.

이때 DRF는 다양한 방법을 통해 view를 더욱 간단하게 처리할 수 있다~!

## 1. APIView

`APIView`는 DRF의 가장 기본이 되는 클래스 기반 뷰(CBV)이다.

1) `***APIView` 클래스를 상속 받은 클래스 기반 뷰*** 또는 2) `***@api_view` 장식자를 통한 함수 기반 뷰***를 통해 활용할 수 있다. 

`APIView`는 view에게 다양한 기본 속성들을 제공한다. (디폴트 값은 귀찮으니까 생략,,)

1. *`renderer_classes` : 직렬화 클래스*
2. *`parser_classes` : 비직렬화 클래스*
3. *`authentication_classes` : 인증 클래스*
4. *`throttle_classes` : 사용량 제한 클래스*
5. *`permission_classes` : 권한 클래스*
6. *`content_negotiation_class` : 요청에 따라 적절한 직렬화/비직렬화 클래스를 선택하는 클래스*
7. *`metadata_class` : 메타 정보를 처리하는 클래스*
8. *`versioning_class` : 요청에서 API 버전 정보를 탐지하는 클래스*

우선 각 method(GET, POST, PUT, DELETE)에 맞게 ***멤버 함수***를 구현하면, 해당 method 요청이 들어올 때 호출된다. 

```python
class **PostListAPIView**(APIView):
	def get(self, request):
		qs = Post.objects.all()
		serializer = PostSerializer(qs, many=True)
		return Response(serializer.data)

	def post(self, request):
		serializer = PostSerializer(data=request.data) # serializer의 유효성 검사
		if serializer.is_valid():
			serializer.save()
			return Response(serializer.data, status=201)
		return Response(serializer.errors, status=400)
```

```python
class **PostDetailAPIView**(APIView):
	def get_object(self, pk):
		return get_object_or_404(Post, pk=pk)

	def get(self, request, pk, format=None):
		post = self.get_object(pk)
		serializer = PostSerializer(post)
		return Response(serializer.data)

	def put(self, request, pk):
		post = self.get_object(pk)
		serializer = PostSerializer(post, data=request.data)
		if serializer.is_valid():
			serializer.save()
			return Response(serializer.data)
		return Response(serializer.errors, status=HTTP_400_BAD_REQUEST)

	def delete(self, request, pk):
		post = self.get_object(pk)
		post.delete()
		return Response(status=HTTP_204_NO_CONTENT)
```

이처럼 `APIView`를 상속 받은 클래스에, method에 맞는 멤버 함수를 구현해서 사용하면 된다.

이때 `APIView`가 제공하는 기능 중에

1. *직렬화/비직렬화 처리 (JSON, ..)*
2. *인증 체크*
3. *사용량 제한 체크 : 호출 허용량 범위인지 체크*
4. *권한 클래스 지정 : 비인증/인증 유저에 대해 해당 API 호출을 허용할 건지 결정*
5. *요청된 API 버전 문자열을 탐지하여 request.version에 저장*

가 있는데, ***얘네를 모두 거친 뒤에 멤버 함수가 호출***된다! 또한 `APIView` 자체가 이미 `csrf_exempt` 장식자로 감싸져있기 때문에 POST 요청에 따로 인증을 위해 `csrf_token` 체크를 하지 않아도 된다! 따라서 일일이 구현해야 한다는 귀찮음이 사라지는 굉장한 장점이 있다.

### (추가) *함수 기반 뷰로는 어떻게 구현할까?*

```python
@api_view(['GET', 'POST'])
def post_list(request):
	if request.method == 'GET':
		serializer = PostSerializer(Post.objects.all(), many=True)
		return Response(serializer.data)
	
	else:
		serializer = PostSerializer(data=request.data)
		if serializer.is_valid():
			serializer.save()
			return Response(serializer.data, status=201)
		return Response(serializer.errors, status=400)
```

```python
@api_view(['GET', 'PUT', 'DELETE'])
def post_detail(request, pk):
	post = get_object_or_404(Post, pk=pk)

	if request.method == 'GET':
		serializer = PostSerializer(post)
		return Response(serializer.data)

	elif request.method == 'PUT':
		serializer = PostSerializer(post, data=request.data)
		if serializer.is_valid():
			serializer.save()
			return Response(serializer.data)
		return Response(serializer.errors, status=HTTP_400_BAD_REQUEST)
	
	else:
		post.delete()
		return Response(status=HTTP_204_NO_CONTENT)
```

`@api_view` 장식자를 사용할 때, 매칭할 method를 명시해주면 된다. 만약 해당 리스트를 비워두면 아예 매칭 자체가 되지 않는다~!

일반적으로 ***하나의 작업***만을 구현하려고 할 땐 `@api_view`를 통해 구현하는 게 편리하다~ (상황에 맞게)

---

## 2. Mixins, Generics

### *Mixins*

위에서 본 것처럼 `APIView`를 통해 view를 구성하면 ***반복되는 로직***들이 생긴다. 하지만 프로그래밍은 반복을 최소화하는 게 좋다~! 그래서 반복되는 로직들을 `mixin` 이라는 이름으로 묶어버렸다.

[encode/django-rest-framework](https://github.com/encode/django-rest-framework/blob/19655edbf782aa1fbdd7f8cd56ff9e0b7786ad3c/rest_framework/mixins.py)

DRF에서 기본으로 제공하는 `mixin`은 5개가 있다. (물론 커스텀으로 다양한 `mixin`을 만들 수도 있다)

1. *CreateModelMixin*
2. *ListModelMixin*
3. *RetrieveModelMixin*
4. *UpdateModelMixin*
5. *DestroyModelMixin*

실제로 `mixin` 클래스가 ~~직접적으로 사용되지는 않고,~~ 다른 클래스에 의해 **상속**되어질 때 사용된다.또한 HTTP method(GET, POST, PUT, DELETE)와 매칭하는 부분 없이 ***단순히 로직만을 구현***한다!

```python
class CreateModelMixin:
	"""
	Create a model instance.
	"""
	def create(self, request, *args, **kwargs):
	    serializer = self.get_serializer(data=request.data)
	    serializer.is_valid(raise_exception=True)
	    self.perform_create(serializer)
	    headers = self.get_success_headers(serializer.data)
	    return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)
	
	def perform_create(self, serializer):
	    serializer.save()
	
	def get_success_headers(self, data):
	    try:
	        return {'Location': str(data[api_settings.URL_FIELD_NAME])}
	    except (TypeError, KeyError):
	        return {}
```

(뭐 이런식,,)

그렇다면 정확하게 어떻게 사용한다는 걸까?

```python
class PostListAPIView(ListModelMixin, CreateModelMixin, GenericAPIView):
	qs = Post.objects.all()
	serializer_class = PostSerializer

	def get(self, request, *args, **kwargs):
		return self.list(request, *args, **kwargs)

	def post(self, request, *args, **kwargs):
		return self.create(request, *args, **kwargs)
```

이처럼 구현한 `PostListAPIView`로 GET 요청이 들어올 땐 `self.list` 함수가 호출된다. 바로 이게 `ListModelMixin`으로부터 상속 받은 함수다! POST 요청이 들어왔을 때 호출되는 `self.create` 함수 또한 `CreateModelMixin`으로부터 상속 받은 함수이다.

또한 여기서 `CreateModelMixin`의 코드를 다시 한번 살펴보면 `create` 함수와 `perform_create` 함수가 있다.

이름이 비슷하게 생겨서 어떤 차이가 있을까 봤는데

실질적인 저장은 `perform_create` 에서 이루어졌다. (`create`에서도 `perform_create`을 내부적으로 호출한다)

따라서 사용자에게 받은 입력 이외에 추가적으로 저장해야할 정보들은 여기서 처리해주면 될 듯 하다!

```python
def perform_create(self, serializer):
	serializer.save(author=self.request.user)
```

불필요한 중복을 없앴지만 이렇게 매번 method와 연결을 해야 하니 여전히 번거로운 건 마찬가지다,,

---

### *Generics APIView*

그래서 등장한 건 바로 `generics APIView`가 되겠다.

[encode/django-rest-framework](https://github.com/encode/django-rest-framework/blob/19655edbf782aa1fbdd7f8cd56ff9e0b7786ad3c/rest_framework/generics.py)

우리가 일반적으로 사용하는 경우에 대해서 HTTP method를 미리 매칭해둬서 사용하기 편리하다.

```python
class CreateAPIView(CreateModelMixin, GenericAPIView):
	"""
	Concrete view for creating a model instance.
	"""
	def post(self, request, *args, **kwargs):
	    return self.create(request, *args, **kwargs)
```

하나만 예를 들어보자면

이처럼 `CreateAPIView`를 통해 view를 처리하면, POST 요청이 들어왔을 때 알아서 `CreateModelMixin`의 `create` 함수를 사용한다. 그러니까 일일이 매핑시켜야 하는 수고를 덜었다~!

---

## 3. ViewSet

지금까지 나왔던 `APIView`, `Mixin`, `Generics APIView`는 모두 하나의 url만 처리할 수 있었다.

하지만 `ViewSet`은 단일 리소스에서 ***관련 있는 여러 view들을 단일 클래스에서 제공***한다!

무슨 말이냐 하면 '/post/'와 같은 ***List Route***에는 `list`, `create`를 매핑하고

'/post/2/'와 같은 ***Detail Route***에는 `detail`, `update`, `partial_update`, `delete`를 기본적으로 매핑한다.

이거를 하나의 클래스에서 구현할 수 있다~!

```python
class PostViewSet(ViewSet):
	"""
	A simple ViewSet for listing or retrieving users.
	"""
	def list(self, request): # GET '/post/' 요청이 들어왔을 때
		qs = Post.objects.all()
		serializer = PostSerializer(qs, many=True)
		return Response(serializer.data)

	def retrieve(self, request, pk): # GET '/post/<int:pk>/' 요청이 들어왔을 때
		qs = Post.objects.all()
		user = get_object_or_404(qs, pk=pk)
		serializer = PostSerializer(user)
		return Response(serializer.data)
```

```python
router = DefaultRouter()
router.register('post', PostViewSet, basename='post')

urlpatterns = [
path('', include(router.urls)),
]
```

이처럼 `PostViewSet`이라는 단일 클래스에서 2개의 url을 처리하고 있다.

또한 `router`라는 걸 통해서 url을 연결시켜주는데, 이를 통해 일괄적으로 등록할 수 있어서 편리하다!

물론 아래와 같이 개별 view를 만들어서 일일이 path를 연결해줄 수도 있다. (하지만 `router`을 주로 활용!)

```python
post_list = PostViewSet.as_view({'get': 'list'})
post_detail = PostViewSet.as_view({'get': 'retrieve'})
```

### *ModelViewSet*

`ModelViewSet`을 사용하면 `ViewSet` 내에서 ***Model, Serializer***를 통해서 ***자동으로 멤버 함수를 구현***해준다.

1. *ReadOnlyModelViewSet : `.list()`, `.retrieve()`*
2. *ModelViewSet : `.list()`, `.create()`, `.retrieve()`, `.update()`, `.partial_update()`, `.delete()`*

```python
class PostViewSet(ModelViewSet):
	queryset = Post.objects.all()
	serializer_class = PostSerializer
```

이렇게 사용하면 된다~!

만약 새로운 Endpoint를 추가하고 싶다면, `@action` 장식자를 활용하면 되는데

```python
class PostViewSet(ModelViewSet):
	queryset = Post.objects.all()
	serializer_class = PostSerializer
	
	@action(detail=False, methods=['GET'])
	def public(self, request):
	    qs = self.queryset.filter(is_public=True)
	    serializer = self.get_serializer(qs, many=True)
	    return Response(serializer.data)
```

이렇게 하면 '/post/public' URL이 매칭된다~!~!