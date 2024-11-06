---
sidebar_position: 1
---

# 컨텐츠 보안 정책 (CSP)

## Content Security Policy

- 크로스 사이트 스크립팅(XSS)과 데이터 주입 공격을 방지하기 위한 **추가 보안 계층**
  > **크로스 사이트 스크립팅(XSS)**
  >
  > - 웹 서버 사용자에 대한 입력값 검증이 미흡할 때 발생하는 취약점
  >   - 브라우저가 서버에서 받은 콘텐츠를 신뢰한다는 점을 악용한 것
  > - 웹 사이트 관리자가 아닌 이가 웹 페이지에 악성 스크립트를 삽입하는 것
  > - 가장 발생하기 쉬운 시나리오
  >   - 게시판 등 사용자 입력 콘텐츠에서 사용자가 입력한 내용을 아무런 필터링 없이 공개하는 것
  >     1. 텍스트 상자에 악의적인 스크립트 삽입 후 데이터 입력
  >     2. 웹 서비스 쪽 프로그램에서 입력된 값을 전혀 확인 하지 않은채 HTML 에 넣어 바로 출력
  >     3. 악의적인 스크립트가 삽입된 채로 게시 되어 콘텐츠를 보는 사람의 브라우저에서 그대로 실행
- 공격 유형 : 데이터 절도, 사이트 훼손, 맬 웨어 배포 등

- 웹 서버는 HTTP 헤더에 **Content-Security-Policy** 헤더를 포함시켜, 브라우저가 특정 리소스만 로드하도록 제어할 수 있습니다.
  ```html
  Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com;
  ```
- 이렇게 설정하면 악성 스크립트나 리소스가 로드되지 않도록 방지하여, **XSS** 등의 공격을 차단하는 데 도움이 됩니다.
- `<meta>` 요소를 사용하여 정책을 구성할 수도 있다.
  ```html
  <meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';" />
  ```

:::info
CSP 위반 보고서 전송과 같은 일부 기능은 HTTP 헤더를 사용할 때만 사용할 수 있다.
:::

## [Threats](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP#threats)(위협)

### **1. 크로스 사이트 스크리팅(XSS) 완화**

- CSP 주요 목표 : XSS 공격을 완화하고 보고하는 것
- 서버 관리자가 브라우저에서 실행 가능한 스크립트의 유효한 소스로 간주해야하는 출처(도메인)을 지정하여 XSS 공격을 방지할 수 있다.
  - CSP 호환 브라우저는 허용한 도메인에서 받은 소스 파일에서 로드된 스크립트만 실행
    ```tsx
    Content-Security-Policy: default-src 'self'
     example.com *.example.com
    ```
  - HTML 속성을 포함한 인라인 스크립트 및 이벤트 처리 등의 다른 모든 스크립트를 무시
    ```tsx
    Content-Security-Policy: default-src 'self';
    	script-src 'self'; style-src 'self';
    ```
    - `default-src 'self'`: 모든 리소스를 동일한 출처에서만 로드한다.
    - `script-src 'self'`: 자바스크립트는 동일한 출처에서만 로드되며, 인라인 스크립트나 이벤트 핸들러는 차단된다.
    - `'unsafe-inline'`을 명시하지 않으면 인라인 스크립트는 기본적으로 차단

### **2. 패킷 스니핑 공격 완화**

> **패킷 스니핑(Packet Sniffing)**
>
> - 네트워크를 통해 전송되는 데이터를 도청하거나 가로채는 행위
> - 공격자는 네트워크 상에서 오가는 패킷(데이터)을 감시하고, 이를 통해 사용자 이름, 비밀번호, 쿠키, 세션 정보와 같은 **민감한 데이터를 훔칠 수 있다**.

- 기본적으로 **리소스의 출처**를 제한하지만,
  **HTTPS** 같은 안전한 프로토콜을 사용하도록 지정할 수도 있다.
