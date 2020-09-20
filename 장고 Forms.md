# 장고 Forms

생성일: 2020년 9월 20일 오전 3:27

### 언제 쓰나요?

- **HTML Form(클라이언트 측)** : 클라이언트에서 ***사용자에게 입력폼을 제공***하고, 이를 서버로 전송하고자 할 때 사용한다.
- **Django Form** : ***클라이언트로부터 전달받은 값들에 대한 유효성 검사를 수행***하고, 이를 ***데이터베이스에 저장하는 등의 처리***를 한다. 또한 장고 Form을 통해 ***HTML Form을 생성***할 수도 있고, 이를 활용해서 인터페이스만 맞추어 직접 HTML Form을 구현해도 괜찮다.

### HTML Form

- ***<form></form> 태그***를 통해서 입력폼을 구성하고, submit 시에는 지정 ***action URL로 데이터 전송***을 시도한다.
- 하나의 <form> 태그는 하나 이상의 widget을 가진다.
- <form> 태그의 필수 속성으로는 action, method, enctype이 있다.
    1. **action** : ***요청을 보낼 주소***
    2. **method** : ***전송 방식***. ***"GET"은 주로 데이터 조회 요청*** 시에 사용된다. ***"POST"는 파괴적인 액션(생성/수정/삭제)***에서 사용된다.
    3. **enctype** : ***인코딩 방식. POST 요청에서만 유효***하며,  GET 요청에서는 "application/x-www-form-urlencoded" type으로 고정된다.

### <form> 태그의 enctype 속성

1. ***"application/x-www-form-urlencoded"*** : ***enctype의 디폴트 값.*** 인자들을 "URL 인코딩"을 수행해서 ***QueryString 형태로 전달***한다. file의 내용이 아니라 file명만 전달되기 때문에 ***파일 업로드가 불가능***하다.
2. ***"multipart/form-data" : 파일 업로드가 가능***하다.
3. ~~"text/plain" : 실제로 사용되지는 않음. 공백만 '+'로 변환한다.~~

### url encode

- ***key=value 값의 쌍이 & 문자로 이어진 형태***
- 공백은 '+'로 인코딩하며 special 문자들은 ASCII 16진수 문자열, UTF8 인코딩 16진수 문자열로 반환한다.

```html
key1=value1&key2=10&name=%EB%B0%A9%ED%83%84%EC%86%8C%EB%85%84%EB%8B%A8
```

### Form 요청에서 인자를 보내는 2가지 방법

1. ***요청 URL 뒤에 ?*** 를 붙이고 인자를 실어서 보내기
    - "***x-www-form-urlencoded"*** 인코딩의 값만 실을 수 있다. (GET 요청의 인코딩 방식은 강제되기 때문)
2. ***요청 Body***에 모든 인코딩의 인자를 실어서 보낸다.
    - ***x-www-form-urlencoded*** 인코딩의 값도 괜찮고, ***multipart/form-data 인코딩의 값(파일 업로드)*** 또한 괜찮다.

### 2가지 Form Method

1. **GET 방식**
    - 엽서에 비유. 물건을 보낼 수 있다.
    - **application/x-www-form-urlencoded ****만이 강제되며, **인코딩된 문자열을 QueryString**으로 전달. ******
    - GET 요청에서는 Header 부분만 존재하기 때문에 파일을 담을 수 없다.

    ```html
    <form method="GET" action="">
    	<input type="text" name="query" />
    	<input type="submit" value="검색" />
    </form>
    ```

