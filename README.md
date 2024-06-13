<h6>** 공유 목적으로 작성된 내용으로 누구나 따라서 진행할 수 있도록 상세히 기재 **</h6>

**[ Docker 설치 및 설정 ]**
1. Local 환경
   - 작성중...

2. Cloud 환경 AWS EC2 기준
   <h6>** 참고 : 2023년 이전에는 amazon-linux-extras 설치 후 amazon-linux-extras를 통해 docker를 설치 했었으나,<br>
   2023년부터 해당 패키지는 없어졌고 yum을 통해 docker 바로 설치 가능해짐<br>
   (공식 문서 - https://docs.aws.amazon.com/ko_kr/serverless-application-model/latest/developerguide/install-docker.html) **</h6>

   1. Amazon Linux IAM 2022년 이하 버전에서 docker 설치
      ~~~
      # yum 패키지 업데이트
      $ sudo yum update -y
   
      # amazon-linux-extras 패키지 설치
      $ sudo yum install -y amazon-linux-extras
   
      # docker 설치
      $ sudo amazon-linux-extras install docker -y
   
      # docker 상태/버전 확인
      $ sudo systemctl status docker
      $ docker --version
   
      # docker 데몬 시작, 부팅 시 자동 재시작
      $ sudo systemctl start docker
      $ sudo systemctl enable docker
      ~~~
   3. Amazon Linux IAM 2023년 이상 버전에서 docker 설치
      ~~~
      # yum 패키지 업데이트
      $ sudo yum update -y
   
      # docker 설치
      $ sudo yum install -y docker
   
      # docker 설치/상태/버전 확인
      $ rpm -qa | grep docker
      $ sudo systemctl status docker
      $ docker --version
   
      # docker 데몬 시작, 부팅 시 자동 재시작
      $ sudo systemctl start docker
      $ sudo systemctl enable docker
      ~~~

<br>

**[ AWS EC2 인스턴스 생성 ]**
   - swarm 구성을 위한 서버 생성으로 3개의 서버를 동일하게 설정

<br>

**[ Docker Swarm 생성 ]**

<br>

**[ Docker Swarm 구성 ]**

