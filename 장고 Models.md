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

```jsx
$ python manage.py makemigrations <app-name> : 마이그레이션 파일 생성
$ python manage.py migrate <app-name> : 마이그레이션 파일을 DB에 적용
$ python manage.py showmigrations <app-name> : 해당 앱의 마이그레이션 적용 현황
$ python manage.py sqlmigrate <app-name> <migration-name> : 지정된 마이그레이션의 SQL 코
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

**Model 설계가 Django 구현의 절반이다!**

---

### Django Admin

- 장고가 제공하는 기본 앱*(django.contrib.admin)*
- 모델 클래스 등록을 통해, ***조회/추가/수정/삭제 웹 UI***을 admin에서 제공
- 내부적으로 Django Form을 적극적으로 사용한다. ex) 각 모델의 필드에 맞는 입력폼이 admin 사이트에 제공

### 모델 클래스를 admin에 등록

```jsx
from django.contrib import admin
from .models import Post

**//방법1: 기본 ModelAdmin으로 등록**
admin.site.register(Post)

**//방법2: admin.ModelAdmin을 상속 (커스터마이징 가능)**
class PostAdmin(admin.ModelAdmin):
	list_display = ['id', 'message', 'created_at', 'updated_at']
admin.site(Post, PostAdmin)

**//방법3: 파이썬의 장식자(decorator) 형태로 등록 (커스터마이징 가능)**
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
	list_display = ['id', 'message', 'created_at', 'updated_at']
	list_display_links = ['message']
```

### 모델 클래스에 __str__ 구현

- admin 사이트의 모델 리스트에서 <모델명 object>로 나타나는 문자열을 마음대로 변경하기 위해 사용.
- 자바의 toString()과 유사.

```jsx
def __str__(self):
return f"Custom Post Object (({self.id})"

// Custom Post Object 1
// Custom Post Object 2
// ...
```

### ModelAdmin 옵션

- list_display, list_display_links, search_fields, list_filter 등

```jsx
//admin.py
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
	list_display = ['id', 'message', 'created_at', 'updated_at', 'is_public']
// admin 사이트 내의 모델 리스트에 출력할 컬럼 지정

	list_display_links = ['message']
//지정된 필드 중 detail 링크를 걸 필드 리스트

	list_filter = ['is public']
// 지정된 필드 값으로 필터링 옵션 제공

	search_fields = ['name']
// admin 사이트 내에서 검색 기준이 될 필드 리스트
```

```jsx
// **list_display** 에는 필드명 뿐 아니라 **모델의 member function**도 가능. **(함수의 인자가 없는 경우)**

*// models.py*
def message_length(self):
	return len(self.message)

*// admin.py*
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

```jsx
// settings.py

MEDIA_URL = '/media/'
// file에 접근할 때 사용
// 각 media file에 대한 URL prefix
// 필드명.url 속성에 의해서 참조되는 설정

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
// file을 저장할 때 사용
// MEDIA_ROOT 설정이 되어 있지 않다면, media file이 root에 바로 저장되어서 뒤죽박죽이 된다.
```

### FileField와 ImageField

- blank=True 옵션 적용.
- ImageField을 두고자 할 경우, Pillow를 먼저 설치해줘야 한다.
- 한 디렉토리(ex) setting.MEDIA_ROOT)에 너무 많은 파일이 쌓이지 않도록 ***upload_to 옵션***을 통해 저장 경로를 계산.

```jsx
photo = models.ImageField(blank=True, upload_to='instagram/post/%Y/%M/%D')
// 사진 파일은 'instagram/post/저장날짜(연)/저장날짜(월)/저장날짜(일) 폴더에 저장된다.

// 인자 유형에는 1) 문자열로 지정 2) 함수로 지정 이 있다.
// 1)은 file을 저장할 경로를 커스터마이징하고, 2)는 file명까지도 커스터마이징이 가능하다.

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

```jsx
from django.conf import settings
from django.conf.urls.static import static

if settings.DEBUG:
	urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
// False일 경우에는 빈 리스트
```

### 템플릿에서 media URL 처리

- .url 속성은 내부적으로 settings.MEDIA_URL과 조합을 처리.
- 따라서 필드에 저장된 값(경로)이 없을 경우 계산에 실패. 그러니까 안전하게 필드명 저장 유무를 체크.
- 파일 시스템 상의 절대경로가 필요하다면 .path 속성을 활용 (setting.MEDIA_ROOT와 조합)

```jsx
{% if post.photo %}
	<img src="{{ post.photo.url }}" />
{% endif %}
```

---