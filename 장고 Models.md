# 장고 Models

생성일: 2020년 9월 2일 오후 7:49

### SQL(Structured Query Language)

- DB에 쿼리하기 위한 언어.
- 같은 작업을 하더라도, 보다 적은 수의 SQL, 보다 높은 성능의 SQL을 사용하면 더 좋은 소프트웨어를 만들 수 있다.
- 장고 모델은 관계형 데이터베이스(RDBMS)만을 지원한다.
- 직접 SQL을 만들어 내기보다는, ORM(Object-relational mapping)을 통해 SQL을 생성/실행하는 걸 추천한다.
- **(!!)***하지만 ORM을 사용하는 경우에도, 내가 작성한 ORM코드를 통해 어떠한 SQL이 실행되고 있는지를 파악해야 한다.(django-debug-toolbar을 적극 활용)*

### Django Model

- 장고 모델은 <DB 테이블>과 <파이썬 클래스를> **1:1**로 mapping
- 매핑되는 모델 클래스는, DB 테이블 필드 내역과 일치해야 한다.
- *모델을 만들기 전에*, 서비스에 맞게 *DB 설계*가 필수.
- 파이썬 클래스가 먼저 만들어진 경우라면 migration을 통해 그에 따라 DB 테이블 생성 가능 (반대의 경우에는 DB 테이블에 따라 파이썬 클래스를 생성하면 된다)
- DB 테이블명의 default는 ex) blog_post(blog앱의 Post 모델인 경우) → 모델 Meta 클래스의 db_table 속성을 통해 변경할 수 있다.

### Django Model 활용

- 장고 모델을 통해 데이터베이스 형상을 관리하는 경우
    1. models.py에 모델 클래스 작성
    2. shell에서 makemigrations, migrate 명령 수행
    3. 모델 활용

```python
$ python manage.py makemigrations <app-name> : 마이그레이션 파일 생성
$ python manage.py migrate <app-name> : 마이그레이션 파일을 DB에 적용
$ python manage.py showmigrations <app-name> : 해당 앱의 마이그레이션 적용 현황
$ python manage.py sqlmigrate <app-name> <migration-name> : 지정된 마이그레이션의 SQL 코드 출력
```

- 장고 외부에서 이미 설계된 데이터베이스 형상을 관리하는 경우

    해당 데이터베이스로 모델 클래스 소스를 생성 후, 활용 → inspectdb 명령

---

### 모델 필드

- AutoField, CharField, TextField, SlugField, DateTimeField, BooleanField, FileField, ImageField 등
- ForiegnKey, ManyToManyField, OneToOneField 등
- 모델 필드들은 DB 필드 타입을 반영. ex) AutoField→int, BinaryField→bytes, BooleanField→bool
- 같은 모델 필드라도, DB 엔진이 바뀌면 다른 타입이 될 수도 있으니 SQL을 확인.

### 주요 필드 공통 옵션

- ***null (DB 옵션)*** : null 허용 여부 (default : False)
- ***unique (DB옵션)*** : 현재 테이블 내에서 유일성 여부 (default : False)
- ***db_index (DB 옵션)*** :  인덱스 필드 여부 (default : False)
- ***blank*** : 입력값 유효성 검사 시에 empty 값 허용 여부 (default : False)
- ***choices*** : select box 소스로 사용
- ***validators*** : 입력값 유효성 검사를 수행할 함수를 다수 지정
    1. 각 모델 필드마다 고유한 validators들이 등록되어 있기도 함.
    2. ex) 이메일만 받기, 최대 길이 제한, 최소 길이 제한, 최대값 제한, 최소값 제한 등
- ***verbose_name*** : 필드 레이블. 미지정시 필드명이 사용
- ***help_text*** : 필드 입력 도움말

입력값 오류를 막기 위해, 최대한 필드 타입을 타이트하게 지정해야 한다.

