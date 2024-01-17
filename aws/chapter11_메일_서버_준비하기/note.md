![img.png](img.png)

웹에 공개되는 어플리케이션은 신규 사용자 등록이나 구매 확인 등 다양한 시점에 메일을 이용한다.

메일 송수신시에는 전용 메일 서버를 일반적으로 이용하며 AWS에서도 메일 송신을 구현하는 서비스를 제공한다.
# 목차
# 11.1 메일 시스템
![img_1.png](img_1.png)

메일 주소는 사용자가 소속된 조직에서 할당 받는다.

wonho@naver.com 과 같은 형식이며 앳(@,at) 기호 뒤에는조직이 관리하는 도메인을 쓴다. wonho 부분은 조직 내에서 중복되지 않는다.

## 11.1.1 메일 전송 시스템
![img_2.png](img_2.png)

> 1. 엘리스가 자신이 속한 조직의 메일 서버에서 밥을 수신자로 한 메일 송신을 요청한다.
> 2. 인터넷에서수신자의 밥 메일 서버를 검색하고 메일을 전송한다.
> 3. 수신지 메일 서버의 메일함에 저장된다.
> 4. 밥이 확인한다.

메일 사양은 크게 다음 두 가지로 구분될 수 있다.

![img_3.png](img_3.png)

SMTP는 송신자가 송신한 메일이 수신자의 메일함에 도착할때까지의 과정을 담당한다. (1~3)
POP3와 IMAP4는 수신자가 자신의 메일함에 있는 메일을 해서 읽는 부분을 담당한다. (4)

---

## 11.1.2 POP3와 IMAP4 비교
![img_4.png](img_4.png)

큰 차이점은 최종적으로 도착한 메일이 저장되는 위치다.

**POP3** 는 메일 소프트웨어에 따라 메일함 안의 메일을 로컬 컴퓨터에 내려 받는다.

한 번 내려받은 뒤에는 네트워크에 연결되지 않은 상태에서도 읽을 수 있지만, 다른 컴퓨터에서는 읽을 수 없다.

**IMAP4** 는 브라우저 등을 이용해 직접 메일함의 메일을 읽는다.

인터넷에 연결되어 있기만 하면 어떤 컴퓨터에서나 메일을 읽을 수 있지만 인터넷 연결이 되어있지 않으면 못 본다.

> 과거 메일의 경우 POP3는 로컬 컴퓨터의 하드 디스크에 여유가 있는 한 계속 저장했지만,  
> 
> IMAP4는 메일함의 상한선 용량까지만 저장할 수 있어 주의가 필요했다.

---

# 11.2 Amazon SES
Amazon Simple Message Service는 메일 송수신 기능을 제공하는 AWS의 매니지드 서비스이다.

단, 일반적인 메일과는 용도가 조금 다른데, 사람이 아니라 에플리케이션에서 메일을 송수신 하는데 활용하기 좋은 기능이다.

---

## 11.2.1 메일 송신
과거 메일 서버에서는 조직에 소속된 많은 사람들을 메일 서버에 등록한다.

해당 사용자가 메일 서버에 SMTP로 접속해 매일을 송신하고 접속시에는 ID와 패스워드를 입력했다.


![img_5.png](img_5.png)
어플리케이션으로부터 메일을 송신할 때 메일 송신자는 사람이 아닌 system@example.com 또는 no-reply.com 

과 같은 계정이 된다. Amazon SES에서는 이런 특별한 계정들을 IAM으로 등록하고, 해당 IAM 사용자를통해 메일을 송신한다.

IAM 사용자를 이용한 SES 연결 시의 인증 방법은 크게 아래 두 가지가 있다.

> 1. Amazon SES API
> 2. Amazon SES SMTP 인터페이스

1번은 AMAZON SES가 제공하는 API를 경유해서 직접 AMAzon SES와 데이터를 주고 받는 방법이다.

이용하는 프로그램 언어용 SDK와 AWS 커멘드라인 인터페이스를 이용해 메일을 송신할수 있다.

2번은 일반적인 메일 서버와 같은 SMTP를 사용해 메일을 송신할 수 있다.

---

## 11.2.2 메일 수신
메일 수신도 일반적인 메일 서버와 크게 다르다.

일반적인 메일 서버는 도착한 메일을 각 사용자의 메일함에 저장한다.

수신자는 POP3나 IMAP4와 같은 프로토콜을 이용해 메일을 읽는다.

그러나 Amazon SES는 POP3 혹은 IMAP4와 같은 프로토콜은 제공하지 않고 대신 메일을 수신했을 때

**엑션** 이라불리는 처리를 실행한다.

액션을 이용해 애플리케이션에서 제공하는 커스텀 API를 실행할 수 있으므로 사용자로부터 도착한 메일을 실시간 처리할 수 있다.

