# 직렬화(Serialization)

생성일: 2021년 2월 2일 오후 9:26

## 직렬화(Serialization)

![%E1%84%8C%E1%85%B5%E1%86%A8%E1%84%85%E1%85%A7%E1%86%AF%E1%84%92%E1%85%AA(Serialization)%2097dfcaacac17441480f8aba852adbe23/_2021-02-04__1.45.27.png](%E1%84%8C%E1%85%B5%E1%86%A8%E1%84%85%E1%85%A7%E1%86%AF%E1%84%92%E1%85%AA(Serialization)%2097dfcaacac17441480f8aba852adbe23/_2021-02-04__1.45.27.png)

모든 프로그래밍 언어의 통신에서 ***데이터는 문자열로 표현***되어야 한다.

그렇기 때문에 송신자는 객체를 문자열로 변환하여 데이터를 전송한다. (**직렬화**)

수신자는 받은 문자열을 다시 객체로 변환하여 활용한다. (**비직렬화**)

최근 API 서버는 거의 ***JSON 형태의 문자열***로 변환하여 사용한다!

```json
// JSON 문자열 예시
{
	"pk": 1,
	"name": "Ahn Jun Young",
	"user_id": "ajyng",
}
```

---

## DJango 직렬화

### 1. JSON 라이브러리

파이썬에는 직렬화/비직렬화를 위해 다양한 방법을 제공한다. 대표적으로 `JSON`, `PICKLE`과 같은 방식이 있는데, 장고 타입에 대한 직렬화 rule이 존재하지 않아서 따로 추가해줘야 한다!!

```python
import json
from django.contrib.auth import get_user_model
User = get_user_model()
json.dumps(User.objects.first()) # User 모델에 대한 직렬화 rule을 모르기 때문에 에러가 발생!
```

### 2. DjangoJSONEncoder

따라서 `JSON` 라이브러리를 상속 받아 몇 가지 직렬화 rule을 추가한 `DjangoJSONEncoder`를 제공한다.

하지만 그래봤자 Model, QuerySet 인스턴스에 대한 직렬화를 할 수 없다.

```python
import json
from django.core.serializer.json import DjangoJSONEncoder
from django.contrib.auth import get_user_model
qs = get_user_model().objects.all()
json_string = json.dumps(qs, cls=DjangoJSONEncoder) # 여기서도 User 모델에 대한 직렬화 에러가 발생한다!
```

**~~*정 하고 싶다면~~*** 장고 타입 인스턴스를 하나씩 순회하면서 직렬화 가능한 형태로 바꿔줄 수 있다.

```python
data = [
	{'id': post.id, 'title': post.title, 'content': post.content}
	for post in Post.objects.all()]
json.dumps(data, ensure_ascii=False) # 이제 data는 단순 리스트이기 때문에 직렬화 가능
```

~~***추가로***~~  커스텀을 통해 직접 직렬화 rule을 추가해주는 방법도 있다.

실제로 DRF에서도 `JSONEncoder`를 커스텀해서 사용한다 !!

```python
from django.core.serializer.json import DjangoJSONEncoder
from django.db.models.query import QuerySet

class MYJSONEncoder(DjangoJSONEncoder):
	def default(self, obj):
		if isinstance(obj, QuerySet):
			return tuple(obj)
		elif isinstance(obj, Post): # obj가 Post 모델인 경우 직렬화 rule을 return,
			return {'id': obj.id, 'title': obj.title, 'content': obj.content}
		elif hasattr(obj, 'as_dict'):
			return obj.as_dict()
		return super().default(obj)

data = Post.objects.all()
json.dumps(data, cls=MYJSONEncoder, ensure_ascii=False) # 이거도 이제 가능 ~~
```

---

## 그렇다면 DRF에서는 어떻게 직렬화를 수행할까?

우선 ~~*장고의 DjangoJSONEncoder가 아니라*~~ `json.JSONEncoder`를 상속 받아 구현한 `rest_framework.utils.encoders.JSONEncoder`를 통해 직렬화를 수행한다.