1. blank, null 지정을 최소화 한다.
2. validators들을 다양하고 타이트하게 지정한다.
3. 클라이언트로부터 넘어온 데이터를 신뢰할 수 없기 때문에 ***Back-end에서의 유효성 검사***는 필수이다.

***Model 설계가 Django 구현의 절반이다!***

---

### Django Admin

- 장고가 제공하는 기본 앱*(django.contrib.admin)*
- 모델 클래스 등록을 통해, ***조회/추가/수정/삭제 웹 UI***을 admin에서 제공
- 내부적으로 Django Form을 적극적으로 사용한다. ex) 각 모델의 필드에 맞는 입력폼이 admin 사이트에 제공

### 모델 클래스를 admin에 등록

```python
from django.contrib import admin
from .models import Post

# **방법1: 기본 ModelAdmin으로 등록**
admin.site.register(Post)

# **방법2: admin.ModelAdmin을 상속 (커스터마이징 가능)**
class PostAdmin(admin.ModelAdmin):
	list_display = ['id', 'message', 'created_at', 'updated_at']
admin.site(Post, PostAdmin)

# **방법3: 파이썬의 장식자(decorator) 형태로 등록 (커스터마이징 가능)**
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
	list_display = ['id', 'message', 'created_at', 'updated_at']
	list_display_links = ['message']
```

### 모델 클래스에 __str__ 구현

- admin 사이트의 모델 리스트에서 <모델명 object>로 나타나는 문자열을 마음대로 변경하기 위해 사용.
- 자바의 toString()과 유사.

```python
def __str__(self):
return f"Custom Post Object (({self.id})"

# Custom Post Object 1
# Custom Post Object 2
# ...
```

### ModelAdmin 옵션

- list_display, list_display_links, search_fields, list_filter 등

```python
# admin.py
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
	list_display = ['id', 'message', 'created_at', 'updated_at', 'is_public']
# admin 사이트 내의 모델 리스트에 출력할 컬럼 지정

	list_display_links = ['message']
# 지정된 필드 중 detail 링크를 걸 필드 리스트

	list_filter = ['is public']
# 지정된 필드 값으로 필터링 옵션 제공

	search_fields = ['name']
# admin 사이트 내에서 검색 기준이 될 필드 리스트
```

```python
# **list_display** 에는 필드명 뿐 아니라 **모델의 member function**도 가능. **(함수의 인자가 없는 경우)**

# *models.py*
def message_length(self):
	return len(self.message)

# *admin.py*
list_display = ['id', 'message', 'message_length', 'created_at', 'updated_at']
```

---

### Static and Media Files

- ***static file*** : 개발 리소스로서의 정적인 파일. **개발자가 직접 저장** (js, css, image 등)
- ***media file*** : **유저가** FileField / ImageField를 통해 저장한 모든 파일.

### Media File 처리 순서

1. HttpRequest.FILES를 통해 파일이 전달.
2. 유효성 검증을 수행. (뷰 로직이나 폼 로직을 이용)
3. 해당 DB 필드(FileField / ImageField)에 **(!!)*file이 저장된 경로(문자열)***를 저장.
4. **setting.MEDIA_ROOT 경로**에 ***실제 file***을 저장.

```python
# settings.py

MEDIA_URL = '/media/'
# file에 접근할 때 사용
# 각 media file에 대한 URL prefix
# 필드명.url 속성에 의해서 참조되는 설정

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
# file을 저장할 때 사용
# MEDIA_ROOT 설정이 되어 있지 않다면, media file이 root에 바로 저장되어서 뒤죽박죽이 된다.
```

### FileField와 ImageField

- blank=True 옵션 적용.
- ImageField을 두고자 할 경우, Pillow를 먼저 설치해줘야 한다.
- 한 디렉토리(ex) setting.MEDIA_ROOT)에 너무 많은 파일이 쌓이지 않도록 ***upload_to 옵션***을 통해 저장 경로를 계산.

