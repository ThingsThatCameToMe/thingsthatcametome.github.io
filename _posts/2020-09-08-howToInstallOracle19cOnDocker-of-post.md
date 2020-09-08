---
title: "도커에 오라클 19c 설치하기"
date: 2020-09-08 21:30:00 -0000
categories: 도커
---
오라클 19c 도커 설치 관련...

* 주의 사항 
도커 이미지 빌드 작업은 가상화가 아닌 PC에서 진행할것.. 이유는 정확히 모르겠지만 네트워크 이슈로 인해서 정상적으로 이미지 생성이 안됨... 
아래의 경우도 PC에서 이미지 생성 작업 완료 후 가상화 서버에 도커 이미지를 올려서 진행하였음..

* 준비 작업 
작업 PC에 Docker 환경 구성
  - 각 환경 구성 방법은 공식 홈페이지보다 인터넷에서 검색하는것이 더 정확함.. centos8, ubuntu 20.04 둘다 인터넷 검색해서 설치한게 맞았음..
  - ubuntu 참고 : https://linuxhint.com/install_configure_docker_ubuntu/
  - centos8 참고 : 나중에 다시 설치하면 찾아둘것...
작업 PC에 git 환경 구
  - sudo yum install git
  - sudo apt install git
오라클 설치 파일 다운로드
  - 아래 경로에서 받으면 되는데 오라클 계정이 있어야함.
  - https://www.oracle.com/database/technologies/oracle-database-software-downloads.html
도커 이미지 생성 프로젝트 클론
  - 특정집단인거 같은데 오라클이 도커허브에 이미지 못올리게 하니까 이미지 생성하는 걸 자동화 해버림..
  - git clone https://github.com/oracle/docker-images

* 이미지화하기 
1. clone한 프로젝트에서 아래 경로로 이동.
  - 오라클에 대해 잘모르기 때문에 하나의 인스턴스만 구성하는것을 목표로함.
  - cd docker-images/OracleDatabase/SingleInstance/dockerfiles
2. 해당 경로의 버전 디렉토리 내부에 오라클 설치파일을 압축을 풀지 않고 넣는다.
  - dockerfiles/<version>/ 경로에 넣는다.
  - 이것저것 많아서 뭘 받아야 할지 모르겠으면 클론한 저장소의 버전 디렉토리 내부의 checksum파일을 열어서 받아야할 파일명을 확인해본다.
3. docker build 자동화 파일 실행
 - buildDockerImage.sh 스크립트를 실행하는데 설치할 버전을 -v옵션으로 넣어주고 -e, -s, -x 중 하나를 이미지화하는 제품 형태도 정해 준다.
 - sudo ./dockerfiles/buildDockerImage.sh -v 19.3.0 -s
4. 내부 도커허브에 올리거나 이미지를 tar파일로 추출한다. (생성한 위치에서 바로 사용한다면 아래의 작업은 필요 없음.)
 - 허브가 없어서 허브에 올리는건 나중에..
 - 이미지 확인 sudo docker images
 - 이미지 추출 sudo docker save -o oracle_database_19_3_0_se2.tar oracle/database:19.3.0-se2
 - 파라미터는 -o옵션으로 추출되는 파일명을 정해주고 대상 이미지 이름을 넣어준다.
 - 추출한 파일의 권한을 변경 sudo chown wschoi:wschoi oracle_database_19_3_0_se2.tar

* 컨테이너화하기
0. 이미지를 로드한다.(생성한 위치에서 바로 사용하는 중이라면 아래의 작업은 필요 없음.)
 - 가상화 서버에도 docker환경이 구성되어 있어야한다.
 - 내부 도커 허브든 ftp든 뭐든 사용해서 이미지 파일을 받아온다.
 - sudo docker load -i oracle_database_19_3_0_se2.tar 명령어로 가상화OS의 도커에 이미지를 올린다.
1. 도커 실행
 - 데이터 저장 경로 별도 준비 (ex. mkdir -p ~/OracleData/19c/oradata)
 - sudo doocker run --name oracledb-19c -p 1521:1521 -p 5500:5500 -e ORACLE_PWD=1 -e ORACLE_EDITION=standard -v ~/OracleData/19c/oradata:/opt/oracle/oradata oracle/database:19.3.0-se2
 - 여기 참고한거임.. https://github.com/oracle/docker-images/blob/master/OracleDatabase/SingleInstance/README.md
2. 접속 
 - 방화벽 따로 관리할거 아니면 내려둘것.. 
 - sudo systemctl disable --now firewalld(사실 관리하는게 맞음.. 넘 어려워 ㅠㅠ)
 - https://localhost:5500/em/ 자원관리? 같은 화면이 뜸. 로그인은 sys/1로 가능..(위에대로 생성했다면..)
 - sudo docker exec -it oracledb-19c sqlplus sys/1@//localhost:1521/ORCLCDB as sysdba
3. 가상화 또는 host OS 재기동해도 도커의 오라클 기동되도록 설정
 - sudo systemctl enable --now docker(요거로 일단 도커 자체는 기동 되도록..)
 - cd /etc/systemd/system
 - sudo vi [설정한 서비스].service

[Unit]
Wants=docker.service
After=docker.service
 
[Service]
RemainAfterExit=yes
ExecStart=/usr/bin/docker start [실행할 docker container 이름]
ExecStop=/usr/bin/docker stop [실행할 docker container 이름]
 
[Install]
WantedBy=multi-user.target

 - systemctl start [설정한 서비스] → 서비스를 시작
 - systemctl enable [설정한 서비스] → 부팅시 실행할 수 있도록 해당 서비스 활성화
 - 여기 참고 https://help.iwinv.kr/manual/read.html?idx=572