우리는 `rest_framework.renderer.JSONRenderer`를 통해 해당 직렬화를 더 편리하게 할 수 있다. 내부적으로는 `json.dumps` 함수를 사용하며 utf-8 인코딩도 추가로 수행해준다고 한다! (

하지만 이렇게 해도 아직 장고 모델 타입에 대한 직렬화는 수행할 수가 없다,, (까다로운 놈이다)

### Serializer/ModelSerializer

[Serializers](https://www.django-rest-framework.org/api-guide/serializers/#serializers)

> *Serializers allow complex data such as querysets and model instances to be converted to native Python datatypes that can then be easily rendered into JSON, XML or other content types.*

- `Serializer`/`ModelSerializer`의 사용을 통해 드디어 장고 모델 타입에 대한 직렬화를 수행할 수 있다!!
- `Serializer`는 간단히 말해서 Model, QuerySet 인스턴스와 같은 데이터를 JSON, XML 등과 같은 컨텐트 타입으로 변환이 가능하게끔 ***파이썬 기본 데이터 타입***으로 먼저 변환시켜 준다.
- 사용법은 장고 `Form`/`ModelForm`과 유사하다.

```python
from rest_framework.serializers import ModelSerializer

class PostSerializer(ModelSerializer):
	class Meta:
		model = Post
		fields = '__all__'

post = Post.objects.first()

serializer = PostSerializer(post) # 다수를 넘길 땐 many=True 속성을 설정! default는 False
serializer.data
```

![%E1%84%8C%E1%85%B5%E1%86%A8%E1%84%85%E1%85%A7%E1%86%AF%E1%84%92%E1%85%AA(Serialization)%2097dfcaacac17441480f8aba852adbe23/_2021-02-04__2.30.40.png](%E1%84%8C%E1%85%B5%E1%86%A8%E1%84%85%E1%85%A7%E1%86%AF%E1%84%92%E1%85%AA(Serialization)%2097dfcaacac17441480f8aba852adbe23/_2021-02-04__2.30.40.png)

출력 결과에서 볼 수 있듯이 Post 모델 인스턴스가 `ReturnDict` 타입으로 변환되었다!

*(ReturnDict 타입은 파이썬의 OrderDict를 상속 받은 데이터 타입이다)*

드디어 드디어 장고 모델 타입을 직렬화 할 수 있게 되었다!

![%E1%84%8C%E1%85%B5%E1%86%A8%E1%84%85%E1%85%A7%E1%86%AF%E1%84%92%E1%85%AA(Serialization)%2097dfcaacac17441480f8aba852adbe23/_2021-02-04__2.38.17.png](%E1%84%8C%E1%85%B5%E1%86%A8%E1%84%85%E1%85%A7%E1%86%AF%E1%84%92%E1%85%AA(Serialization)%2097dfcaacac17441480f8aba852adbe23/_2021-02-04__2.38.17.png)

`json_str_python`은 파이썬 내장 모듈을 활용하여 직렬화를 마무리 했고, `json_str_drf`는 DRF의 `JSONRenderer`를 활용하였다. 이제서야 직렬화 프로세스를 모두 마치게 되는 것이다!

*(출력하는 문자열이 다른 이유는 서로 적용된 옵션이 다르기 때문이다)*

이제 직렬화가 어떤 것이고, 장고와 DRF에서 직렬화를 어떻게 수행하는지까지 정리했다!

---

## JSON 응답을 하는 방법

위에서 얘기한 거처럼 최근 API 서버에서는 JSON 형태의 요청/응답을 통해 서로 통신한다.

## *장고에서*

기본적으로 모든 View에서는 `HttpReponse` 타입의 JSON 응답을 해야 한다. 방법은 2가지이다.

1. `json.dumps()`를 통해 직렬화 → `HttpReponse()`를 통해 응답
2. `JsonResponse()`를 통해 1번 과정을 한꺼번에 수행

```python
qs = Post.objects.all()

encoder = MYJSONEncoder
safe = False # True: data가 dict인 경우 / False: dict가 아닌 경우
json_dumps_params = {'ensure_ascii': False}
kwargs = {}

from Django.http import JsonResponse
response = JsonResponse(qs, encoder, safe, json_dumps_params, **kwargs)
```

### *DRF에서*

[Responses](https://www.django-rest-framework.org/api-guide/responses/#response)

DRF에서는 `rest_framework.response.Response`를 활용할 수 있다!

```python
qs = Post.objects.all()
serializer = PostSerializer(qs, many=True)

from rest_framework.response import Response
response = Response(serializer.data)
# serializer.data의 직렬화를 마무리하고, HttpResponse 형태로 응답까지 만들어준다.
```

`Response(data, status=None, template_name=None, headers=None, content_type=None)`

이게 Response 함수의 원형이다. 굳이 하나씩 파보자면

`data`: 직렬화된 데이터

`status`: 응답 상태 코드. (디폴트 200)

`template_name`: HTMLRenderer로 렌더링한다면, 그때 사용될 template.

`headers`: 응답 헤더

`content_type`: 응답의 콘텐츠 타입인데 일반적으로는 Renderer가 알아서 지정한다. (디폴트 text/html)

---

## DRF Serializer 활용 예시

```python
from rest_framework.generics import ListAPIView

class PostListAPIView(ListAPIView):
	qs = Post.objects.all()
	serializer_class = PostSerializer # 이렇게 간단하게 사용 가능하다~!~!

post_list = PostListAPIView.as_view()
```

추가적으로 장고 모델에 `외래키` 필드가 있을 때, Serializer는 `외래키`의 키값으로 응답한다.

예를 들어 "author": 1 과 같이 해당 필드의 키값이 응답으로 온다~~ 하지만 우리는 그걸 원하는 게 아니라 실제 값을 사용하고자 할 때가 많을테니까 해결 방법이 필요하다.

### 1. serializers.***ReadOnlyField***

```python
from rest_framework.serializers import ModelSerializer, ReadOnlyField
from .models import Post

class PostSerializer(ModelSerializer):
	**username** = ReadOnlyField(source='author.username)

	class Meta:
		model = Post
		fields = ['pk', **'username'**, 'title', 'content']
```

### *2. Serializer 중첩*

```python
from django.contrib.auth import get_user_model
from rest_framework.serializers import ModelSerializer
from .models import Post

class **AuthorSerializer**(ModelSerializer):
	class Meta:
		model = get_user_model()
		fields = ['username']

class PostSerializer(ModelSerializer):
	author = **AuthorSerializer()**

	class Meta:
		model = Post
		fields = ['pk', 'author', 'title', 'content']
```