```python
photo = models.ImageField(blank=True, upload_to='instagram/post/%Y/%M/%D')
# 사진 파일은 'instagram/post/저장날짜(연)/저장날짜(월)/저장날짜(일) 폴더에 저장된다.

# 인자 유형에는 1) 문자열로 지정 2) 함수로 지정 이 있다.
# 1)은 file을 저장할 경로를 커스터마이징하고, 2)는 file명까지도 커스터마이징이 가능하다.

def uuid_name_upload_to(instance, filename):
	app_label = instance.__class__._meta.app_label // 앱 별로
	cls_name = instance.__class__.__name__.lower() // 모델 별로
	ymd_path = timezone.now().strftime('%Y/%M/%D')
	uuid_name = uuid4.hex()
	extension = os.path.splitext(filename)[-1].lower() // 확장자 추출
	return '/'.join([
		app_label, cls_name, ymd_path, uuid_name[:2], uuid_name + extension,])
```

### 개발환경에서의 Media File 서빙

- static 파일과는 다르게 개발서버에서 서빙 미지원.
- 개발 편의성 목적으로 직접 서빙 rule 추가 가능.

```python
from django.conf import settings
from django.conf.urls.static import static

if settings.DEBUG:
	urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
# False일 경우에는 빈 리스트
```

### 템플릿에서 media URL 처리

- .url 속성은 내부적으로 settings.MEDIA_URL과 조합을 처리.
- 따라서 필드에 저장된 값(경로)이 없을 경우 계산에 실패. 그러니까 안전하게 필드명 저장 유무를 체크.
- 파일 시스템 상의 절대경로가 필요하다면 .path 속성을 활용 (setting.MEDIA_ROOT와 조합)

```python
{% if post.photo %}
	<img src="{{ post.photo.url }}" />
{% endif %}
```

---

### Model Manager

- *데이터베이스 질의 인터페이스*를 제공.
- 디폴트 Manager로서 ModelCls.objects가 제공.

```python
ModelCls.objects.all() # 특정 모델의 전체 데이터 조회
ModelCls.objects.all().order_by('-id')[:10] # id 필드로 정렬된 특정 모델의 최근 10개 데이터 조회
Modelcls.objects.create(title="New Title") # 특정 모델의 새로운 Row 생성
```

### QuerySet

- *SQL을 생성해주는 인터페이스.*
- **순회** 가능한 객체.
- Model Manager를 통해, 해당 모델에 대한 queryset 획득.

```python
ex) Post.objects.all() → "SELECT * FROM POST ..."

ex) Post.objects.create() → "INSERT INTO Post VALUES ..."
```

- Chaining 지원.
- queryset은 Lazy한 특성을 가진다. 즉, 데이터가 정말 필요할 때까지 장고가 SQL을 호출하지 않는다.

### DB 데이터 조회

- filter(AND, OR) ↔ exclude(AND, OR)

```python
from django.db.models import Q

qs1 = Post.objects.all() # 데이터를 획득할 준비
qs1 = qs.filter(조건필드1=조건값1, 조건필드2=조건값2)
qs1 = qs.filter(조건필드3=조건값3) # 최종 qs에는 조건1~3이 적용된다.
# chaining을 통해 qs = qs.filter(조건1, 조건2, 조건)의 결과 queryset와 같다.

qs2 = Post.objects.all()
qs2 = qs.filter(Q(조건필드1=조건값) | Q(조건필드2=조건값2)) # 조건1 또는 조건2가 적용
```

**특정 모델 객체 1개**를 얻으려고 하는 경우, ***qs[index], qs.get(...), qs.first(), qs.last()*** 로 획득 가능하다.

**OR 조건을 사용**하기 위해선, 개별 조건들을 ***Q 객체***로 감싸줘야 한다.

### 필드 타입별 다양한 조건 매칭

- **숫자/날짜/시간 필드** : __lt, __lte, __gt, __gte
- **문자열 필드** : *__startswith, __istartswith(대소문자 문시), __endswith, __iendswith(대소문자 무시), __contains, __icontains(대소문자 무시)*

### filter을 이용한 간단한 검색 구현

