# 장고 Views

생성일: 2020년 9월 2일 오후 7:49

### View

- [urls.py/urlpatterns](http://urls.py/urlpatterns) 리스트에 매핑된 ***호출 가능한 객체***를 의미한다.(함수도 그중 하나)
- 웹 클라이언트로부터 들어온 **HTTP 요청을 처리**한다.
- *1개의 HTTP 요청*에 대해서는, *1개의 view*가 호출된다.

종류1. ***함수 기반 뷰(Function Based View)*** : 장고 view의 기본. ***호출 가능한 객체 그 자체***를 의미한다.

종류2. ***클래스 기반 뷰(Class Based View)*** : ***as_view() 클래스 함수를*** 통해 ***호출 가능한 객체를 생성***하고 리턴한다. (클래스.as_view()와 같은 형태로 호출한다)

### View 호출 시, 인자

- 1번째 인자는 ***HttpRequest* 객체**를 가진다. (*현재 요청에 대한 모든 내역*을 담고 있습니다)
- 2번째 인자는, 현재 요청의 URL로부터 ***capture된 문자열들.***

```python
**# urls.py**
urlpatterns = [
	path('<int:pk>/', views.post_detail), # 정수 패턴이 있다면 pk라는 이름으로 넘겨준다.
	..
]

**# views.py**
define post_detail(request, pk): 
	...
```

- url이나 re_path 를 통한 처리에서는 **모든 인자를 str 타입으로 전달**한다.
- path 을 통한 처리에서는 매핑된 Converter을 **to_python에 의해서 변환한 값**을 전달.(위의 경우에는 int 값)

### View 호출에 대한 리턴값

- (!!)반드시 ***HttpResponse 객체를 반환***해야 한다. (그렇지 않으면 middleware에서 처리 오류)
- 파일like객체 또는 str/bytes 타입의 응답도 지원한다.

```python
response = HttpResponse(파일like객체 or str객체 or bytes객체)
response.write(str객체 or bytes 객체)
```

```python
define example(request):
	reponse = HttpResponse()
	reponse.write("Hello, World!") # response -> file-like object
	return reponse
```

- str 문자열을 직접 utf8로 인코딩 할 필요가 없다. (장고 디폴트 설정에서 수행)

### FBV의 예시

```python
**# views.py**
def post_detail(request, pk):
	post = get_object_or_404(Post, pk=pk)
	return render(request, 'instagram/post_detail.html',{
	'post' : post,
	})

**# urls.py**
urlpatterns = [
	path('<int:pk>/', views.post_detail),
	..
	]
```

### CBV의 예시

```python
# 1번째 방법
from django.views.generic import ListView

post_list = ListView.as_view(model=Post)

# 2번째 방법
from django.views.generic import ListView

class PostListView(ListView):
	model = Post
post_lit = PostListView.as_view()
```

---

### URL Dispatcher

- ***<특정 URL 패턴과 매칭되는 View>***의 List. *(urlpatterns)*
- settings.py에서 ROOT_URLCONF 모듈을 지정하고, 최초의 urlpatterns로부터 include를 통해 하위 구조의 urlpatterns을 확장해 나간다.(tree 구조) 결과적으로 **하나의 List 구조**로 유지를 하게 된다.
- HTTP 요청이 들어올 때마다, 등록된 urlpatterns 상의 매핑 리스트를 처음부터 순차적으로 탐색한다. **매칭되는 URL Rule이 여러 개가 존재한다면 처음 Rule만을 사용**한다. 만약 **매칭되는 URL Rule이 없는 경우 '404 Page Not Found' 에러가** 발생한다.

### path()와 re_path()

- Django 1.X에서의 django.conf.urls.url() 사용법이 ***path(), re_path()*** 2가지로 나눠졌다.
    1. path() : 기본 지원되는 Path converters를 통해 ***정규표현식 기입을 간소화***했다. (만능은 아니다)
        - *기본 지원되는 Path converters*

            **IntConverter** - r"[0-9]+"

            **StringConverter** - r"[^/]+"

            **UUIDConverter** - r"[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}"

            **SlugConverter** (StringConverter 상속) - r"[-a-zA-Z0-9_]+"

            **PathConverter** (StringConverter 상속) - r".+"

        - *자주 사용하는 패턴을 Converter로 등록*하면 재활용 측면에서 편리하다.

        ```python
        from django.urls import path,re_path, register_converter

        class YearConverter:
            regex = r"20\d{2}" # 20XX년만 입력 가능케 했다.
            
            def to_python(self,value): # url로부터 추출한 문자열을 뷰에 넘겨주기 전에 호출
                return int(value)
            def to_url(self,value): # url reverse 시에 호출
                return str(value)

        register_converter(YearConverter, 'year')

        urlpatterns = [
        	path('archives/<year:year>/', views.archives_year),
        	..
        ]
        ```

    2. re_path() : 기존 django.conf.urls.url()과 사용법이 동일.

### 정규표현식

- 거의 모든 프로그래밍 언어에서 지원
- ***문자열의 패턴, 규칙, Rule을 정의***하는 방법
- 문자열 검색이나 치환 작업을 간편하게 처리할 수 있다.
- *장고의 URL Dispatcher에서는 정규표현식을 통한 URL 매칭*이 이뤄진다.
- 정규표현식은 띄어쓰기 하나에도 민감하므로 주의해서 사용해야 한다.

1자리 숫자 - "[0123456789]" 혹은 "[0-9]" 혹은 r"[\d]" 혹은 r"\d"
2자리 숫자 - "[0123456789][0123456789]" 혹은 "[0-9][0-9]" 혹은 "\d\d"
3자리 숫자 - r"\d\d\d" 혹은 r"\d{3}"
2자리~4자리 숫자 - r"\d{2,4}"
휴대폰 번호 - r"010[1-9]\d{7}"
알파벳 소문자 1글자 -r "[abcdefghijklmnopqrstuvwxyz]" 혹은 "[a-z]"
알파벳 대문자 1글자 - r"[ABCDEFGHIJKLMNOPQRSTUVWXYZ]" 혹은 "[A-Z]"
한글이름 2글자 혹은 3글자 - r"[ㄱ-힣]{2,3}"
성이 "김"씨인 이름  - r"김[ㄱ-힣]{1,2}"

---