![img_6.png](img_6.png)

SNS 액션을 등록해두면, 사용자로부터 메일이 도착했을 때 관리자에게 메일이나 휴대전화 푸시 알림을 보낼 수 있다.

또한 람다를 연동해 커스텀 API를 호출할 수 있다.

이렇게 Amazon SES에서는 메일을 수신했을 때 실시간으로 다양한 처리를 계속해서 실행할 수 있다.
이를 통해 메일에 자동으로 대응할 수 있다.

> SES에서의 메일 수신
> 
> SES 에서는 POP3나 IMAP4를 제공하지 않는다. 즉관리자가 메일을 수동으로 대응할 수 없다.
> 
> 대신 엑션 기능을 고려하면 소위 고객 관리 시스템 이라 불리는 시스템과 연동할 수 있다.
> 
> 사용자로부터 메일을 이용한 문의에 어떻게 대응할지 검토한 후에 메일 수신 기능을 구현하거나 다른 메일 서버로 구현할지 판단한다.
> 
---

### 11.2.3 SES를 생성하는 리전
2020년 7월부터 서울 리전도 SES를 이용할 수 있으나, 2022년 3월 기준 송신만할 수 있다.

수신도 이용하려면 버지니아 리전을 사용해야 한다.

---

### 11.2.4 샌드박스
샌드박스는 외부에 영향을 주지 않도록 격리된 환경을 말한다.

SES 생성 초기에는 악용을 방지하기 위해 샌드 박스 내부에 위치하며 아래와 같은 제약이 있다.

> 1. 송신 대상지는 검증된 주소로만 한정한다.
> 2. 송신지는 검증된 주소 또는 등록한 도메인으로만 한정한다.
> 3. 송수신할 수 있는 메일 건 수가 200건/24시간 또는 1건/1초로 제한된다.

외부로 SES를 이동시키려면 AWS 지원 센터로 요청해야 한다.

---

# 11.3 메일 송수신 기능 생성하기
SES 를 생성하여 메일 송/수신 기능을 하는 서버를 생성해본다.

송수신을 모두 수행하는 SES를 만들기위해 버지니아 북부에 생성했다.

---

## 11.3.1 도메인 설정 내용
먼저 메일 서버로 도메인을 설정한다. 아래 표와 같다.
![img_7.png](img_7.png)

---

## 11.3.2 도메인 설정 순서
서비스 > SES 대시보드 > Configuration > Verified indeintities를 클릭하여 create Identity 버튼을 클릭한다.

![img_8.png](img_8.png)

다음은 추가할 도메인 이름을 선택한다. 보인 인증 유형에서 도메인을 선택하고 aws-intro-sample.com을 입력한다.

Assign a default configuration set / use a custom MAIL FROM domain 옵션은 선택하지 않아도 된다.

![img_9.png](img_9.png)

도메인 확인 란에서는 DKIM (DomainKeys Identified Main) 설정 방법을 정한다.

DKIM은 메일에 전자 서명을 넣어 위변조된 메일이 아님을 보증하며 무료로 사용 가능하다.

![img_10.png](img_10.png)

> SES에서 관리하는 도메인을 일반 DNS 서버에서 관리한다면, 여기 쓰인 정보를
> 
> DNS 레코드를 이용해 수동으로 추가해서 대응할 수 있다.
> 

다음으로 Identity type에서 Email address를 선택한 후에 추가할 이메일 주소를 입력하고 생성한다.

![img_11.png](img_11.png)

검증이 필요하다

다음으로, 검증용으로 작성한 메일의 클라이언트를 열어서 전송된 검증용 메일을 찾는다.

메일 내용에 따라 확인용 URL을 클릭한다.

SES 데시보드에 Verified Identities 화면으로 오면 검증용 메일 주소가 추가된 것을 확인할 수 있다.

![img_12.png](img_12.png)

---

## 11.3.4 관리콘솔에서 테스트 메일 송신
이제 SES 관리 콘솔에서 테스트 메일을 보낼 준비가 끝났다.

테스트 메일을 보내보자.

### 테스트 메일 송신 순서
먼저 생성한 도메인의 체크박스에 체크한 뒤 테스트 베일 보내기를 클릭한다.

아래 표를 통해 송신 내용을 적고 송신한다.

![img_13.png](img_13.png)

![img_14.png](img_14.png)

![img_15.png](img_15.png)

---

## 11.3.5 SMTP를 이용한 메일 송신
어플리케이션에서 메일을 보내는 방법은 **SES API** 와 **SES SMTP 인터페이스** 가 있다고 했다.

책에서는 일반 메일 서버와 같은 방법으로 구현할 수 있는 Amazon SES SMTP 인터페이스를 이용해본다.

### 송신 순서
SES 대시보드 > Account dashboard > Simple Mail Transfer Protocol settings > create 누른다.