- 예시 ) 모든 콘텐츠가 HTTPS를 사용하여 로드되도록 지정 가능
  ```tsx
  Content-Security-Policy: default-src https:;
  ```
  - **HSTS (HTTP Strict Transport Security)**
    - 모든 쿠키에 secure 속성을 표시하고 HTTP 페이지는 해당 HTTPS 페이지로 자동 리디렉션을 제공한다.
    - HTTP 헤더 Strict-Transport-Security를 사용하여 브라우저가 암호화된 채널을 통해서만 사이트에 연결하도록 할 수 있다.
      ```tsx
      Strict-Transport-Security: max-age=31536000; includeSubDomains
      ```

## CSP 사용하기

HTTP 헤더에 Content-Security-Policy를 웹 페이지에 추가하고 사용자 에이전트가 해당 페이지에 대해 로드할 수 있는 리소스를 제어하는 값을 지정

content-Security-Plicy로 리소스에 대해 설정할 수 있는 데이터 속성

> | 키워드 / 데이터 속성 | 설명                                                                                                                                  |
> | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
> | none                 | 로드를 금지한다.                                                                                                                      |
> | self                 | 같은 출처를 지정한다.                                                                                                                 |
> | unsafe-inline        | 스크립트 및 인라인의 `<script>`태그, <br/> 이벤트 핸들러의 자바스크립트 : 표기, 인라인의 `<style>`을 허가한다.<br/> - XSS의 위험 존재 |
> | unsafe-eval          | 문자열을 자바스크립트로서 실행하는 `eval()`, `new Function()`, `setTimeout()` 등의 실행을 허가한다. <br/> - XSS의 위험 존재           |
> | data:                | data URI를 허가한다.                                                                                                                  |
> | mediastream          | mediastream:URI 허가한다.                                                                                                             |
> | bolb:                | blob:URI 를 허가한다.                                                                                                                 |
> | filesystem:          | filesystem:URI 를 허가한다.                                                                                                           |
>
> - mediastream: 데이터 속성 - HTML5 스트리밍에 사용
> - data: 데이터 속성 - BASE64로 인코딩 된 이미지 파일의 문자열을 `<image>` 태그의 소스로 설정하거나 CSS 텍스트에서 이미지 데이터를 삽입하여 이미지를 표시할 때 사용한다.

### 정책 지정하기

```tsx
// HTTP 헤더
Content-Security-Policy: policy
```

- default-src
  - CSP 구문이 정의되지 않았을 때 이를 대체하는 용도
- 인라인 스크립트 실행 방지 및 `eval()` 사용 차단 하려면, default-src 또는 script-src 지시문 포함 필요
- 인라인 스타일이 `<style>` 요소 또는 `style` 속성에 적용되는 것을 제한 하려면 default-src 또는 style-src 지시문 포함 필요

### 일반적인 사용 사례

- **예제 1)**
  - 모든 콘텐츠가 사이트 자체 출처(하위 도메인 제외)에서 오기를 원함
    ```tsx
    Content-Security-Policy: default-src 'self'
    ```
- **예제** 2)
  - 신뢰할 수 있는 도메인 및 모든 하위 도메인의 콘텐츠를 허용
    - CSP 가 설정된 도메인과 동일할 필요는 없다.
      - **특정 신뢰할 수 있는 도메인**에서 제공되는 리소스를 사용할 수 있도록 설정 가능.
        예를 들어, example.com에서 제공하는 CDN(Content Delivery Network)이나 라이브러리를 사용할 수 있도록 허용하는 것.
    ```tsx
    Content-Security-Policy: default-src 'self'
     example.com *.example.com
    ```
  - ‘self’ : CSP가 설정된 웹사이트의 동일 출처(same-origin)에서 리소스를 로드 가능
  - 웹 페이지가 호스팅된 서버로부터만 스크립트, 이미지, 스타일 시트 등을 로드할 수 있다.