```python
def post_list(request):
    qs = Post.objects.all() # 전체를 가져올 준비
    q = request.GET.get('q', '') # q라는 이름의 인자가 있으면 가져오고(request에서 q라는 이름으로 데이터를 날리기 때문), 없으면 None
    if q:
        qs = qs.filter(message__icontains=q)
        # instagram/templates/instagram/post_list.html
    return render(request, 'instagram/post_list.html', {
        'post_list' : qs,
        'q' : q,
    })
```

### QuerySet에 정렬 조건 추가

- 정렬 조건을 추가하지 않으면 일관된 순서를 보장 받을 수 없음.
- DB에서 다수 필드에 대한 정렬을 지원 (하지만 가급적 단일 필드로 하는 게 성능에 이익)
    1. 모델 클래스의 Meta 속성으로 ordering 설정. default로 지정된다.
    2. queryset 코드에서 직접 order_by(...)로 설정. default 정렬 조건을 무시할 수 있다.

```python
#1
class Meta:
        ordering = ['-id', 'name'] # 1차 기준, 2차 기준
#2
queryset = queryset.order_by('-id', 'name')
```

### QuerySet에 범위 조건 추가

- 객체[start:stop:step]로 지정 : OFFSET→start, LIMIT→stop
- step을 사용하면, queryset이 아니라 list을 반환하기 때문에 사용을 추천하지 않는다.

```python
queryset = queryset[:10] # 현재 queryset에서 처음10개만 가져오는 조건을 추가한 queryset
queryset = queryset[10:20] # 현재 queryset에서 처음10번째부터 20번째까지를 가져오는 조건을 추가한 queryset

# 리스트 슬라이싱과 거의 유사하나, 역순 슬라이싱은 지원하지 않음
queryset = queryset[-10:] # AssertionError 예외 발생

# 이때는 먼저 특정 필드 기준으로 내림차순 정렬을 먼저 수행한 뒤, 슬라이싱
queryset = queryset.order_by('-id')[:10]

```

---

### django-debug-toolbar을 통한 SQL 디버깅

- 현재 request/response에 대한 다양한 디버깅 정보를 보여준다. (다만, 응답이 HTML 포맷일 때만 가능)
- SQL 패널을 통해, 각 요청 처리 시에 발생한 SQL 내역 확인 가능.
- Ajax 요청에 대한 지원은 불가.
- 웹페이지의 template에 반드시 <body> 태그가 있어야만 정상적으로 동작한다.

```python
pip install django-debug-toolbar
```

```python
# mysite/settings.py

INSTALLED_APPS = [..., "debug_toolbar"]
MIDDLEWARE = ["debug_toolbar.middleware.DebugToolbarMiddleware", ...]
INTERNAL_IPS = ["127.0.0.1"]

# mysite/urls.py
from django.conf import settings
from django.conf.urls import include, url
  # 중략 ...

if settings.DEBUG: # setting.py의 DEBUG = True인 경우
    import debug_toolbar
    urlpatterns += [
        url(r'^__debug__/', include(debug_toolbar.urls)),
    ]
```

### 코드를 통한 SQL 내역 확인

- settings.DEBUG = True 시에만 쿼리 실행내역을 메모리에 누적되고, 프로세스가 재시작되면 초기화.
- 실사용 시에는 서버가 재시작하지 않기 때문에 메모리가 부족할 수 있다. 따라서 DEBUG 옵션을 False로 설정.

---

(!!) *ORM은 어디까지나, SQL 생성을 도와주는 라이브러리. DB에 대한 모든 걸 해결해주지 않기 때문에, 사용할 DB 엔진과 SQL에 대한 높은 이해가 필요하다.*

### RDBMS에서의 관계 예시

- **1 :1 관계** → models.OneToOneField로 표현.
- **1 : N 관계** → models.ForeignKey로 표현.
- **M : N 관계** → models.ManyToManyKey로 표현.
- RDBMS에서의 *관계는 설계하기 나름*이다.

### Foreign Key

