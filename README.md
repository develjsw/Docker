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
   - AWS console login → 서비스 → EC2 → 인스턴스 시작
   - 설정 값 입력
      - 이름 설정<br>
        ex) docker-swarm1, docker-swarm2, docker-swarm3
      - Amazon Linux AMI 사용
      - t2.micro 선택
      - Key Pair 생성(첫번째) / 기존 Key Pair 사용(2~3번째)
      - 보안그룹 생성(첫번째) / 기존 보안그룹 선택(2~3번째)
      - SSH 트래픽 허용 - 위치 무관
      - 스토리지 8GB - SSD gp3

<br>

**[ Docker Swarm 생성 ]**
1. Swarm 클러스터 생성 명령어 실행
   ~~~
   # 현재 노드의 IP주소가 사용되며, Swarm 모드가 활성화되고 클러스터가 생성됨
   # $ docker swarm init
   
   # 현재 노드의 IP주소가 사용되며, Swarm 모드가 활성화되고 클러스터가 생성되고, Swarm 클러스터에 참여하는 다른 노드들이 접근할 수 있는 IP주소 또는 네트워크 인터페이스를 지정함
   # $ docker swarm init --advertise-addr [IP|INTERFACE]
   
   # 현재 노드의 IP주소가 사용되며, Swarm 모드가 활성화되고 클러스터가 생성되고, 다른 노드들이 연결할 수 있는 주소를 지정함. 기본값 - 0.0.0.0:2377
   # $ docker swarm init --advertise-addr --listen-addr [IP:PORT]
   
   $ docker swarm init --advertise-addr 아이피
   ~~~

2. Swarm 클러스터 생성 명령어 실행 결과
   1) Swarm 모드 활성화 : 현재 접속중인 서버의 Docker 데몬이 Swarm모드로 전환됨
   2) 초기 리더 노드 설정 : 현재 노드가 Swarm 클러스터의 첫 번째 매니저(리더) 노드로 설정됨
   3) Swarm 클러스터 생성 : 새로운 Swarm 클러스터가 생성됨
   4) 토큰 생성 : 클러스터에 다른 노드를 추가할 수 있는 join 토큰이 생성됨
   ~~~
   Swarm initialized: current node (현재 노드의 ID값) is now a manager.

	To add a worker to this swarm, run the following command:

		docker swarm join --token SWMTKN-1-토큰값 아이피:2377

	To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
   ~~~

<br>

**[ Docker Swarm 구성 ]**