- **예제 3)**
  - 자신의 콘텐츠에 모든 원본 이미지를 포함할 수 있게 허용하지만,
    오디오 또는 비디오 미디어는 신뢰할 수 있는 공급자로 제한하고
    모든 스크립트는 신뢰할 수 있는 코드를 호스팅하는 특정 서버로만 제한
    ```tsx
    Content-Security-Policy: default-src 'self';
    img-src *;
    media-src example.org example.net;
    script-src userscripts.example.com
    ```
    - 이미지는 출처에 상관없이 로드 가능 (’\*’: 와일드 카드)
    - 미디어는 example.org 및 [example.net](http://example.net) 에서만 허용, 하위 도메인 비허용
    - userscripts.example.com 에서 온 스크립트만 실행 가능
- **예제 4)**
  - 온라인 뱅킹 사이트의 웹 사이트 관리자는 공격자가 요청을 도청하는 것을 방지하기 위해
    모든 콘텐츠가 TLS를 사용하여 로드되었는지 확인
    ```tsx
    Content-Security-Policy: default-src
    https://onlinebanking.example.com
    ```
- **예제 4)**
  - 전자 메일에 HTML을 허용하고 어디에서나 로드된 이미지 허용
    Javascript 또는 기타 잠재적 위험한 콘텐츠는 허용하지 않음
    ```tsx
    Content-Security-Policy: default-src 'self'
    *.example.com; img-src *
    ```
    - script-src를 지정하지 않고 default-src 사용해서, 원본 서버에서만 스크립트 로드 가능하게 함

## 정책 테스트하기

- Content-Security-Policy는 브라우저가 실시하는 검사
  - 오류는 클라이언트 쪽에 표시된다.
    - 브라우저의 개발자 도구(console)에서 확인 가능
  - 서버 개발자가 그 오류를 바로 볼 수 없다.
    - 그러나 오류 보고서를 보낼 곳을 지정해, 클라이언트가 오류 정보를 서버로 통지할 수 있다.
      - **report-uri**나 **report-to**를 사용해 위반 보고서를 수집할 수 있다. 이를 위한 외부 서비스도 존재한다(예: Report URI).
- CSP는 **XSS 대책**으로 매우 강력한 수단이 될 수 있지만, 너무 엄격한 설정은 정상적인 웹사이트 동작을 방해할 수 있다.
- **Content-Security-Policy-Report-Only** 헤더를 사용하면 정책 위반 사항을 보고만 하고, 실제로 차단하지는 않기 때문에 미리 테스트할 수 있다.
  ```
  Content-Security-Policy-Report-Only: policy
  ```
  - 인라인 스크립트를 모두 중지하면, **Google Analytics**와 같은 도구가 정상적으로 작동하지 않을 수 있다. 이 경우, 적절한 설정을 통해 문제를 해결해야 한다.

## 보고 활성화

- 일반적으로 위반 보고서는 전송되지 않는다.
- 보고서를 전달받을 하나 이상의 URI를 report-to 정책 지시문에 지정 필요

```tsx
Content-Security-Policy: default-src 'self';
report-to
http://reportcollector.example.com/collector.cgi

```

- 서버에서 보고서를 수신하도록 설정

## 위반 보고서 예시

- http://example.com/signup.html에 있는 페이지
- cdn.example.com의 스타일시트를 제외한 모든 항목은 허용하지 않도록 설정

```tsx
Content-Security-Policy: default-src 'none';
style-src cdn.example.com;
report-to /_/csp-reports
```

```tsx
<!doctype html>
<html lang="en-US">
  <head>
    <meta charset="UTF-8" />
    <title>Sign Up</title>
    <link rel="stylesheet" href="css/style.css" />
  </head>
  <body>
    여기에 컨텐츠.
  </body>
</html>

```

- 문제점 :
  - 스타일시트는 `cdn.example.com`에서만 로드할 수 있지만
    웹사이트는 자체 원본(`http://example.com`)에서 스타일시트를 로드하려고 한다.
- 위반보고서 :
  - CSP를 적용할 수 있는 브라우저는 문서를 방문할 때 위반 보고서를 `http://example.com/_/csp-reports`에 POST 요청으로 보낸다.
  ```json
  {
    "csp-report": {
      "blocked-uri": "http://example.com/css/style.css", // 위반경로
      "disposition": "report",
      "document-uri": "http://example.com/signup.html",
      "effective-directive": "style-src-elem",
      "original-policy": "default-src 'none'; style-src cdn.example.com; report-to /_/csp-reports",
      "referrer": "",
      "status-code": 200,
      "violated-directive": "style-src-elem"
    }
  }
  ```