2. **POST 방식**
    - 택배에 비유. 다양한 물건을 보낼 수 있다.
    - 다양한 인코딩을 모두 사용할 수 있으며, **인코딩된 데이터를 "요청 Body"**에 담아서 전달.

    ```html
    <form method="POST" action="" enctype="multipart/form-data">
    	<input type="text" name="title">
    	<textarea name="content"></textarea>
    	<input type="file" name="photo" />
    	<input type="submit" value="저장" />
    </form>

    <!--> 장고 Form에서는 GET요청과 POST요청을 같은 URL에서 받기 때문에, action을 비워놔도 무방하다. <-->
    ```

    ### 장고 뷰에서의 인자 접근

    - ***request.GET*** : **모든 QueryString 인자 목록**. GET/POST 요청에서 모두 가능 ***(파일에 대해서 접근 불가)***
    - [***request.](http://request.POST)POST*** : POST 요청에서만 가능. **파일 내역은 제외한 모든 POST 인자 목록.** (요청 Body를 파싱한 QueryDict 객체)***(파일에 대해서 접근 불가)***
    - ***request.FILES*** : POST 요청에서만 가능. **요청 Body에서 파일 내역만 파싱한 MultiValueDict 객체.** (***파일에 대해서 접근 가능)***

    ---

    ### HttpRequest 객체

    - 클라이언트로부터의 ***모든 요청 내용***을 담고 있다.
    - 요청 패킷에서 header 영역과 body 영역 전체를 파싱한다.
        1. ***함수 기반 뷰*** : 매 요청 시마다 ***뷰 함수의 첫번째 인자 request***로 전달
        2. ***클래스 기반 뷰*** : 매 요청 시마다 ***self.request***를 통해 접근
    - *Form 처리 관련 속성들*
        1. .**method** : 요청의 종류("GET", "POST")로서 모두 대문자
        2. .**GET** : GET 인자 목록 (QueryDict 타입)
        3. .**POST** : POST 인자 목록 (QueryDict 타입)
        4. .**FILES** : POST 인자 중에서 파일 목록 (MultiValueDict 타입)
        - ***MultiValueDict와 QueryDict***
            - ***MultiValueDict***는 **동일 Key의 다수 Value를 지원**하는 사전.

            ```python
            from django.utils.datastructures import MultiValueDict

            d = MultiValueDict({'name' : ['Adrian', 'Simon'], 'position' : [Developer]})

            d['name'] # 가장 나중 값인 'Simon'을 출력한다.
            d.getlist('name') # ['Adrian', 'Simon']
            d.getlist('doesnotexist') # 없는 Key에 접근하면 빈 리스트를 반환

            d['name'] = 'changed'
            d # <MultiValueDict : {'name' : ['changed'], 'position' : ['Developer']}>
            ```

            - ***QueryDict***는 **수정불가능한 MultiValueDict**이다.

            ```python
            from django.http import QueryDict

            qd = QueryDict('name=Adrian&name=Simon&position=Developer', encoding='utf8')

            qd['name'] # 'Simon'
            qd.getlist('name') # ['Adrian', 'Simon']

            qd['name'] = 'changed'
            # AttributeError: This **QueryDict instance is immutable** 발생!!
            ```

    ### HttpResponse 객체

    - django.http.HttpResponse
    - ***다양한 응답을 Wrapping*** 한다. (HTML문자열, 이미지 등등)
    - ***View에서는 반환값으로서 HttpResponse 객체***를 기대한다.
    - JsonResponse, StreamingHttpResponse, FileResponse 등도 있다.

    - 파일-like 객체와 같이 ***.write(content), .flush(), .tell() 함수를 지원***한다.

    ```python
    response = HttpResponse(
    	"<p>Here's the text of the Web page.</p>"
    	"<p>Here's another paragraph.</p>"
    )

    response = HttpResponse()
    response.write("<p>Here's the text of the Web page.</p>")
    response.wrtie("<p>Here's another paragraph.</p>")
    ```

    - 사전-like 인터페이스로 ***응답의 커스텀 헤더 추가/삭제*** 가능

    ```python
    response = HttpResponse()
    response['Age'] = 120
    del response['Age']
    ```

    - ***파일 첨부로 처리되기를 브라우저에게 알리기***

    ```python
    response = HttpResponse(excel_data, content_type='application/vnd.ms-excel')
    response['Content-Disposition'] = 'attachment; filename="foo.xls"'
    ```

---

### Form

**장고를 더욱 장고스럽게** 만들어주는 주옥같은 feature

- 입력 Form(HTML)과의 interaction
    1. ***입력폼 HTML을 생성***
    2. 입력폼 값에 대한 ***유효성 검증(validation) 및 값 변환***
    3. 검증을 ***통과한 값들을 dict 형태***로 제공*(cleaned_data)*

```python
#myapp/forms.py
from django import forms

class PostForm(forms.Form):
	title = forms.CharField()
	content = forms.CharField(widget=form.Textarea) #widget이라는 키워드를 통해서 입력 widget을 custom 변경.
```

### Django 스타일의 Form 처리

- ***하나의 URL (하나의 View)***에서 ***2가지 역할을 모두 수행***한다.
    1. ***빈 폼을 보여주는 역할***
    2. 폼을 통해 ***입력된 값을 검증***하고 ***저장***하는 역할

```python
@login_required # request.user을 외래키 할당하려면 필히 로그인 상황임을 보장 받아야 함.
def post_new(request):
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES) # 데이터를 입력 받아서 유효성 검증 수행
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            return redirect(post)
    else:
        form = PostForm()
		# ***POST에 대해서 먼저 조건체크***를 하는 게 장고 스타일이다.

    return render(request, 'instagram/post_form.html', {
        'form' : form,
    })
```

### Form/ModelForm 클래스 정의

- ***클래스 정의***
    1. ***Form의 경우***

        ```python
        #myapp/forms.py
        from django import forms

        class PostForm(*forms.Form*):
        	title = forms.CharField()
        	content = forms.CharField(widget=form.Textarea)
        ```

    2. ***ModelForm의 경우***

        ```python
        #myapp/forms.py
        from django import forms
        from .models import Post

        class PostForm(*forms.ModelForm*):
        	class Meta:
        		model = Post
        		fields = '__all__'
        # 모델과 필드를 지정하면 모델폼이 자동으로 폼 필드를 생성
        ```

- ***필드 별 유효성 검사 함수 추가 적용***
    1. ***Form의 경우***

        ```python
        #myapp/forms.py
        from django import forms

        def min_length_3_validator(value):
        	if len(value) < 3:
        		raise forms.ValidationError('3글자 이상 입력해주세요.')

        class PostForm(*forms.Form*):
        	title = forms.CharField(validators=[min_length_3_validator])
        	content = forms.CharField(widget=form.Textarea)
        ```

    2. ***ModelForm의 경우***

        ```python
        #myapp/models.py
        from django import forms
        from django import models

        def min_length_3_validator(value):
        	if len(value) < 3:
        		raise forms.ValidationError('3글자 이상 입력해주세요.')

        class Post(models*.Model*):
        	title = models.CharField(max_length=100, validators=[min_length_3_validator])
        	content = models.TextField()
        # 최대한 model을 풍성하게 만드는 게 좋다.
        # 따라서 validators는 form 보다는 model에 적용하는 걸 추천
        # 해당 model을 통해 ModelForm을 구성할 때 가져가서 사용할 수 있다.

        #myapp/forms.py
        from django import forms
        from .models import Post

        class PostForm(*forms.Form*):
        	class Meta:
        		model = Post
        		fields = '__all__'
        ```

- ***POST 요청에 한해 입력값 유효성 검증***

    ```python
    if request.method == 'POST':
    	# POST인자는 request.POST와 request.FILES를 제공 받는다.
    	form = PostForm(request.POST, request.FILES)

    	# 인자로 받은 값들에 대해서 유효성 검증 수행
    	if form.is_valid()
    		post = Post(**form.cleaned_data) # 검증에 성공한 값들을 사전타입으로 제공 받음.(cleaned_data)
    		post.save()
    	else:
    		form.errors

    else: # GET 요청일 때
    	form = PostForm()
    	return render(request, 'myapp/form.html', {'form':form})

    ```

- ***템플릿을 통해 HTML 폼 노출***
    1. *GET 요청일 때*
    2. *POST 요청이지만 유효성 검증에서 실패했을 때*

    ```python
    #post_form.html
    <form action="" method="post" enctype="multipart/form-data">
        {% csrf_token %}
        <table>
            {{ form.as_table }}
        </table>
        <input type="submit" value="저장" />
    </form>

    # form 인스턴스를 HTML 폼을 출력한다.
    # 이때 오류메세지가 있다면 함께 출력한다.
    ```

### Form Fields

- Model Fields와 유사
    1. ***Model Fields*** : **DB Field** 들을 *파이썬 클래스화*
    2. ***Form Fields*** : **HTML Form Field** 들을 *파이썬 클래스화*

---

### Cross Site Request Forgery(CSRF)

- *사용자가 의도하지 않게* 게시판에 글을 작성하거나, 쇼핑을 하게 하는 등의 공격

### CSRF Token

- 서버 입장에서 공격을 막기 위해 ***Token을 통한 체크***가 필요하다.
- ***POST 요청에 한해*** CsrfViewMiddleware를 통한 체크를 한다. 만약 POST 요청을 받을 때 ***Token 값이 없거나 유효하지 않은 경우*, *403 Forbidden 응답***이 발생한다.
    1. ***입력폼을 보여줄 때 CSRF Token 값도 같이 할당***된다. (CSRF Token은 유저마다 다르며 계속해서 변경)
    2. 해당 입력폼을 통해 Token 값이 전달되면, ***Token에 대한 유효성 검증***

```html
<form action="" method="POST">
	{% csrf_token %}
	<input type="text" name="title">
	<textarea name="content"></textarea>
	<input type="submit">
</form>

<input type="hidden" name="csrfmiddlewaretoken" value="...."/>
```

### 주의사항

- CSRF Token ≠ 유저인증 Token
- CSRF Token ≠ JWT (JSON Web Token)
- *CSRF Token 체크 기능은 가급적이면 끄지 않는 게 좋다.*
- 만약 특정 View에 한해 ***CSRF Token 체크에서 배제***하려면, ***해당 View에 @csrf_exempt 장식자***를 적용하면 된다.

    ```python
    form django.views.decorators.csrf import csrf_exempt

    @csrf_exempt
    def post_new_for_api(request):
    	#..
    	#..
    ```

---

### ModelForm

- ***장고 Form을 상속*** 받는다.
- 지정된 Model로부터 필드 정보를 읽어 들여서, ***Form Fields를 알아서 설정***한다.
- 내부적으로 Model Instance를 유지한다.
- 유효성 검증에 통과한 값들로 ***지정 Model Instance로의 저장(create/update)을 지원***한다.(유효성 검사는 *forms.py에 지정된 필드에 한해서* 이루어진다)

```python
class PostForm(forms.ModelForm):
	class Meta:
		model = Post
		fields = '__all__'
		# 전체 필드를 지정.
		# 혹은 읽어올 필드명을 list로 지정해도 괜찮다.
```

### ModelForm.save(commit=True)

- Form의 *cleaned_data를 Model Instance 생성에 사용*하고, 그 Instance를 리턴.
    1. *commit=True 경우*
        - ***model instance의 save()*** 및 ***form.save_m2m()***을 호출한다.
        - ***이때 form.save()와 instance.save()는 같지 않다.***
    2. *commit=False 경우*
        - ***instance.save() 함수 호출을 지연***하고자할 때 사용한다.

### View에서의 ModelForm 처리

1. New

    ```python
    @login_required
    def post_new(request):
    	if request.method == 'POST':
    	  form = PostForm(request.POST, request.FILES)
        if form.is_valid():

    	    ***post = form.save(commit=False)**
    			# 필수 필드인 author의 값을 user로부터 입력 받지 않고
    			# 프로그램으로부터 채워 넣을 것이기 때문에 commit=False로 설정		*****
          post.author = request.user
          ***post.save()**
    			# author의 값을 채워 넣고 instance.save() 호출*

          return redirect(post)
      else:
          form = PostForm()

    	return render(request, 'instagram/post_form.html', {
    		'form' : form,
    	})
    ```

2. Edit

    ```python
    @login_required
    def post_edit(request, ***pk***):
    	***post = get_object_or_404(Post, pk=pk)***

    	# 작성자 Check Tip!
    	if post.author != request.user:
    		messages.error(request, '작성자만 수정할 수 있습니다.')
    		return redirect(post)

    	if request.method == 'POST':
    		form = PostForm(request.POST, request.FILES, ***instance=post***)
    		if form.is_valid():
    		  post = form.save()
    		  return redirect(post)
    	else:
    		form = PostForm(***instance=post***)

    	return render(request, 'instagram/post_form.html', {
    		'form' : form,
    	})
    ```

### 주의사항

```python
form = CommentForm(request.POST)
if form.is_valid():
	~~message = request.POST['message']~~
	# 폼 인스턴스 내에서 clean 함수를 통해 데이터가 변환되었을 수도 있기 때문에 cleaned_data에서 가져온다.

	**message = form.cleaned_data['message']**
	comment = Comment(message=message)
	comment.save()
return redirect(post)
```

---