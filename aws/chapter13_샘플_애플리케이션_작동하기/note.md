![img_1.png](img_1.png)
![img_2.png](img_2.png)
#목차
# 13.1 인프라스트럭처에 배치할 애플리케이션
AWS는 특정 프레임워크에 의존하지 않으므로 어떤 애플리케이션이라도 작동할 수 있다.

이번에는 유명한 루비 온 레일즈의 튜토리얼 사이트를 AWS에서 작동하도록 개선해보겠다.

앞으로 살펴볼 애플리케이션은 아래 기능을 가진 간단한 SNS 사이트이다.

• 사용자 등록

• 짧은 문장과 이미지 게시

• 다른 사용자 팔로잉

![img_3.png](img_3.png)
![img_4.png](img_4.png)
![img_5.png](img_5.png)

---

# 13.2 인프라 구성 확인하기
샘플 어플리케이션 작동을 위한 구성은 크게 두 가지이다.

1. 인프라스트럭처
2. 미들웨어

인프라는 AWS 서비스를 조합하고 미들웨어는 웹 서버에 도입하는 다양한 소프트웨어에 관한 것이다.

---

### 13.2.1 인프라스트럭처
아래는 이 애플리케이션을 배포하는 인프라스트럭처 구성이다.

![img_6.png](img_6.png)

#### 리소스 관리 (IAM)
3장과 동일하다

----
#### 네트워크(vpc)
하나의 VPC 안에 퍼블릭 서브넷과 프라이빗 서브넷을 두 개씩 제공하고

일반 사용자로부터 웹 요청을, 관리자로부터 SSH 연결 요청을 받기위해 2개의 보안 그룹을 생성한다.

인터넷에서 VPC 내부로의 접근과 외부로의 접근이 가능하도록 인터넷 게이트웨이와 NAT 게이트웨이를 만들고 VPC 안에 리소스 끼리 통신할 수 있도록

라우팅 테이블을 설정한다.

---
#### 웹 서버(EC2)
EC2 인스턴스는 3개를 제공한다. 1개는 점프서버이고 애플리케이션 관리자가 외부로부터 연결하기 위한 입구가 되는 서버로, 퍼블릭 서브넷에 제공한다.

나머지 2개는 웁 서버 이고 애플리케이션 사용자가 접근하는 샘플 애플리케이션을 작동하는 서버이며, 2개의 프라이빗 서브넷에 분산하여 배치한다.

---
#### 로드 벨런서(ec2)
LB는 사용자의 요청을 받아서 분산한다. 1개 생성한다.

---
#### 데이터베이스 서버 RDB
VPC private 영역에 생성하고 1개로 보이지만 실제로는 멀티 AZ 등을 이용해 여러 가용영역에 구성된다.

---
#### 이미지 저장 스토리지 (S3)
이미지를 저장하는 S3는 VPC 외부에 생성된다.

---
#### 도메인(Route53)
시스템의 도메인을 관리하는 퍼블릭과 VPC 내부의 서버에 붙인 이름을 관리한다.

---
# 메일 서버(Amazon ses)
VPC 외부에 위치힌다.

---
# 캐시서버 (일래스틱캐시)
VPC 내부에 생성되며 RDS와 마찬가지로 실제로는 클러스터를 사용하여 여러 가용영역 안에 생성된다.

---

### 13.2.2 미들웨어
미들웨어는 웹 서버에서 작동한다.

루비 온 레일즈 자체도 웹 서버로서의 기능을 갖지만, 더 효율적인 많은 사람의 요청을 받을 수 있도록 로드 벨런서와 루비온 레일즈 사이에 엔진엑스 웹 서버를

제공한다.

![img_7.png](img_7.png)

---

# 13.3 샘플 어플리케이션 설치하기
샘플 어플리케이션을 웹 서버에 설치한다. 순서는 아래와 같다.

1. OS 설정과 미들웨어 설치 (엔진엑스)
2. 루비 온 레일즈 환경 구축
3. 샘플 어플리케이션 설치

이 설정은 각각 웹 서버 두 개인 web01 과 web02에 각각 수행해야 한다. 

---
### 13.3.1 OS 설정과 미들웨어 설치

OS 관리자 권한을 가진 ec2-user로 수행한다.

![img_8.png](img_8.png)

#### 미들웨어 설치

