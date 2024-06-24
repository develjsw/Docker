<h6>** 공유 목적으로 작성된 내용으로 누구나 따라서 진행할 수 있도록 상세히 기재 **</h6>

**[ Docker 설치 및 설정 ]**
1. Local 환경
   - 작성중...

2. Cloud 환경 AWS EC2 기준
   <h6>** 참고 : 2023년 이전에는 amazon-linux-extras 설치 후 amazon-linux-extras를 통해 docker를 설치 했었으나,<br>
   2023년부터 해당 패키지는 없어졌고 yum을 통해 docker 바로 설치 가능해짐**<br>
   (공식 문서 - https://docs.aws.amazon.com/ko_kr/serverless-application-model/latest/developerguide/install-docker.html)<br>
   </h6>

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
   - swarm 구성을 위한 서버 생성으로 5개의 서버를 동일하게 설정
   - AWS console login → 서비스 → EC2 → 인스턴스 시작
   - 설정 값 입력
      - 이름 설정<br>
        ex) docker-swarm1, docker-swarm2, docker-swarm3, docker-swarm4, docker-swarm5
      - Amazon Linux AMI 사용
      - t2.micro 선택
      - Key Pair 생성(첫번째) / 기존 Key Pair 사용(2~3번째)
      - 보안그룹 생성(첫번째) / 기존 보안그룹 선택(2~3번째)
      - SSH 트래픽 허용 - 위치 무관
      - 스토리지 8GB - SSD gp3

<br>

**[ AWS EC2 방화벽 설정 ]**
- 인바운드/아웃바운드 방화벽 허용 리스트
  - TCP 2377 - 클러스터 관리용 포트
  - TCP/UDP 7946 - 노드 간 통신 포트
  - UDP 4789 - 클러스터에서 사용되는 docker ingress overlay 네트워크 포트
- 방화벽 설정
  - 별도의 아웃바운드 설정은 하지 않아도 됨(디폴트로 전체 허용되어 있으므로)
  - AWS console login → AWS EC2 인스턴스 → 보안 → 보안 그룹 → 인바운드 규칙 → 인바운드 규칙 편집 →
    - 1. 사용자 지정 TCP, 2377, 사용자 지정 0.0.0.0/0
    - 2. 사용자 지정 TCP, 7946, 사용자 지정 0.0.0.0/0
    - 3. 사용자 지정 UDP, 7946, 사용자 지정 0.0.0.0/0
    - 4. 사용자 지정 UDP, 4789, 사용자 지정 0.0.0.0/0
<br>

**[ Docker Swarm 생성 ]**
1. Swarm 클러스터 생성 명령어 실행
   ~~~
   # 현재 노드의 IP주소가 사용되며, Swarm 모드가 활성화되고 클러스터가 생성됨
   # $ sudo docker swarm init
   
   # 현재 노드의 IP주소가 사용되며, Swarm 모드가 활성화되고 클러스터가 생성되고, Swarm 클러스터에 참여하는 다른 노드들이 접근할 수 있는 IP주소 또는 네트워크 인터페이스를 지정함
   # $ sudo docker swarm init --advertise-addr [IP|INTERFACE]
   
   # 현재 노드의 IP주소가 사용되며, Swarm 모드가 활성화되고 클러스터가 생성되고, 다른 노드들이 연결할 수 있는 주소를 지정함. 기본값 - 0.0.0.0:2377
   # $ sudo docker swarm init --advertise-addr --listen-addr [IP:PORT]
   
   $ sudo docker swarm init --advertise-addr 아이피
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
    	<h6>** 참고 : Leader Node에 문제가 생긴 경우 Manger Node에서 자동으로 새로운 Leader Node가 선출됨(고가용성) **</h6>
4. Swarm 활성화 확인
   - Swarm은 Docker에 내장되어 있으므로 docker info 명령어를 통해 정보 확인이 가능
     ~~~
     $ sudo docker info
     ~~~
   - 간략하게 Swarm 활성화 여부만 확인하고 싶다면 아래 명령어를 통해 확인 가능
     ~~~
     $ sudo docker info | grep Swarm
     ~~~
     <h6>** 참고 : Leader/Manager Node에서 사용 가능한 'docker node ls', 'docker service ls'등의 명령어 Worker Node에서는 사용 불가능 **</h6>
<br>

**[ Docker Swarm 구성 ]**
- Worker/Manager Node 클러스터에 추가할 수 있는 명령어 받기 (Leader Node에서 실행)
  - 클러스터에 Worker Node 추가할 수 있는 명령어 받기(3개)
	~~~
	$ sudo docker swarm join-token worker
	
	# 명령어 실행 결과
	To add a worker to this swarm, run the following command:
	
		docker swarm join --token 토큰 아이피:2377
	~~~
  - 클러스터에 Manager Node 추가할 수 있는 명령어 받기(2개)
	~~~
	$ sudo docker swarm join-token manager
	To add a manager to this swarm, run the following command:
		
		docker swarm join --token 토큰 아이피:2377
	~~~
- 클러스터에 Node(Worker/Manager) 추가하기
  - 추가할 서버로 접속 후, 위에서 받은 join 명령어 실행
	  ~~~
	  $ sudo docker swarm join --token 토큰 아이피:2377
	  ~~~
  <h6>** 주의 : Leader Node를 포함하여 Manger Node는 홀수 갯수를 유지해야 함 **</h6>

<br>

**[ Docker Swarm 기타 사항 ]**
- 클러스터에서 현재 node(Leader/Worker/Manager Node) 연결 끊기
  ~~~
  # Worker/Manager Node
  $ sudo docker swarm leave

  # Leader Node (명령어 사용시 주의 필요!)
  $ sudo docker swarm leave --force
  ~~~
- 클러스터에서 node 제거(Leader/Manager Node에서 사용)
  ~~~
  # node의 상태가 Ready인 경우 삭제가 불가능하므로 docker swarm leave 후 진행
  $ sudo docker node rm <노드 ID>
  ~~~
- 노드 상세정보 출력
  ~~~
  # 특정 노드의 상세정보 출력
  $ sudo docker node inspect <노드id|호스트네임> --pretty
  ex) docker node inspect twcctrhyg1iqf4w8tfmlbifin --pretty

  # 현재 노드의 상세정보 출력
  $ sudo docker node inspect self --pretty
  ~~~
- 노드 유형 변경 (promote, demote는 Leader/Manager Node에서 사용 가능)
  ~~~
  # worker node를 manager node로 변경
  $ docker node demote <노드 호스트네임>
  
  # 여러개의 worker node를 manager node로 일괄 변경
  $ docker node demote <노드 호스트네임1> <노드 호스트네임2> <노드 호스트네임3>

  # manager node를 worker node로 변경
  $ docker node promote <노드 호스트네임>

  # 여러개의 manager node를 worker node로 일괄 변경
  $ docker node promote <노드 호스트네임1> <노드 호스트네임2> <노드 호스트네임3>
  ~~~
