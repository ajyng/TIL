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

### Class Based View

- View 함수를 만들어주는 클래스. (as_view() 클래스 함수를 통해, View 함수를 생성한다)
- 상속을 통해서 여러 기능들을 믹스인한다.

### CBV 컨셉 구현

```python
#함수로 구현
def generate_view_fn(model):
	def view_fn(request, id):
		instance = get_object_or_404(model, id=id)
		instance_name = model._meta.model_name
		template_name = '{}/{}_detail.html'.format(model._meta.app_label, instance_name)
		return render(request, template_name, {
			instance_name = instance,
		})
	return view_fn
# generate_view_fn 함수가 실행될 때마다 매번 새로운 함수가 정의된다.

post_detail = generate_view_fn(Post)
article_detail = generate_view_fn(Article)
```

```python
# 클래스로 구현
class DetailView:
	def __init__(self,model):
		self.model = model

	def get_object(self, *args, **kwargs):
		return get_object_or_404(self.model, id=kwargs['id'])

	def get_template_name(self):
		return '{}/{}_detail.html'.format(
			self.model_meta.app_label,
			self.model._meta.model_name)
	
	def dispatch(self, request, *args, **kwargs):
		object = self.get_object(*args, **kwargs)
		return render(request, self.get_template_name(), {
			self.model._meta.model_name: object,
		})
	
	@classmethod
	def as_view(cls, model):
		def view(request, *args, **kwargs):
			self = cls(model)
			return self.dispatch(request, *args, **kwargs)
		return view
```

```python
# 장고 기본 제공 CBV 활용
from django.views.generic import DetailView

post_detail = DetailView.as_view(model=Post, pk_url_kwarg='id') # pk_url_kwarg의 디폴트 값은 pk
article_detail = DetailView.as_view(model=Article, pk_url_kwarg='id')

------------------------------------------------------------------------

#상속을 통한 CBV 속성 정의
from django.view.generic import DetailView

class PostDetailView(DetailView):
	model = Post
	pk_url_kwarg = 'id'

post_detail = PostDetailView.as_view()
```

### Tip

- CBV가 정한 관례대로 개발할 경우, 아주 적은 양의 코드로 구현할 수 있다.
- 그 관례에 대해 이해하기 위해서는 FBV를 통한 개발 경험이 큰 도움.

---

### Built in CBV API

- ***Base view*** : View, TemplateView, RedirectView
- ***Generic display views*** : DetailView, ListView
- ***Generic date views*** : ArchiveIndexView, YearArchiveView, MonthArchiveView, WeekArchiveView, DayArchiveView, TodayArchiveView, DateDetailView
- ***Generic editing views*** : FormView, CreateView, UpdateView, DeleteView

### Base View

- django/views/generic/base.py에 위치
1. **View**
    - 모든 CBV의 모체가 된다. (직접 사용할 일은 거의 없다)
    - **HTTP method 별로 지정 이름의 멤버 함수**를 호출하도록 구현되어 있다.
    - *CBV.as_view(**initkwargs)*
2. **TemplateView**
    - *class TemplateView(TemplateResponseMixin, ContextMixin, View)*

    ```python
    from django.views.generic import TemplateView

    urlpatterns = [
    	path('', TemplateView.as_view(template_name='root.html'), name='root'),
    	..
    	..
    ]		
    ```