우선 ssh로 두 서버에 연결 한 후에 엔진엑스 본체와 루비 온 레일즈를 작동하는 데 필요한 미들웨어를 설치한다.
~~~
sudo yum -y install git gcc-c++ glibc-headers openssl-devel readline libyaml-devel readline-devel zlib zlib-devel libffi-devel libxml2 libxslt libxml2-devel libxslt-devel
sqlite-devel libcurl-devel mysql mysql-devel ImageMagick

sudo amazon-linux-extras install -y nginx1
~~~

---

#### 엔진엑스 설정
엔진엑스가 샘플 어플리케이션을 실행하는 루비 온 레일즈와 연동 되도록한다.

우선 아래 내용을 /etc/nginx/cnf.d/rails.conf 파일에 저장해야하고 이때 관리자 권한이 필요하다.

~~~
sudo vim /etc/nginx/conf.d/rails.conf
~~~

~~~
upstream puma {
# puma 설정으로 지정한 Socket 파일 지정
server unix:///var//www/aws-intro-sample/tmp/sockets/puma.sock;
}
server 1
# 엔진엑스가 리스닝할 포트 설정
listen 3000 default_server; listen [::]:300 default_ server;
server_name puma;
location ~^/assets/ {
root /var/www/aws-intro-sample/public;
}
location / {
proxy_read_timeout 300; proxy_connect_timeout 300;
proxy_redirect off;
proxy_set_header Host Shost;
proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#위 server_name에서 설정한 이름으로 지정
proxy_pass http://puma;}}
~~~

---

#### deploy 사용자 생성
delploy 사용자를 생성한다. deploy 사용자는 샘플 어플리케이션을 작동하는 권한을 가진 일반 사용자로 한다.

ec2-user 사용자는 관리자에 가까운 권한을 가지므로, 보통 지금처럼 애플리케이션을 작동하는 권한을 가진 일반 사용자를 생성한다.

~~~
sudo adduser deploy
~~~

---

#### 어플리케이션을 작동시킬 디렉토리
마지막으로 샘플 어플리케이션을 작동할 디렉토리를 생성한다.

이 디렉토리는 deploy 사용자로 조작하므로 chown 명령어를 이용하여 디렉토리 권한도 변경한다.

~~~
sudo mkdir -p /var/www
sudo chown deploy:deploy /var/www
~~~

이것으로 미들웨어 설치를 완료했다.

---

### 13.3.2 루비온 레일즈 환경 구축

#### deploy 사용자로 전환

먼저 사용자를 전환한다.

~~~
sudo su - deploy
~~~

---

#### 루비 설치

샘플 어플리케이션은 루비 온 레일즈로 만들어져 있으므로 루비를 설치하고 여기서는 rbenv rbenv는 여러 버전의 루비를 설치 할 수 있는 소프트웨이이다.

~~~
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
source ~/.bash_profile
~~~

다음은 rbenv를 이용해 루비를 설치한다 5~10분 걸린다.

/home/deploy/.rbenv/bin/rbenv
~~~
rbenv install 2.6.6
rbenv global 2.6.6
~~~

---

### 13.3.3 샘플 애플리케이션 설치
샘플은 https://github.com/moseskim/aws-intro-sample 에서 제공한다.

#### DB와 사용자 생성 (web01 only)
먼저 RDS상에 샘플 어플리케이션이 이용할 DB와 사용자를 준비한다.

mysql 명령어를 실행한 뒤 8.7.2에서 각자 설정한 마스터 암호를 입력하여 mysql 서버에 연결한다.

성공 후 DB와 사용자를 생성한다.

** 여기에서 결정한 비밀번호는 뒤에서 샘플 어플리케이션 설정 시 사용하므로 잘 기억해둔다** 

그리고 mysql 명령어 뒤에 ; 를 붙이는 것에 주의 한다.

이 순서는 RDS를 업데이트 하기 위한 것이므로 web01 에서만 수행한다.

~~~
mysql -u admin -p -h db.home
(암호 입력) 62seUu13
create database sample_app;
create user sample_app identified by '비밀번호 입력';
grant all privileges on sample_app.* to sample_app@'%';
quit

~~~

---
#### 샘플 에플리케이션 다운로드
~~~
cd /var/www
git clone https://github.com/moseskim/aws-intro-sample.git
~~~

---

#### 루비 라이브러리 설치
샘플 어플리케이션에 필요한 루비 라이브러리(gem) 을 설치한다.
~~~
안돼서 아래 두 명렁어 검색해서 실행함
sudo yum install libcurl-devel (sudo su에서 함)
gem install ovirt-engine-sdk -v '4.4.0' --source 'https://rubygems.org/'

cd aws-intro-sample/
bundle install

~~~

---

#### 시크릿 키 생성 [web01 only]
루비 온 레일즈 보안을 보증하기 위해 필요한 무작위 값을 생성한다.

출력물은 저장한다.
~~~
rails secret

f07a8eb0d69c41f94f138e88ff32e8de1b6c6ea7eda5bfaae3c755dbd345b5346ca60272414214d1d7261cf614a766420b35c7b9c0120223bc68f0c44cb35f98
~~~

> web01과 web02에서 동일한 시크릿 키를 쓰면 된다.

---

#### 샘플 애플리케이션 설정
deploy 사용자 홈 디렉토리에 지정된 .bash_profile 파일에 입력한다.

![img_9.png](img_9.png)

# 샘플 애플리케이션용 설정
~~~
export SECRET_KEY_BASE=f07a8eb0d69c41f94f138e88ff32e8de1b6c6ea7eda5bfaae3c755dbd345b5346ca60272414214d1d7261cf614a766420b35c7b9c0120223bc68f0c44cb35f98
export AWS_INTRO_SAMPLE_DATABASE_PASSWORD=62seUu13
export ANS_INTRO_SAMPLE_HOST=www.tlsdnjsgh.net
export ANS_INTRO_SAMPLE_S3_REGION=ap-northeast-2
export ANS_INTRO_SAMPLE_S3_BUCKET=aws-intro-wonhoshin-upload
export ANS_INTRO_SAMPLE_REDIS_ADDRESS=wonho-elasticache-02.ye8cw2.clustercfg.use1.cache.amazonaws.com:6379
export AWS_INTRO_SAMPLE_SMTP_DOMAIN=tlsdnjsgh.net
export ANS_INTRO_SAMPLE_SMTP_ADDRESS=email-smtp.us-east-1.amazonaws.com
export AWS_INTRO_SAMPLE_SMTP_USERNAME=AKIAQU2HMWAJQYPIAPM2
export ANS_INTRO_SAMPLE_SMTP_PASSWORD=BDfS/yGw7hjqbM05GYbuTwyx/PYMNEcFifizmxHUJ2Ll
~~~

위 내용을 입력했으면 해당 파일을 반영한다.

~~~
source ~/.bash_profile
~~~

deploy 사용자의 연결을 끊었다가 재연결하면 자동으로 .bash_profile의 내용이 반영되므로 source 명령어를 실행하지 않아도 된다.

---

#### 테이블 생성[web01 only]
내려받은 샘플 어플리케이션에는 DB 테이블을 생성하는 설정이 포함된다 해당 내용을 반영한다.
~~~
rails db:migrate RAILS_ENV=production
~~~

---

#### 샘플 어플리케이션 실행

잘 되는지 보자

##### 사용자 전환
우선 원래 사용자로 돌아간다

~~~
exit
~~~

##### 변경한 설정 갱신
변경 내용 갱신을 위해 엔진엑스를 재기동 한다.

~~~
sudo systemctl restart nginx.service
~~~
---

#### 샘플 실행

다음으로 deploy 사용자로 전환 한 후에 샘플 어플리케이션을 실행한다.

디렉토리는 샘플 어플리케이션 코드를 가진 위치이다.

~~~
sudo su - deploy
cd /var/www/aws-intro-sample/
rails assets:precompile RAILS_ENV=production
rails server -e production
~~~

작동 확인하기에서 실패 하여 IAM 엑세스 키 / 시크릿 키 생성 후에 (AKIAQU2HMWAJVIIFVWHR/KNVMxpfTk5h5Jps+HWLWXNRXXbb8ocq1X372QAoN)

위 코드 다시수행 후 성공

~/www/config/lnitizalizers/carrier_wave.rb 파일에 설정
~~~
# TO-BE
if Rails.env.production?
  CarrierWave.configure do |config|
    config.fog_provider = 'fog/aws'                   # required
    config.fog_credentials = {
      provider:              'AWS',                   # required
      aws_access_key_id:     'AKIAQU2HMWAJVIIFVWHR',  # required unless using use_iam_profile
      aws_secret_access_key: 'KNVMxpfTk5h5Jps', # required unless using use_iam_profile
      region:                'ap-northeast-2',        # optional, defaults to 'us-east 
    }
    config.fog_directory  = 'wonho-aws-intro' # required
  end
end
~~~

---

# 13.4 작동 확인하기

## 샘플 어플리케이션에 연결

앞서 생성한 도메인 이름 (AWS_INTRO_SAMPLE_SMTP_DOMAIN)에 설정한 값으로 접근해본다.