![img_16.png](img_16.png)

SMTP 인증 시 이용할 IAM 사용자를 생성한다.

![img_18.png](img_18.png)

생성 후, 자격 증명 다운로드를 클릭하여 인증 정보를 다운로드 한다. 지금 안하면 다시얻을 수 없다.

다음으로 IAM 사용자를 이용해 SMTP를 경유한 메일 송신을 테스트한다.

테스트용 프로그램은 vim 에디터를 이용하여 파이썬으로 작성하였다.

아래 표를 참고하여 작성한다.

![img_19.png](img_19.png)

![img_20.png](img_20.png)

ssh로 각 웹 서버에 접속 후 파이썬 아래 명령어를 실행한다.

~~~
python sendmailtest.py
~~~

~~~
# -*- coding: utf8 -*-
from email import header
import stmplib
from email.mime.text import MIMEText
from email.header import Header
from email import charset

# 각종 정보
account = 'AKIAQU2HMWAJQYPIAPM2'
password = 'BDfS/yGw7hjqbM05GYbuTwyx/PYMNEcFifizmxHUJ2Ll'
server = 'email-smtp.us-east-1.amazonaws.com'
from_addr = 'no_reply@tlsdnjsgh.net'
to_addr = 'wonho0604@gmail.com'

# SMTP 서버에 연결
con = smtplib.SMTP_SSL(server ,465)
con.login(account, password)

# 송신할 메일 메세지 생성
cset = 'utf-8'
message = MIMEText(u'SMTP 테스트', 'plain', cset)
message['Subject'] = Header(u'SMTP를 경유한 전자메일 송신 테스트', cset)
message['From'] = from_addr
message['To'] = to_addr

# 메일 송신
con.sendmail(from_addr, [to_addr], message.as_string())

# SMTP 연결 끊기
con.close()
~~~

![img_21.png](img_21.png)

![img_22.png](img_22.png)

---

## 11.3.6 메일 수신
메일 수신을 위한 설정을 한다.

메일을 받으면 수신할 때의 액션을 지정할 수 있다.

여기서는 수신한 메일을 S3에 저장한다.

미리 SES에서 쓰기 가능한 S3버킷을 준비한다. 여기서는 'aws-intro-sample-mailbox' 라는 s3를 미리 만들었다고 하고 한다.

![img_23.png](img_23.png)

![img_24.png](img_24.png)

### 수신 규칙 설정
규칙 세부 정보를 설정한다.

규칙이름에는 'sample-rule-inquiry'를 입력한다.

---

### 수신 가능 메일 주소 설정
수신용 메일 주소를 설정한다. 단, 메일 주소 도메인은 SES에 등록한 도메인 이름과 같아야 한다.

![img_25.png](img_25.png)

![img_26.png](img_26.png)

![img_27.png](img_27.png)

>
> Principal은 AWS Identity and Access Management (IAM) 정책에서 사용되는 요소 중 하나로,
> 
> 특정 작업을 수행할 주체를 지정합니다. 즉, 누가 특정 작업을 수행할 수 있는지를 나타냅니다.
>
>Principal은 주로 다음 중 하나로 정의됩니다:
>
>AWS 계정 ID: 다른 AWS 계정에 대한 작업을 허용하려면 Principal에 해당 계정의 ID를 지정합니다.
>
>AWS 서비스: AWS 서비스가 다른 AWS 서비스에 대한 권한을 부여하려는 경우에 사용됩니다. 이 경우 Service 키에 서비스 이름이 명시됩니다.
>
>AWS IAM 사용자, 역할 또는 그룹: 특정 IAM 엔터티(사용자, 역할 또는 그룹)에 대한 권한을 부여하려는 경우 
> 
> Principal에 해당 엔터티의 Amazon Resource Name (ARN)을 지정합니다.
>
>Principal을 사용하여 정책을 작성할 때, 특정 작업에 대한 권한을 누가 가지고 있는지를 명시적으로 지정하는 것이 중요합니다.
> 
> 앞서 제공한 S3 버킷 정책에서 Principal은 Service 키에 "ses.amazonaws.com"로 설정되어 있어, 해당 정책은 SES 서비스에게 S3 버킷에 대한 권한을 부여하고 있습니다.
> 

이렇게 하고 메일을 보면 액션에 의해 수신 하였을때 s3 버킷에 저장된다.

![img_28.png](img_28.png)

![img_29.png](img_29.png)

---

## 11.3.7 샌드박스 외부로 이동 
이제 메일 서버를 Amazon SES 외부로 이동 시켜야 한다.

### 이동 순서
대시보드 > 설정 시작 > 프로덕션 엑세스 요청 버튼 클릭

![img_30.png](img_30.png)

아래와 같은 정보로 기입한다.

![img_31.png](img_31.png)