3. **RedirectView**

    ```python
    class RedirectView(View):
    	permanent = False # True는 301 응답(영구적 이동), False는 302 응답(임시 이동)
    	url = None # 이동할 URL 문자열
    	pattern_name = None # URL Reverse을 수행할 문자열
    	query_string = False # QueryString을 그대로 넘길 것인지 여부
    ```

    ```python
    from django.views.generic import RedirectView

    urlpatterns = [
    	path('', RedirectView.as_view(
    		url = '/instagram' #방법 1
    		pattern_name = 'instagram:post_list', #방법 2 (더 선호하는 방식)
    	), name='root'),
    	..
    	..
    ]
    ```

    ---

    ### Generic display views

    1. **DetailView**
        - ***1개 모델의 1개 Object에 대한 템플릿 처리***
        - *모델명(소문자)이라는 이름의 Model Instance*을 **템플릿에 전달**한다.
        - pk/slug을 직접 명시하지 않아도 내부적으로 찾아서 처리한다.
        - template_name이 지정되지 않았다면 모델명으로 템플릿 경로를 유추한다.

        ```python
        class PostDetailView(DetailView):
        	model = Post

        	def get_queryset(self): # DetailView 내부의 **get_queryset 메소드를 커스터마이징**해서 로그인한 유저만 읽기 권한을 준다.
        		qs = super().get_queryset()
        		if not self.request.user.is_authenticated:
        			qs = qs.filter(is_public=True) # filter을 걸어줄 수 있다.
        			return qs

        post_detail = PostDetailView.as_view()
        ```

    2. **ListView**
        - ***1개 모델에 대한 List 템플릿 처리***
        - *모델명(소문자)_list 라는 이름의 QuerySet*을 **템플릿에 전달**한다.
        - **페이징 처리를 지원**한다
        - template_name이 지정되지 않았다면 모델명으로 템플릿 경로를 유추한다.
        - DetailView 와의 차이는, 1개를 반환하느냐(DetailView) 목록을 반환하느냐(ListView).

        ```python
        post_list = ListView.as_view(model=Post, paginate_by=10)
        ```

        ---

        ### 장식자(Decorators)

        - ***어떤 함수를 감싸는(wrapping) 함수***

        ```python
        from django.shortcuts import render
        from django.contrib.auth.decorator import login_required

        #방법1
        @login_required
        def protected_view1(request):
        	return render(request, 'myapp/secret.html')

        #방법2
        def protected_view2(request):
        	return render(request, 'myapp/secret.html')
        protected_view2 = login_require(protected_view2)
        ```

        ### 몇 가지 장고 기본 Decorators

        1. ***django.views.decorators.http***
            - require_http_methods, require_GET, require_POST, require_safe
            - 지정한 method가 아닐 경우 HttpResponseNotAllowed 응답 반환
        2. ***django.contrib.auth.decorators***
            - user_passes_test : 지정 함수가 False를 반환하면 login_url로 redirect
            - login_required : 로그아웃 상황에서 login_url로 redirect (디폴트는 '/account/login/')
            - permission_required : 지정 퍼미션이 없을 때 login_url로 redirect
        3. django.contrib.admin.views.decorators
            - staff_member_required : staff member가 아닐 경우 login_url로 redirect

        ### CBV에 장식자 입히기

        ```python
        **#방법1**
        #가독성이 좋지 않아서 추천하지 않는 방법
        from django.contrib.auth.decorators import login_required
        from django.views.generic import TemplateView

        class SecretView(TemplateView):
        	template_name = 'myapp/secret.html'

        view_fn = SecretView.as_view()

        secret_view = login_required(view_fn) # as_view()로 만들어진 반환 값을 장식자에 넣어서 urls에 바로 매핑
        ```

        ```python
        **#방법2**
        #dispatch 함수를 재정의(실제 요청이 왔을 때 처리되는 method)
        ****from django.contrib.auth.decorators import login_required
        from django.utils.decorators import method_decorator
        from django.views.generic import TemplateView

        class SecretView(TemplateView):
        	template_name = 'myapp/secret.html'

        	@method_decorator
        	def dispatch(login_required):
        		return super().dispatch(*args, **kwargs)

        secret_view = login_required(view_fn)
        ```

        ```python
        #방법3(추천)
        #클래스에 직접 적용
        from django.contrib.auth.decorators import login_required
        from django.utils.decorators import method_decorator
        from django.views.generic import TemplateView

        @method_decorator(login_required, name='dispatch')
        class SecretView(TemplateView):
        	template_name = 'myapp/secret.html'

        secret_view = SecretView.as_view()
        ```

    ---

    ### Generic date views

    - *ArchiveIndexView* : 지정 날짜필드 역순으로 정렬된 목록
    - *YearArchiveView* : 지정 year년도의 목록
    - *MonthArchiveView* : 지정 year/month 월의 목록
    - *DayArchiveView* : 지정 year/month/day 일의 목록
    - *TodayArchiveView* : 오늘 날짜의 목록
    - *DateDetailView* : 지정 year/month/day 목록 중에서 특정 pk의 detail (URL에 year/month/day를 쓰고자 할 때 유용)

    ```python
    from django.views.generic import ArchiveIndexView

    post_archive = ArchiveIndexView.as_view(model=Post, date_field='created_at', paginate_by=10)
    ```

    ```python
    urlpatterns = [
    	path('archive/', views.post_archive, name='post_archive'),
    	..
    	..
    ]
    ```

    ```python
    <h2>latest</h2>
    {{ latest }}

    <h2>date_list</h2>
    {% for date in date_list %}
        {{ date.year }}
    {% endfor %}
    ```

    ---

    ### HTTP 상태코드

    - 웹서버는 적절한 상태코드로서 응답해야 한다.
    - 각 HttpResponse 클래스마다 고유한 status_code가 할당된다.
    1. 200번대 : 성공
    2. 300번대 : 요청을 마치기 위해서는 추가 조치가 필요하다
    3. 400번대 : 클라이언트측 오류
    4. 500번대 : 서버측 오류 (뷰에서 요청 처리 중에, 뷰에서 미처 잡지못한 오류가 발생했을 경우)

