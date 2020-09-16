# 웹 프론트엔드 기초 및 장고 static

생성일: 2020년 9월 2일 오후 7:49

### WEB Front-End을 위한 3가지 언어

- HTML(Hyper Text Markup Language) : 웹페이지의 ***내용 및 구조***
- CSS(Cascading Style Sheet) : 웹페이지의 ***스타일***
- JavaScript : 웹페이지의 ***로직***
- 대부분 하나의 HTML 파일에 CSS/JavaScript 파일을 분리해서 관리
    1. HTML *응답 body 크기*를 줄일 수 있다.
    2. 여러 번 새로고침하더라도, *브라우저 캐싱  기능*을 통해 같은 파일을 서버로부터 다시 읽어들이지 않는다.
    3. 웹페이지 응답성을 높여준다.

### 웹 요청 및 응답

- 웹은 **HTTP(S) 프로토콜**로 동작한다.
- 클라이언트가 웹서버로 **하나의 요청**을 보내며, 웹서버는 **요청에 맞게 응답**을 해야 한다.
- 웹서버에서 응답을 만들 때 ***요청의 종류를 구분 하는 기준***으로는 ***URL(일반적), 요청헤더, 세션, 쿠키*** 등이 있다.
- 웹서버 구성에 따라 특정 요청에 대한 응답을 Apache/Nginx 웹서버에서 할 수도 있고, Django 뷰에서 응답을 할 수도 있다.

### 웹 요청 및 응답 순서

1. 브라우저는 서버로 HTTP 요청
2. 서버에서는 해당 HTTP 요청에 대한 처리 : 장고에서는 ***관련 뷰 함수가 호출***
3. ***뷰 함수에서 리턴해야만 비로소 HTTP 응답***이 시작되며, 그 HTTP 응답을 받기 전까지는 *하얀 화면*만 보여진다. 따라서 *뷰 처리시간이 길어질수록 빈 화면이 보여지는 시간이 길어진다.*
4. 브라우저는 서버로부터 HTTP 문자열 응답을 1줄씩 해석하여 그래피적으로 표현한다.

---

### CSS Framework

- 기본적인 CSS 스타일을 이미 구성해뒀기 때문에 초기 구성이 용이하다.
- 하지만, 같은 CSS Framework을 쓴 사이트는 같은 서비스인 것처럼 보여지기 때문에 커스텀해서 사용하는 게 좋다.

### CDN

- 최적화된 전세계적으로 촘촘히 분산된 서버로 이루어진 플랫폼
- 전 세계 유저에게 빠르고 안전한 정적 파일 전송
- **1개의 원본(Origin) 서버**를 가지고, CDN 서비스 업체에서는 **전 세계에 걸쳐 컨텐츠 서버**를 가지고 있고 ***원본 서버로부터 각 컨텐츠 서버로 데이터를 복제***한다.
- 전 세계의 유저들이 동일한 주소로 컨텐츠를 요청하면, CDN 서비스에서는 이 요청을 ***해당 유저와 물리적으로 가까운 CDN 컨텐츠 서버에서 응답***하도록 구성.

### CDN을 통한 BootStrap 적용법

```html
<head>
	..
	<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
	<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js" integrity="sha384-B4gt1jrGC7Jh4AgTPSdUtOBvfO8shuf57BaghqFfPlYxofvL8/KUEfYiJOMMV+rV" crossorigin="anonymous"></script>
	<script src="https://code.jquery.com/jquery-2.2.4.js" integrity="sha256-iT6Q9iMJYuQiMWNd9lDyBUStIq/8PuOW33aOqmvFpqI=" crossorigin="anonymous"></script>
	..
</head>
```

장고 프로젝트에서 사용할 정적 파일들은 가급적이면 우리가 관리하는 서비스 내에서 서빙하는 게 좋다. 외부에서 끌어 오는 경우에는 해외 망이 불안정하거나 하다면 CDN 서빙을 못 받을 수도 있다.

### Template 상속

```html
<!-- layout.html -->
<html>

<head>
	<title>
		{% block title %}
		{% endblock title %}
	</title>
</head>

<body>
	{% block content %}
	{% endblock content %}
</body>

</html>
```

