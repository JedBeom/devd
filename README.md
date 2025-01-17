
[![Travis Build Status](https://travis-ci.org/cortesi/devd.svg?branch=master)](https://travis-ci.org/cortesi/devd)



# devd: 개발자를 위한 로컬 웹서버

![스크린샷](docs/devd-terminal.png "devd 실행화면")

# 설치

[릴리즈 페이지](https://github.com/cortesi/devd/releases/latest)에 가서, OS에 맞는 패키지를 다운로드하고 실행파일을 PATH로 복사하세요.

Go가 설치되어 있다면, 다음 명령어를 사용할 수 있습니다.

    go get github.com/cortesi/devd/cmd/devd

# 빠르게 시작하기

현재 디렉터리를 제공하고, 브라우저에서 열고(**-o**), 파일이 바뀌면 실시간으로 새로고침합니다(**-l**):


```bash
devd -ol .
```

http://localhost:8080으로 리버스 프록시를 구성하고, **src** 디렉터리의 파일이 바뀌었을 때만 실시간 새로고침합니다:

```bash
devd -w ./src http://localhost:8080
```


# modd와 함께 devd 사용하기

[modd](https://github.conrtesi/modd)는 파일 시스템의 변경에 맞추어 명령어를 실행하고 데몬을 관리하는
개발 도구인 devd의 자매 프로젝트입니다.
devd는 파일 시스템 변경이 감지되었을 때 프로젝트를 재빌드하고 브라우저를 새로고침할 때
modd와 함께 사용될 수 있습니다.

상황을 설명하기 위한 간단한 *modd.conf* 파일 예시입니다.

```
src/** {
    prep: render ./src ./rendered
}

rendered/*.css ./rendered/*.html {
    daemon: devd -m ./rendered
}
```

첫번째 블록은 *src* 디렉터리의 어떤 것이라도 변경되었을 때 *render* 스크립트를
실행합니다. 두번째 블록은 *rendered* 속 .css나 .html 파일이 변경되었을 때 devd
인스턴스를 시작하고 신호와 함께 실시간 새로고침을 발동합니다.

자세한 내용은 [modd](https://github.com/cortesi/modd) 프로젝트 페이지를 참고하세요.


# 특징

### 크로스 플랫폼과 자체포함

devd는 의도적으로 외부 의존이 없는 하나의 컴파일된 실행파일이고, macOS, Linux, Windows로
릴리즈 되었습니다. 작업 중인 가벼운 도커 인스턴스에 Node나 Python을 설치하고 싶지
않다구요? 그냥 devd 실행파일을 복사하면 해결됩니다.


### 터미널을 위해 디자인됨

위 말은 실행 파일과 데몬이 없고, 개발자가 터미널에서 읽을 수 있게 디자인된 로그가
있다는 것을 말합니다. 로그는 컬러이고 로그 엔티티는 여러 줄을 포함합니다.
devd의 로고는 자세하고, 다른 데몬들이 무시하는 외딴 문제들을 경고하고, 자세한
타이밍 정보나 전체 헤더 같은 것들을 추가적으로 포함할 수 있습니다.

### 편리성

가능한 간단하게 인스턴스를 빠르게 실행하기 위해서, devd는 (명시되지 않았다면)
실행할 열린 포트를 자동으로 선택하고, 사용자를 위해 데몬 루트를 가리키는 브라우저
윈도우를 열 수 있습니다(위의 예시의 **-o** 플래그). 한 번에 devd를 위한 자가 서명 인증서를
자동으로 생성하고, ~/.devd.certs에 저장하고 TLS를 활성화 시키는 **-s** 플래그 같은 유틸성
특징도 있습니다.

### Livereload

실시간 새로고침이 활성화되면, devd는 HTML 페이지의 *head* 닫는 대그 직전에 작은 스크립트를 
삽입합니다. 그 스크립트는 websocket 연결을 통해 변경 알림을 듣고, 필요한 리소스들을
새로고침합니다. 브라우저 확장 프로그램은 필요하지 않고, 리버스 프록시 앱에서도 
실시간 새로고침은 작동합니다. 만약 CSS 파일의 변경만 있다면, devd는 외부 CSS 리소스만
새로고침하고, 그렇지 않다면 페이지 전체를 새로고침합니다. 아래 명령은 실시간 새로고침을
활성화하고 현재 디렉터리를 제공합니다.

<pre class="terminal">devd -l .</pre>

소스 파일이 변경되었을 때 리버스 프록시 앱을 새로고침하면서 제공되지 않는
파일의 새로고침을 촉발할 수 있습니다. 아래 명령은 *src* 디렉터리 트리와
, 로컬에서 실행되는 앱을 향하는 리버스 프록시를 관찰합니다:

<pre class="terminal">devd -w ./src http://localhost:8888</pre>

**-x** 플래그는 [패턴 명시](#실시간-새로고침으로부터-파일-제외)에 기반한 실시간
새로고침에서 파일을 제외합니다. 다음 명령은 ".less" 확장자를 가진 모든 파일의
실시간 새로고침을 비활성화합니다:

```terminal
devd -x "**.less" -l .
```

실시간 새로고침이 활성화 되었을 때 (**-L**, **-l**, **-w** 플래그와 함께) devd는
모든 연결된 브라우저에 실시간 새로고침 공지를 알림으로써 SIGHUP에 응답합니다.
이는 devd의 자매 프로젝트인 **modd** 같은 외부 도구가 실시간 새로고침을
촉발할 수 있게 합니다. 만약 실시간 새로고침이 활성화 되지 않으면, SIGHUP은
데몬이 종료하도록 합니다.

*head* 닫는 태그는 원격 파일의 처음 30KB에 있어야 하는데, 그렇지 않으면
그 파일의 실시간 새로고침은 비활성화 됩니다.


### 리버스 프록시 + 정적 파일 서버 + 유동적인 라우팅

모던 앱은 웹 서버들의 집합이 되는 경향이 있는데, devd는 유동적인
리버스 프록시로 이를 제공합니다. 하나의 영역에 서비스의 집합을 더하기 위해,
 네이티브적으로 지원하지 않는 서비스에 실시간 새로고침을 추가하기 위해, 이미 있는
 서비스에 쓰로틀링과 지연 시뮬레이션을 추가하기 위해서 그리고 다른 일들을 위해서
 devd를 사용할 수 있습니다.

이 모든 것들을 한 번에 할 수 있는 더 복잡한 예시가 있습니다. 두 앱과 정적 파일들을
더합니다. 실시간 새로고침은 정적 파일에 활성화 되어있고(**-l**), 리버스 프록시 앱의
소스 파일이 변경되었을 때도 촉발됩니다(**-w**).

<pre class="terminal">
devd -l \
-w ./src/ \
/=http://localhost:8888 \
/api/=http://localhost:8889 \
/static/=./assets
</pre>

[라우트 명시 문법](#라우트)은 간단하지만 대부분의 사용 사례를 잡기에는 충분히 강력합니다.

### 가벼운 가상 호스팅

devd는 간단한 가상 호스팅을 위해 **devd.io**라는 전용 도메인을 사용합니다.
이 도메인과 아래의 서브 도메인은 127.0.0.1을 가리키는데, */etc/hosts*나 다른
로컬 설정을 변경하지 않고 가상 호스팅을 설정할 때 사용합니다.
**/**로 시작하지 않는 라우트 명시는 **devd.io**의 서브 도메인으로 사용됩니다.
그러므로, 아래 명령은 devd.io에서 정적 파일을 제공하며, 로컬에서 실행 중인
앱을 api.devd.io로의 리버스 프록시를 구성합니다.

<pre class="terminal">
devd ./static api=http://localhost:8888
</pre>


### 지연과 대역폭 시뮬레이션

미얀마의 스마트폰으로 여러분의 멋진 5MB HTML5 앱을 사용하면 어떨지 궁금한가요?
[여기](http://www.cisco.com/c/en/us/solutions/collateral/service-provider/global-cloud-index-gci/CloudIndex_Supplement.html)에서
대역폭과 지연을 찾아보고 devd를 설정하세요. (초당 킬로비트를 초당 킬로바이트로
전환하는 것과 서버의 위치를 고려하는 걸 잊지 마세요.)

<pre class="terminal">devd -d 114 -u 51 -n 275 .</pre>

devd는 대역폭와 지연을 상당히 정확하게 시뮬레이션하려고 시도합니다.
쓰로틀링을 위해 토큰 버켓 실행을 사용하고, 적절하게 동시성 요청을 수행하고,
데이터의 흐름이 부드럽도록 트래픽을 넣습니다.


## 라우트

devd 명령은 하나 이상의 라우트 명시를 인자로 받습니다. 라우트는
**root=endpoint**의 기본적인 형식을 갖습니다. "/favicon.ico"나 "/images/"
(뒷부분의 슬래시를 유의하세요) 같은 서브 트리 같이 루트가 고정될 수 있습니다.
엔드포인트는 HTTP 서버로 보내질 파일 시스템의 경로거나 URL이 될 수 있습니다.

이하는 */assets* 아래에 디렉터리 *./static*을 서버에 제공하는 라우트입니다:

```
/assets/=./static
```

**devd.io** 서브 도메인(127.0.0.1을 가리킵니다)을 사용하기 위해서는, 루트 명시
앞에 추가하세요. 서브 도메인은 **/**로 시작하지 않는다는 사실로 서브도메인을
구별합니다. 그러므로 이하의 라우트는 **static.devd.io/assets**로 **/static**
디렉터리를 제공합니다.

```
static/assets=./static
```

리버스 프록시 명시는 비슷한데, 엔드포인트 명시는 URL입니다. 이하의 명령은 
루트 **app.devd.io/login**에서 로컬 URL을 제공합니다.

```
app/login=http://localhost:8888
```

만약 **root** 명시가 누락되면, 모든 경로에 맞는 패턴인 "/"인 것으로 추정합니다.
그러므로, 간단한 디렉터리 명시는 **devd.io**에서 디렉터리를 제공합니다.

```
devd ./static
```

비슷하게, 간단한 리버스 프록시는 이하와 같이 할 수 있습니다:

```
devd http://localhost:8888
```

localhost를 가리키는 리버스 프록시를 만드는 단축어도 있습니다:

```
devd :8888

```

### 찾을 수 없는 파일에 기본 콘텐츠를 제공하기

**--notfound** 플래그는 여러 번 전달될 수 있고, 정적 파일 서버가 요청된 파일을
찾을 수 없을 때 참조하는 라우트의 집합을 명시합니다. 기본 문법은 **root=path**인데,
**root**는 라우트 명시와 같은 구문입니다. 라우트와 함께라면, **root=** 구성 요소는
부수적이고, 만약 없을 시에는 "/"과 같게 취급됩니다. **경로**는 언제나 제공되는
정적 디렉터리에 상대적입니다. 슬래시("/")가 시작에 있다면, devd는 트리의 루트에
상대적인 하나의 위치에서만 대체 파일을 찾습니다. 그렇지 않다면, 트리의 루트까지의 
모든 경로 구성요소와 함께 맞는 파일을 찾습니다.

예시와 함께 표현해보겠습니다. 다음과 같이 */static* 디렉터리가 있다고 합시다.

```
./static
├── bar
│   └── index.html
└── index.html
```

devd가 정적 트리의 루트를 향하는 경로 어디에서나 *index.html*을 찾도록
다음과 같이 명시할 수 있습니다:

```
devd --notfound index.html  /static
```

이제, 이하의 상황이 일어납니다:

* */존재하지않는.html*로의 요청은 */index.html*의 내용을 반환합니다.
* */bar/존재하지않는.html*로의 요청은 */bar/index.html*의 내용을 반환합니다.
* */foo/bar/voting/index.html*로의 요청은 */index.html*의 내용을 반환합니다.

이상의 모든 예시에 */index.html*의 내용이 반환되어야 할 때, 라우트에
절대 경로를 대신 명시할 수 있습니다.

```
devd --notfound /index.html  /static
```
만약 들어오는 요청의 예상되는 타입이 무효 명시와 맞지 않는다면 devd는 무효된
페이지를 제공하지 않습니다. 만약 타입이 분명하게 확인되지 않을 때 *text/html*을
기본으로 하면서, 무시와 요청의 파일 확장자와 예상되는 MIME 타입을 확인해서
이를 수행합니다.

## 실시간 새로고침으로부터 파일 제외

**-x** 플래그는 다음 용어를 지원합니다:

용어          | 뜻
------------- | -------
`*`           | 어떠한 비경로 구분자의 연속과 일치합니다.
`**`          | 경로 구분자를 포함해 어떠한 문자의 연속과 일치합니다.
`?`           | 어떤 하나의 비경로 구분자 문자와 일치합니다.
`[class]`     | 문자의 종류(class)가 아닌 어떠한 하나의 비경로 구분자와 일치합니다.
`{alt1,...}`  | 쉼표로 구분된 후보들 중 하나와 일치하면 문자의 연속과 일치합니다.

특별한 의미를 지닌 어떠한 문자도 백슬래시(`\`)로 이스케이프될 수 있습니다. 문자 종류는 이하를 지원합니다:

종류(class) | 뜻
----------- | -------
`[abc]`     | 집합에 있는 어떤 문자 하나와 일치합니다.
`[a-z]`     | 범위 안의 어떤 문자 하나와 일치합니다.
`[^class]`  | 종류와 일치하지 *않는* 어떤 문자 하나와 일치합니다.

## 리버스 프록시에 대하여

devd는 리버스 프록시에서 전송되는 SSL 인증서를 검증하지 않습니다.
저희가 사용하는 사례에서는 개발 서버는 보통 테스트를 위한 자가 서명 인증서와
주로 로컬에서 실행됩니다. 전송되는 인증서의 검증이 중요한 사례에서는 devd를
사용하시면 안됩니다.

리버스 프록시된 트래픽을 위한 devd 서버의 주소와 프로토콜에
*X-Forwarded-Host*와 *X-Forwarded-Proto* 헤더가 설정되어 있습니다.
앱이 리다이렉트와 등등이 올바르게 작동하기 위해서 앱에 이것들을
지원하도록 해주십시오.

# 개발

배포를 위해 이 패키지를 빌드하는데 사용된 스크립트는 [여기](https://github.com/cortesi/godist)에서
찾으실 수 있습니다. 외부 패키지는 [dep](https://github.com/golang/dep)을 이용해 포함되었습니다.