---

### URL Dispatcher

- [***urls.py](http://urls.py) 변경***만으로 ***<각 뷰에 대한 URL>이 변경***되는 유연한 URL 시스템
- 개발자가 일일이 URL을 계산할 필요 없이 URL Reverse가 변경된 URL을 추적.

### URL Reverse를 수행하는 4가지 함수

1. ***url 템플릿 태그***
    - 내부적으로 reverse 함수 사용

    ```python
    {% for post in post_list %}
    	..
    	<a href="{% url 'instagram:post_detail' post.pk %}">
    	..
    {% endfor %}
    ```

2. ***reverse 함수***
    - 매칭 URL이 없으면 NoReverseMatch 예외 발생

    ```python
    from django.core.urlresolvers import reverse

    reverse('blog:post_list') # '/blog/'
    reverse('blog:post_detail', args=[10]) # '/blog/10/' args 인자로 리스트 지정 필요
    reverse('blog:post_detail', kwargs={'id':10}) # '/blog/10/'
    reverse('/hello/') # NoReverseMatch 오류 발생
    ```

3. ***resolve_url 함수***
    - 내부적으로 reverse 함수 사용
    - 매칭 URL이 없으면 ***'인자 문자열'***을 그대로 리턴

    ```python
    from django.shortcuts import resolve_url

    resolve_url('blog:post_list') # '/blog/'
    resolve_url('blog:post_detail', 10) # '/blog/10/'
    resolve_url('blog:post_detail', id=10) # '/blog/10/'
    resolve_url('/hello/') # '/hello/' 문자열 그대로 리턴
    ```

4. ***redirect 함수***
    - 내부적으로 resolve_url 함수 사용
    - 매칭 URL이 없으면 ***'인자 문자열'을 그대로 URL로 사용***
    - ***HttpResponse의 instance***를 만들기 때문에 view에서 바로 사용 가능할 수 있다.

    ```python
    from django.shortcuts import redirect

    redirect('blog:post_detail', 10)
    # <HttpResponseRedirect status_code=302, "text/html; charset=utf-8", url="/blog/10/">
    ```

### 모델 클래스에 get_absolute_url() 구현

***특정 모델에 대한 Detail 뷰***에 대한 URLConf설정을 하자마자, 필히 ***get_absolute 설정***을 하면 코드가 보다 간결해진다!

- resolve_url 함수는 가장 먼저 get_absolute_url() 함수의 존재여부를 체크하고, 존재할 경우 reverse를 수행하지 않고 그 리턴값을 즉시 리턴한다.
- 어떠한 모델에 대해서 detail 뷰를 만들게 되면 get_absolute_url() 멤버 함수를 무조건 선언

```python
class Post(models.Model):
	..
	def get_absolute_url(self):
        return reverse('instagram:post_detail', args=[self.pk])
```

```python
{% for post in post_list %}
	..
	<a href="{{ post.get_absolute_url }}">
	..
{% endfor %}
```

- ***CreateView, UpdateView***에 *success_url*을 제공하지 않는 경우 **해당 model instance의 get_absolute_url 주소로 이동이 가능한지 체크**한다.