- 1 : N 관계에서, **N측에 명시.**
- ***models.ForeignKey(to, on_delete)***
    1. to : ***대상 모델.*** 클래스를 직접 지정하거나, 클래스명을 문자열로 지정. 자기 참조는 'self' 지정.
    2. on_delete : **레코드 삭제 시 규칙** (ex) CASCADE : 외래키로 참조하는 다른 모델의 레코드도 삭제)

```python
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    # 실제로 DB 필드에는 post_id 필드가 생성된다. (id는 Post의 PK)
```

### 올바른 User 모델 설정

```python
# (settings.py)
AUTO_USER_MODEL = 'auth.User' # 현재 활성화된 User 모델. User 모델을 변경할 경우, 그에 맞춰서 바꿔줘야 한다.

# (models.py)
class Post(models.Model):
	author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
-----------------------------------------------------------------------------------------------
# (settings.py)
# from django.contrib.auth.models import User 

# (models.py)
# class Post(models.Model):
# 	author = models.Foreignkey(User, on_delete=models.CASCADE)

# 아래와 같은 방법은 추천하지 않는다.
```

### ForeignKey에서 reverse_name

- 1 : N 관계에서 1측에서 사용한다. (1측에는 참조할 이름이 없기 때문)
- 디폴트 속성명은 ***"모델명(소문자)_set"***
- 다만, reverse_name(default)은 모델명만 고려하기 때문에 서로 다른 앱에 같은 이름의 모델이 존재한다면 충돌이 난다. 그럴 경우 reverse_name을 지정해주거나 변경을 하면 된다.
- .limit_choices_to 옵션을 통해, admin page에서의 선택항목을 제한할 수 있다.

```python
from django.db import models

class Post(models.Model):
	title = models.CharField(max_length=100)
    content = models.TextField()

class Comment(models.Model):
	post = models.ForeignKey(Post, on_delete=models.CASCADE,
		limit_choices_to={'is_public' : True})
    message = models.TextField()

# post.comment_set.all()로 사용한다.
# admin page에서 Comment 모델을 생성할 때, is_public 옵션이 True인 Post 모델만 선택 가능하다.
```

```python
# blog앱의 Post모델
author = ForeignKey(User)
# shop앱의 Post모델
author = ForeignKey(User)

# 이름 충돌이 발생하기 때문에, makemigrations 명령이 실패.
```

### OneToOneField

- 1 : 1 관계에서 어느 쪽이라도 가능
- ForeignKey(unique=True)와 유사하지만, reverse 차이
    - User:Profile을 FK로 지정한다면 → profile.***user_set.first()*** → user, 값이 없다면 ***None 발생.***
    - User:Profile을  O2O로 지정한다면 → profile.***user*** → user, 예외 처리로 ***DoNotExists 발생.***
- related_name의 디폴트는 *모델명(소문자)*로 설정된다!

### ManyToManyField

- M : N 관계에서 어느 쪽이라도 필드 지정 가능.
- ***models.ManyToManyField(to, blank=False)***
- 포스팅을 할 때, 태그를 안 넣을 수 있는 것처럼 blank=True로 설정해줘야 하는 상황도 발생한다.

```python
class Post(models.Model):
	tag_set = models.ManyToManyField('tag', blank=True) # Tag 모델은 아래에 있기 때문에, 문자열로 명시
# 양쪽에서 필드 지정이 가능하지만, 해당 필드를 활용하는 쪽에서 지정하는 게 바람직하다.

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    #post_set = models.ManyToManyField(Post)
```

---

### Migration

- 모델의 변경 내역을 *데이터베이스 스키마*로 반영시키는 효율적인 방법을 제공.
- 장고를 통해서 직접 DB 스키마 히스토리를 관리.

