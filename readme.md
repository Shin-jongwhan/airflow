### 230901
# airflow 정리 공간
### airflow 는 DAG (Directed Acyclic Graph) 로 endpoint 가 있는 작업의 연결을 해주는 framework 이다.
### apache 재단에서 관리한다. 에어비앤비가 복잡해지는 워크플로우를 관리하기 위해서 시작한 프로젝트이다.
### <br/><br/><br/>

# docker bitnami/airflow 로 airflow 구축하기
### bitnami 에서 관리하는 docker 를 사용한다.
### 5 millon 이상 다운로드 받을 정도로 굉장히 인기가 많은 image 이다.
#### https://hub.docker.com/r/bitnami/airflow
### bitnami 는 여러가지 소프트웨어를 하나로 묶어서 설치할 수 있게 해주는 소프트웨어 패키지이다. 주로 웹서버 구축할 때 사용한다. 
### <br/>

### 위 도커 이미지는 docker-compose 를 이용해서 구축하는 게 편리하다.
#### docker-compose 간단 정리 참고
#### https://github.com/Shin-jongwhan/docker/tree/main/docker-compose#230816
### 아래 주소에서 docker-compose.yml 을 다운로드받을 수 있다.
#### https://raw.githubusercontent.com/bitnami/containers/main/bitnami/airflow/docker-compose.yml
```
# download
curl -sSL https://raw.githubusercontent.com/bitnami/containers/main/bitnami/airflow/docker-compose.yml > docker-compose.yml
```
### <br/>

## docker volume 생성
```
docker volume create jhshin_airflow_postgresql_data
docker volume create jhshin_airflow_redis_data
```
#### 
### <br/>

## yaml 파일 수정
### airflow web server 의 port 는 8080 이 default 허용이다.
### 이는 AIRFLOW_WEBSERVER_PORT_NUMBER=8088 와 같이 env 세팅으로 바꿀 수 있다.
#### 나는 8080 포트가 이미 점유 상태라서, 8088 로 대체하였다.
### volume 에 'driver : local' 은 local 에 위치한 volume 인지, 클라우드에 위치한 volume 인지 명시하는 것이다.
```
# Copyright VMware, Inc.
# SPDX-License-Identifier: APACHE-2.0

version: '2'

services:
  postgresql:
    image: docker.io/bitnami/postgresql:15
    volumes:
      - 'jhshin_airflow_postgresql_data:/bitnami/postgresql'
    environment:
      - POSTGRESQL_DATABASE=bitnami_airflow
      - POSTGRESQL_USERNAME=bn_airflow
      - POSTGRESQL_PASSWORD=bitnami1
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
  redis:
    image: docker.io/bitnami/redis:7.0
    volumes:
      - 'jhshin_airflow_redis_data:/bitnami'
    environment:
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
  airflow-scheduler:
    image: docker.io/bitnami/airflow-scheduler:2
    environment:
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USERNAME=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_WEBSERVER_HOST=airflow
      - AIRFLOW_WEBSERVER_PORT_NUMBER=8088
  airflow-worker:
    image: docker.io/bitnami/airflow-worker:2
    environment:
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USERNAME=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_WEBSERVER_HOST=airflow
      - AIRFLOW_WEBSERVER_PORT_NUMBER=8088
  airflow:
    image: docker.io/bitnami/airflow:2
    environment:
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USERNAME=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_WEBSERVER_PORT_NUMBER=8088
    ports:
      - '8088:8088'
volumes:
  jhshin_airflow_postgresql_data:
    driver: local
  jhshin_airflow_redis_data:
    driver: local
```
### <br/>

### docker-compose 실행
```
docker-compose up -d
```
### docker-compose 종료
```
docker-compose down
```
### <br/>

## airflow 웹서버 실행 확인
### 먼저 telnet 으로 접속 되는지 확인하였다
```
telnet [IP address] [port]
```
### 그 다음 웹으로 접속해보았다.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/617a766d-8677-4be8-b90f-eb9e2e41fc76)
### 기본 접속 id, pw 는 아래와 같다.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/6c5861fc-15c5-4363-b469-e520cfd86ea4)
### 접속 확인
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/78f5393d-25ed-432d-941e-c3c7fd351667)
### <br/>

## 아이디 생성
### docker 환경 접속
- id : jhshin
- pw : System!2
```
# 컨테이너 접속
docker exec -it [container ID] /bin/bash
# 아이디 생성
airflow users create --username jhshin --firstname jonghwan --lastname shin --role Admin --email jhshin@test.com
```
### 접속 확인
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/8c88a843-d344-4b70-bfee-e676256306c1)
#### * docker-compose 를 껐다 켜도 해당 아이디로 접속이 된다. 왜냐면 아이디는 DB 에 저장되어 있고 docker volume 안에 저장된 데이터이기 때문에 container 가 종료되어도 삭제 안 되고 유지된다.
### <br/>

## airflow worker 와 scheduler 에서 에러
### airflow webserver 를 인식할 수 없다고 한다.
### 아마 default port 가 8080 으로 되어 있어서 그런 듯 하다.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/b47b11a5-2c98-4644-aeff-a4aa1e36b205)
### docker compose yaml 파일에 scheduler 와 worker 에도 다음을 추가해주자.
- AIRFLOW_WEBSERVER_PORT_NUMBER=8088
### healthy 확인
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/844747bc-2ad6-4709-ac52-820ab8f7ba3a)
### <br/>

### example DAG 를 한 번 실행해보자.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/8b8ff647-2ed6-41e9-a925-847e2c0bc358)
### 실행하면 왼쪽과 같이 초록색 바가 나타날 것이다.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/dc6b9d2d-76cc-45dc-b04b-5674784936e5)
### <br/>

## time zone 변경
### 여러가지 찾아봤는데 그냥 웹 서버에서 local 로 변경할 수 있다.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/e757ab16-dd90-48af-a628-cc666eb2f133)
### <br/>

## log 에러 처리
### airflow 관련한 docker 모두에 다음을 똑같이 써준다. key 는 모두 같아야 한다.
```
- AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
- AIRFLOW_SECRET_KEY=a25mQ1FHTUh3MnFRSk5KMEIyVVU2YmN0VGRyYTVXY08=
```
### 그럼 이렇게 로그가 조회가 될 것 이다.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/9f0ceadc-0ab5-4a18-a261-de77b5461194)
### <br/>

## DAG 실행 예제
### airflow 는 interactive 하게 현재 진행 중인 상태를 보여준다.
https://github.com/Shin-jongwhan/airflow/assets/62974484/57af569d-4053-4392-b131-dbda8b0d8e09

### <br/><br/><br/>