```html
<!-- post_detail.html -->
{% extends 'instagram/layout.html' %}

{% block title %}
    Post Detail pk#{{ post.pk }}
{% endblock title %}

{% block content %}
    <h2>Author : {{ post.author }}</h2>

    {% if post.photo %}
        <div>
            <img src="{{ post.photo.url }}" />
        </div>
    {% endif %}

    {{ post.message }}
    <hr>
    <a href="{% url 'instagram:post_list' %}" class="btn btn-primary">
        목록
    </a>
{% endblock content %}
```

### 휴대폰 망을 통해 접속하는 방법

- 'python [manage.py](http://manage.py) runserver' 명령을 통해서 서버가 bind 되는 IP(default)는 127.0.0.1이다. bind 주소가 127.0.0.1 이면 서버를 실행시킨 컴퓨터에서만 접속이 가능하다.
- 같은 네트워크의 다른 컴퓨터에서 접속하려면, bind 주소를 해당 컴퓨터의 IP로 설정하거나, 0.0.0.0으로 지정해서 컴퓨터가 가진 모든 IP에 대해 접속을 받을 수 있도록 설정해야 한다. (python [manage.py](http://manage.py) runserver 0.0.0.0:8000)
- 외부망에서 접속하려면 외부 네트워크 설정이 추가로 필요하다.

![https://wayhome25.github.io/assets/post-img/django/ngrok.png](https://wayhome25.github.io/assets/post-img/django/ngrok.png)

- ***ngrok***을 통해 ***개발 단계***에서 [localhost](http://localhost) 터널을 열어서 외부에서 접근을 가능케 한다. (개발 단계에서만 사용한다)(포트포워딩 등의 설정이 필요 없다)
    1. ngrok를 다운, 압축을 풀어 ngrok 실행 파일을 [manage.py](http://manage.py/) 가 존재하는 장고 프로젝트 경로로 복사
    2. 장고 개발 서버를 8000 포트로 (디폴트) 구동

    ```bash
    $ python manage.py runsever 8000
    $ ./ngrok http 8000
    ```

    - ngrok 실행을 나타나는 주소를 ***settings.ALLOWED_HOSTS***에 추가한다.

    ```python
    ALLOWED_HOSTS = [
    	'f1ee182d.ngrok.io',
    ]
    ```

    ---

    ### Static 파일

    - 개발 리소스로서의 ***정적 파일***(js, css, image 등)
    - ***앱/프로젝트 단위로 저장/서빙***

    ### Static 파일을 다루는 방법

    - ***하나의 앱을 위한 static 파***일은 app***/static/app*** 경로에 둔다.
    - ***프로젝트 전반적으로 사용되는 static 파일***은 ***settings.STATICFILES_DIR***로 지정된 경로에 둔다.
    - 따라서 여러 군데 분산되어 있는 static 파일을 ***'collectstatic' 명령을 통해서 settings.STATIC_ROOT로 지정된 경로에 모아서(복사)*** 서비스에 사용한다.

    ```python
    STATIC_URL = '/static/'
    # 각 static 파일에 대한 URL Prefix
    # 템플릿 태그 {% static "경로" %}에서 참조되는 설정

    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
    # 프로젝트 내의 static 파일들이 여기로 모여서(복사) 서비스 된다

    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'askcompany', 'static'),
    ]
    # File System Loader에 의해 참조되는 설정
    ```

    ### 템플릿에서 static URL 처리 방법

    - settings.STATIC_URL, Prefix를 하드코딩 하기

    ```html
    <img src="/static/blog.title.png" />
    ```

    하지만, settings.STATIC_URL 설정은 언제라도/프로젝트마다 변경될 수 있다. 또한 하드코딩 하는 게 번거롭고, 변경이 된다면 일일이 수정해야 한다.

    - Template Tag를 통한 처리

    ```html
    {% load static %}
    <img src="{% static "blog/title.png" %}" />
    ```

    ***프로젝트 설정에 따라 유연하게 static url prefix을 할당***할 수 있다.

    ### collectstatic 명령

    - ***실 서비스 배포 전에는 반드시 collectstatic 명령***을 통해, 여러 디렉토리에 나눠져있는 static 파일들을 한 곳으로 복사해야 한다.
    - 왜냐하면, ***여러 디렉토리에 나눠 저장된 static 파일들의 위치는 '현재 장고 프로젝트'만 알고 있으며*** 외부 웹 서버는 전혀 알지 못한다.
    - 외부 웹 서버에서 Finder의 도움 없이도 static 파일들을 서빙하기 위함. (한 디렉토리에 모두 모여 있다면 Finder의 도움이 필요 없다)