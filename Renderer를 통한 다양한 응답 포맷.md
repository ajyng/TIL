# Renderer를 통한 다양한 응답 포맷

생성일: 2021년 2월 6일 오후 3:59

DRF에서는 다양한 `Renderer`를 통해서 Model, QuerySet 객체를 다양한 포맷으로 변환해주는 기능을 제공한다!

대표적으로 `JSONRenderer`가 있으며, 써드파티 중에는 `XLSXRenderer` (엑셀 포맷으로 변환) 도 있다. 필요하다면 이미지나 docx/pptx 파일로 생성해주는 `Renderer`도 만들어볼 수 있다!

*~~대표적인 몇 개만 알아보자.~~*

### 1. *JSONRenderer (Default)*

[Renderers](https://www.django-rest-framework.org/api-guide/renderers/#jsonrenderer)

내부적으로 `json.dumps`를 활용하여 JSON 직렬화 결과를 렌더링 한다!

- .media_type: `application/json`
- .format: `'json'`

### 2. *BrowsableAPIRenderer (Default)*

[Renderers](https://www.django-rest-framework.org/api-guide/renderers/#browsableapirenderer)

![Renderer%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%83%E1%85%A1%E1%84%8B%E1%85%A3%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%B3%E1%86%BC%E1%84%83%E1%85%A1%E1%86%B8%20%E1%84%91%E1%85%A9%E1%84%86%E1%85%A2%E1%86%BA%20e78ef2d957d04c9c9404d00d51eea4e2/_2021-02-07__4.26.55.png](Renderer%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%83%E1%85%A1%E1%84%8B%E1%85%A3%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%B3%E1%86%BC%E1%84%83%E1%85%A1%E1%86%B8%20%E1%84%91%E1%85%A9%E1%84%86%E1%85%A2%E1%86%BA%20e78ef2d957d04c9c9404d00d51eea4e2/_2021-02-07__4.26.55.png)

가장 높은 우선순위가 주어졌을 `renderer`를 결정하고, 해당 `renderer`를 사용하여 HTML 페이지 내에 위와 같이 API 스타일 응답을 표시한다!

웹브라우저로 단순히 접근하게 되면 `Browsable API` 방식으로 처리가 된다. 이는 브라우저를 통한 요청에서는 `Accept` 헤더에 `text/html` 이 포함되어있어서 그렇다!

만약 `BrowsableAPIRenderer`가 없었다면 같은 주소로 GET 요청을 보내더라도 다음과 같은 응답을 받는다!!

![Renderer%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%83%E1%85%A1%E1%84%8B%E1%85%A3%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%B3%E1%86%BC%E1%84%83%E1%85%A1%E1%86%B8%20%E1%84%91%E1%85%A9%E1%84%86%E1%85%A2%E1%86%BA%20e78ef2d957d04c9c9404d00d51eea4e2/_2021-02-07__4.30.07.png](Renderer%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%83%E1%85%A1%E1%84%8B%E1%85%A3%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%B3%E1%86%BC%E1%84%83%E1%85%A1%E1%86%B8%20%E1%84%91%E1%85%A9%E1%84%86%E1%85%A2%E1%86%BA%20e78ef2d957d04c9c9404d00d51eea4e2/_2021-02-07__4.30.07.png)

또한 *실제 사용하는 `renderer`*와, `*browsable API`에서 사용하는 `renderer`*를 달리 하고 싶다면 커스텀할 수 있다. 예를 들어 HTML을 기본 응답 `format`으로 사용하지만, `browsable API`에서는 JSON 형태의 응답을 받고 싶다면 아래와 같은 ***Custom Browsable Renderer***를 작성할 수 있다.

```python
class CustomBrowsableAPIRenderer(BrowsableAPIRenderer):
	def get_default_renderer(self, view):
		return JSONRenderer()
```

- .media_type: `text/html`
- .format: `'api'`

### 3. *TemplateHTMLRenderer*

[Renderers](https://www.django-rest-framework.org/api-guide/renderers/#templatehtmlrenderer)

직접 지정한 Template을 통한 렌더링을 할 수 있다. 그렇기 때문에 `template_name`을 따로 지정해줘야 한다.

또한 다른 `renderer`와는 다르게 해당 template을 통해 render를 수행하기 때문에 별도의 serializer가 필요하지 않다.

```python
class UserDetail(RetrieveAPIView):
	queryset = User.objects.all()
	renderer_classes = [TemplateHTMLRenderer]
	
	def get(self, request, *args, **kwargs):
		self.object = self.get_object()
		return Response({'user': self.object}, template_name='user_detail.html')
```

```python
class UserDetail(RetrieveAPIView):
	queryset = User.objects.all()
	renderer_classes = [TemplateHTMLRenderer]
	template_name='user_detail.html'
	
	def get(self, request, *args, **kwargs):
		self.object = self.get_object()
		return Response({'user': self.object})
```

`TemplateHTMLRenderer`를 활용한 웹사이트를 설계할 때는 `renderer_classes`의 가장 첫번째로 작성해주는 게 좋다. 그래야 브라우저가 요청 헤더를 잘못 보내는 경우에도 우선순위가 설정된다.

- .media_type: `text/html`
- .format: `'html'`

## 그렇다면 Renderer는 어떻게 결정될까?

view에 대해 유효한 `renderer`는 항상 ***클래스들의 리스트***로 주어진다.

요청이 들어오면 DRF가 `content negotiation`을 수행하여 ***가장 적합한 renderer***을 결정한다!

`content negotiation`은 기본적으로 요청의 `**Accept` 헤더**를 검사하여 media types을 결정한다.

또한 선택적으로 **URL** **suffix**에 Format을 지정하는 경우도 있다. 예를 들어 *`/post/?format=json`, `/post.json`*와 같다.

### *Renderer 클래스 리스트 지정*

```python
REST_FRAMEWORK = {
	'DEFAULT_RENDERER_CLASSES': [
	    'rest_framework.renderers.JSONRenderer',
	    'rest_framework.renderers.BrowsableAPIRenderer',
	]
}
```

```python
class UserCountView(APIView):
  renderer_classes = [JSONRenderer]

  def get(self, request, format=None):
      user_count = User.objects.filter(active=True).count()
      content = {'user_count': user_count}
      return Response(content)
```

```python
@api_view(['GET'])
@renderer_classes([JSONRenderer])
def user_count_view(request, format=None):
  user_count = User.objects.filter(active=True).count()
  content = {'user_count': user_count}
  return Response(content)
```