```python
$ python manage.py makemigrations <app-name> : 마이그레이션 파일 생성(입력값 유효성 검사는 하지 않는다)
$ python manage.py migrate <app-name> : 마이그레이션 파일을 DB에 적용
$ python manage.py showmigrations <app-name> : 해당 앱의 마이그레이션 적용 현황
$ python manage.py sqlmigrate <app-name> <migration-name> : 지정된 마이그레이션의 SQL 코드 출력

# 장고 프로젝트를 처음 생성하면, **기본 앱들에 대한 migration**을 위해 python manage.py migrate 실행!
# makemigrations, migrate 명령을 수행할 때 **app-name을 지정**해주는 것이 좋다.(예상치 못한 migrations을 피할 수 있고, 빠르고 명확하다.)
# migrate 명령 이전에 sqlmigrate 명령을 통해, **적용될 SQL 쿼리를 확인하는 습관**이 필요하다.
```

### Migration 파일

- *데이터베이스에 어떤 변화*를 가하는 명령들을 List 형태로 나열.
- ***makemigrations 명령***을 통해서 대개 ***모델로부터 자동 생성***된다.
- 같은 migration 파일이라도 DB 종류에 따라 다른 SQL이 생성된다.

![Untitled](https://user-images.githubusercontent.com/67837091/92785913-c581d480-f3e2-11ea-98e9-70bfb2246a16.png)

- 모델 필드와 관련된 어떠한 변경이라도 발생 시에 migration 파일이 생성된다. 예를 들어 모델의 member function 구현 시에 **실제 DB 스키마에 가해지는 변화가 없더라도 수행**한다.
- migration 파일은 ***모델의 변경 내역을 누적***하는 역할을 한다.
- **적용된 migration 파일은 절대로 삭제하면 안된다.** 만약 파일의 수가 너무 많다면 squashmigrations 명령을 통해 여러 migration 파일을 통합할 수 있다.(rollback으로 인한 미적용 파일은 삭제해도 괜찮다)

### 마이그레이션 migrate (Forward-Backward)

```python
$ python manage.py migrate <app-name>
# 미적용 migration 파일부터 최근 migration 파일까지 순차적으로 수행한다.**(Forward)**

$ python manage.py migrate <app-name> <migration-name>
# 지정된 migration 파일이 현재 적용된 migration 보다
# 이후라면, 정방향으로 지정 migration까지 **Forward** 수행.
# 이전이라면, 역방향으로 지정 migration까지 **Backward** 수행.
```

### Migration 이름 지정

- 전체 파일명을 지정하지 않더라도, 판독이 가능하다면 파일명 일부로도 지정이 가능하다.

```python
# 파일명 예시
blog/migrations/0001_initial.py
blog/migrations/0002_create_field.py
blog/migrations/0002_update_field.py

python manage.py migrate blog 0001 # OK
python manage.py migrate blog 0002 # FAIL (다수 파일 해당)
python manage.py migrate blog zero # 해당 앱의 모든 migration을 rollback
```

### Migration 순서

- 마이그레이션 순서는 **각** **migration 파일의 dependency**에 따라 정의

```python
# migration file
from django.db import migrations, models

class Migration(migrations.Migration):
	dependencies = [
		('shop', '0001_initial'), # 다음 migration 순서를 지정
	]
	operation = [
		...
	]
```

### id 필드

- 모든 DB 테이블에는 각 Row의 식별 기준인 기본키(Primary Key)가 필요
- 장고에서는 기본키로 id(AutoField)가 디폴트 생성
- 다른 필드를 기본키로 사용하고 싶다면 primary_key = True 옵션을 적용(가급적 id 필드 사용 권장)

### 기존 모델 클래스에 필수 필드 추가

- 필드 옵션 blank, null의 디폴트는 False이다. 따라서 기본적으로 모든 필드는 필수 필드이다.
- 만약 새로운 필수 필드를 추가해서 makemigrations를 수행하면, ***기존에 있는 레코드에 대해서 어떤 값을 채워 넣을지*** 설정해야한다.(새로 추가되는 필수 필드가 3개라면, 해당 질문이 3번 돈다)

    *선택1. 지금 값을 입력*

    *선택2. 명령 수행을 중단*

---
