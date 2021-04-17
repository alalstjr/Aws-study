--------------------
# Aws-study
--------------------

# 목차

- [AWS FAQ](#AWS-FAQ)
- [서버리스 웹 호스팅과 CloudFront로 웹 가속화 구성하기](#서버리스-웹-호스팅과-CloudFront로-웹-가속화-구성하기)
    - [아키텍처에 구현할 기술](#아키텍처에-구현할-기술)

# AWS FAQ

- Amazon S3
  - FAQ : https://aws.amazon.com/ko/s3/faqs/?nc=sn&loc=7
- Amazon CloudFront
  - FAQ : https://aws.amazon.com/ko/cloudfront/faqs/?nc=sn&loc=5&dn=2
- Amazon EC2
  - FAQ : https://aws.amazon.com/ko/ec2/faqs/
- Elastic Load Balancer
  - FAQ : https://aws.amazon.com/ko/elasticloadbalancing/faqs/?nc=sn&loc=6
- Amazon VPC
  - FAQ : https://aws.amazon.com/ko/vpc/faqs/
- Amazon RDS
  - FAQ : https://aws.amazon.com/ko/rds/faqs/
- Auto Scaling
  - FAQ : https://aws.amazon.com/ko/autoscaling/faqs/
- IAM User
    - User 생성 가이드 : https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/id_users_create.html

# 서버리스 웹 호스팅과 CloudFront로 웹 가속화 구성하기

##  아키텍처에 구현할 기술

서버가 없이도 구성이 가능한 정적 웹 호스팅을 만들고, 웹 속도를 높이기 위하여 콘텐츠 전송 네트워크(CDN) 서비스를 연동합니다.

- 필요한 서비스
    - Amazon S3
    - Amazon CloudFront

## S3

Html, 영상, 기타 컨텐츠를 업로드하여 웹 호스팅 설정을 하게되면 웹 사이트처럼 작동할 수 있습니다.  
기본적으로 S3는 인터넷용 Storage 이며 웹 서비스 인터페이스를 사용해서 웹에서 언제든지 어디서나 원하는 양의  
데이터를 저장하고 검색할 수 있기 때문에 S3 만으로도 웹 호스팅이 가능하지만 웹 브라우저에서 읽어야 하는 컨텐츠  
크기가 커지면 그만큼 로딩이 지연되는 문제가 발생합니다.  
이러한 문제를 해결할 수 있는 것이 바로 `컨텐츠 전송 네트워크 서비스(CloudFront)` 입니다.

### S3 생성 후 살펴보기

1. 생성한 버킷
2. 메뉴 - 속성
    - 정적 웹 사이트 호스팅 - 편집 
    - 상태 - 활성화
    - 인덱스 문서 - 웹 사이트의 홈 페이지 또는 기본 페이지를 지정합니다.
        - index.html 작성하여 시작시 가장 처음 읽는 파일을 설정합니다.
3. 메뉴 - 권한
    - 버킷 정책 - 편집
    - 정책 생성기
        - Select Type of Policy : S3 Bucket Policy
    - 권한적인 부분(Add Statement) 설정
        - Principal : [ * ] 입력하여 모든 서비스를 적용한다고 명시합니다.
        - Actions : 어떠한 권한을 적용할것인지 설정합니다.
        - Amazon Resource Name (ARN) : 버킷 ARN 복사 붙여넣기하여 적용하기 (img_01)
        - Add Statement -> Generate Policy 클릭하여 생성된 정책을 복사합니다. (img_02)
    - 정책 편집기 - 정책 수정 (img_02)
        - 이렇게 되면 컨텐츠를 버킷에 올리더라도 웹 호스팅이 되지 않게 됩니다.
        - Resource : [ /* ] 문자를 추가하여 버킷 안에있는 모든 컨텐츠는 웹으로 나가는 것을 허용한다고 명시합니다. 
            - "Resource": "arn:aws:s3:::jjunpro-site/*",
    
![S3 권한 설정](./images/img_01.png)
![S3 권한 설정](./images/img_02.png)
![S3 권한 설정](./images/img_03.png)

- 4. html 업로드
    - html 파일 클릭 후 객체 URL 클릭 하면 서버없이 웹 호스팅이 가능합니다.

![S3 웹 호스팅](./images/img_04.png)

## CloudFront

컨텐츠를 빠르게 읽을 수 있도록 캐싱 기능을 제공하므로 더욱 가속화 된 웹을 제공할 수 있습니다.  

https://console.aws.amazon.com/cloudfront/home?region=ap-northeast-2#

1. Create Distribution -> Web Get Started
2. Distribution Settings - Price Class
    - 특정 지역에만 캐싱을 하겠다고 설정합니다.
        - Use Only U.S., Canada and Europe - 특정 지역
        - Use U.S., Canada, Europe, Asia, Middle East and Africa - 특정 지역
        - Use All Edge Locations (Best Performance) - 모든 지역
3. CloudFront에서 Create Distribution 클릭 후 Deploy가 되기까지는 약 5분~10분정도의 긴 시간이 소요
4. CloudFront 링크테스트
    - html 에서 불러오는 이미지의 경로를 CloudFront 도메인 으로 변경합니다.
    - 그리고 S3 버킷에 업로드 합니다.

~~~
<html>
<head>My CloudFront Test</head>
<body>
<p>My text content goes here.</p>
<p><img src="https://<CloudFront 도메인>/awslogo.png" alt="AWS LOGO"/>
<p><img src="https://d1ut0p6g43vawo.cloudfront.net/awslogo.png" alt="AWS LOGO"/>
</body>
</html>
~~~ 

5. 브라우저 테스트
    - https://d1ut0p6g43vawo.cloudfront.net/index3.html
    - x-cache: Hit from cloudfront 
        정상적으로 cloudfront 로 캐싱된 상태임을 확인할 수 있습니다. 
    
![S3 웹 호스팅](./images/img_05.png) 

# EC2-LAMP-ELB 구성하기

##  아키텍처에 구현할 기술

Linux 기반의 가상 서버에 Apache 웹 서버, MySQL 데이터베이스, php 어플리케이션을 구축하고 로드 밸런서를 이용하여 이중화 구성을 만듭니다.

- 필요 AWS 서비스
    - Amazon Elastic Compute Cloud(EC2)
    - Amazon Virtual Private cloud(VPC)
    - Elastic Load Balancing / Applicaion Load Balancer
    
서버를 생성하여 `Internet Gateway` 통하여 외부와 통신하게 됩니다.  
이런 단일 구성만으로도 시스템이 돌아가는데 문제는 없지만 실제로 운영되는 웹 서비스들은  
트래픽 증가 장애등과 같은 이슈가 발생할 가능성이 높은데  
이런 문제를 해결하기 위해서 여러개의 서버를 만들고  
네트워크 트래픽 분산 서비스의 `Elastic Load Balancing` 에 연결하여  
상황에 따라 시스템이 원활하게 돌아가도록 해줍니다.  
여기서 ELB 유형중에 `Applicaion Load Balancer` 을 사용합니다.

- 간단한 로직 표현
    - internet
        - Internet Gateway
            - ELB-ALB
                - EC2 서버
                - EC2 서버

## 1. amazon linux 2에 LAMP 웹 서버 설치하기
    - LAMP 서버 설치 및 테스트
        - EC2 생성 시 User Data 스크립트 추가하여 자동으로 설치
        - LAMP 서버 테스트
    - Custom AMI 생성
    - Custom AMI 로 두 번째 LAMP 서버 생성
    - PuTTY로 SSH 접속하여 데이터베이스 보안 설정

- EC2 생성 및 LAMP 웹 서버 설치
    - LAMP 웹 서버 설치를 위한 User Data 스크립트는 EC2가 생성되는 과정에서 Apache 웹 서버, MySQL 데이터베이스, PHP 어플리케이션이 설치될 수 있게 해줍니다.
    - 스크립트의 세부 내용은 아래와 같으며, EC2 생성 단계 중 User Data에 아래 내용을 복사하여 붙여넣으셔도 동일한 LAMP 웹 서버가 설치됩니다.

- [PuTTY를 이용하여 EC2에 접속]
    - Windows 환경에서 실습을 진행하시는 분들은 EC2에 SSH 접속을 하기 위해서 PuTTY라는 프로그램이 필요합니다. 이와 관련하여 아래 내용을 참고하여 주시기 바랍니다.
    - PuTTY 다운로드 페이지 : https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
    - Mac 환경에서 실습을 진행하시는 분들은 기본 터미널 프로그램 또는 iTerm과 같은 프로그램을 사용하시면 됩니다.
- [Custom AMI(Amazon Machine Image) 생성 및 Custom AMI를 통한 EC2 생성]
    - AMI 생성에서 Create Image 클릭 후 Image 생성까지는 약 2분~3분정도의 시간이 소요되며, 원활한 실습 영상을 위하여 대기시간은 편집하였습니다.

## 2. Application Load Balancer 시작하기

1. Load Balancer 유형 선택
2. Load Balancer 및 리스너 구성
3. Load Balancer 에 대한 보안 그룹 구성
4. 대상 그룹 구성
5. 대상 그룹에 대상 등록
6. Load Balancer 생성 및 테스트
7. Load Balancer 삭제 선택 사항

## 3. 설치 진행하기

- https://ap-northeast-2.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-2  
- EC2 인스턴스 시작 버튼을 클릭합니다.  
    - 1. AMI 선택Amazon Linux 2 AMI (HVM), SSD Volume Type
    - 2. 인스턴스 유형 선택 : t2.micro
    - 3. 인스턴스 세부 정보 구성
        - 서브넷 : ap-northeast-2a 선택
        - 퍼블릭 IP 자동 할당 : 자동을 공인 IP 를 받도록 설정 선택
        - 고급 세부 정보 
            - 사용자 데이터 : 시작 시 인스턴스를 구성하거나 구성 스크립트를 실행할 때 사용할 사용자 데이터를 지정할 수 있습니다.
                - #include https://bit.ly/Userdata
                - 혹은 아래 코드를 입력
    - 5. 태그 추가 : 어떤 용도의 EC2 인지 구분짓는 꼬리표입니다.
        - Key : Name
        - Value : lab-web-public-2a
            - 웹으로 사용하는 공개한 서브넷 서버의 2a
    - 6. 보안 그룹 구성 : aws 에서 기본으로 제공해주는 방화벽
        - 보안 그룹 이름 : lab-web-security
        - 기본적으로 Linux 이므로 SSH 규칙이 기본으로 설정되어 있습니다.
        - 추가로 웹 서버 테스트를 할것이므로 규칙을 추가합니다.
            - HTTP 규칙 추가 
                - 소스 : [ 0.0.0.0/0, ::/0 ] 에서 [ ::/0 ] IPV6 용 이므로 삭제합니다.
    - 7. EC2 웹으로 접근
        - 생성된 ec2 의 public 주소를 접근하면 정상적으로 aws 가동중인 웹페이지가 출력됩니다.
~~~
#!/bin/bash
yum update -y
amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
yum install -y httpd mariadb-server
systemctl start httpd
systemctl enable httpd
usermod -a -G apache ec2-user
chown -R ec2-user:apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
if [ ! -f /var/www/html/bootcamp-app.tar.gz ]; then
cd /var/www/html
wget https://s3.amazonaws.com/immersionday-labs/bootcamp-app.tar
tar xvf bootcamp-app.tar
chown apache:root /var/www/html/rds.conf.php
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
mkdir phpMyAdmin && tar -xvzf phpMyAdmin-latest-all-languages.tar.gz -C phpMyAdmin --strip-components 1
cd /var/www/html/phpMyAdmin/
cp config.sample.inc.php config.inc.php
fi
~~~

## 4. Custom AMI(Amazon Machine Image) 생성 및 Custom AMI를 통한 EC2 생성

![EC2 이미지생성](./images/img_06.png)

- 이미지 생성
    - 이름 : 어떤 용도의 AMI 를 생성하는지 구분 짓기 위해서 작성
        - lab-web-202104
    - 재부팅 안 함 : 해당 EC2 가 정지된 상태에서 이미지 생성을 하겠는가 아니면 러닝되어 있는 상태에서 생성을 하겠는가의 선택
        - 실제 서비스 중인 EC2 에서 재부팅 안함을 선택하고 생성하면 문제가 발생할 수 있음으로 체크해줍니다.
    - 이미지 생성
- EC2 에 2a 로 하나의 공간이 있습니다. 추가로 2c 를 방금 생성한 AMI 로 추가 생성하겠습니다.
- 추가로 생성된 이미지 확인
    - https://ap-northeast-2.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-2#Images:
    - 생성된 이미지 체크 후 시작하기 버튼 클릭
        - 1, 2 동일하게 사용
        - 3. 인스턴스 세부 정보 구성
            - 서브넷 : ap-northeast-2a 선택
            - 고급 세부 정보 
                - 사용자 데이터 : 따로 작성할 필요 없습니다. 이미 복사된 AMI 에 추가되어 있기 때문입니다.
        - 5. 태그 추가 : 어떤 용도의 EC2 인지 구분짓는 꼬리표입니다.
                - Key : Name
                - Value : lab-web-public-2c
                    - 웹으로 사용하는 공개한 서브넷 서버의 2c
        - 6. 보안 그룹 구성 : 기존에 생성한 web 용 보안 그룹을 사용합니다.
            - 보안 그룹 이름 : lab-web-security

## 5. Load Balancer 설정

- https://ap-northeast-2.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-2#LoadBalancers:
- Load Balancer 생성
    - Application Load Balancer
        - 1. Load Balancer 구성
            - 이름 : lab-web-alb
            - 체계 : 웹용의 인터널인지 내부용 인터널인지 설정
            - 리스너 : ELB 가 통과하는 트래픽의 포트번호를 정의
            - 가용 영역 : 2a, 2c
            - 태그
                - Key : Name
                - Value : lab-web-alb
        - 3. 보안 그룹 구성 : ALB 용 보안그룹을 따로 생성합니다.
        - 4. 라우팅 구성 : 어떤 EC2 를 등록하는지 설정합니다.
            - 이름 : lab-web-alb-target
        - 5. 대상 등록 : EC2 를 타겟 그룹으로 넣습니다.
            - 방금전에 생성한 EC2 [ 2a, 2c ] 를 선택합니다.
    - 생성된 ALB 의 DNS 이름
        - EC2 로 바로 접근해서 웹 페이지를 보느것이 아니라 ALB 의 DNS 를 활용해서 생성한 2개의 EC2 웹 화면을 보도록 분산시키겠습니다.
        - 생성된 ALB 도메인 네임을 검색
            - http://lab-web-alb-1384992291.ap-northeast-2.elb.amazonaws.com/
                - 새로고침 할때마다 [ 2a, 2c ] 번갈아 가며 서버가 교체됩니다. 정상적으로 분산되어 있는것을 확인했습니다.
                        
         