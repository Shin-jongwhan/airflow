### 230901
# airflow 정리 공간
### airflow 는 DAG (Directed Acyclic Graph) 로 endpoint 가 있는 작업의 연결을 해주는 framework 이다.
### apache 재단에서 관리한다. 에어비앤비가 복잡해지는 워크플로우를 관리하기 위해서 시작한 프로젝트이다.
### 참고 자료
- [Airflow 엄청 자세한 튜토리얼 #왕초심자용](https://velog.io/@clueless_coder/Airflow-%EC%97%84%EC%B2%AD-%EC%9E%90%EC%84%B8%ED%95%9C-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC-%EC%99%95%EC%B4%88%EC%8B%AC%EC%9E%90%EC%9A%A9)
- [bitnami/airflow - docker hub](https://hub.docker.com/r/bitnami/airflow)
- [docker기반 Airflow 2.0 설치](https://burning-dba.tistory.com/127)
### <br/><br/><br/>

## airflow architecture
### airflow 는 다음의 과정으로 분석이 진행된다.
### 만약 error 가 나면 error alert system 을 거친다.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/061a4e44-b7ac-4fd6-b71a-af034f7a666f)
### airflow 의 구성 요소
- A `scheduler`, which handles both triggering scheduled workflows, and submitting Tasks to the executor to run.
- An `executor`, which handles running tasks. In the default Airflow installation, this runs everything inside the scheduler, but most production-suitable executors actually push task execution out to workers.
- A `webserver`, which presents a handy user interface to inspect, trigger and debug the behaviour of DAGs and tasks.
- A `folder of DAG file`, read by the scheduler and executor (and any workers the executor has)
- A `metadata database`, used by the scheduler, executor and webserver to store state.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/103e04dc-426d-4c3f-b3d2-980ddba88cd8)
### <br/><br/><br/>

## Operator 종류
### 자세한 정보는 airflow 공식 홈페이지를 참고한다.
#### [airflow - operators](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/operators.html)
- Action Operator : 간단한 연산 수행 오퍼레이터, airflow.operators 모듈 아래에 존재. 실습에서 사용할 대부분의 오퍼레이터는 여기에 속한다.
- Transfer Operator : 데이터를 옮기는 오퍼레이터, <출발>To<도착>Operator 꼴.
- Sensor : 태스크를 언제 실행시킬 트리거(이벤트)를 기다리는 특별한 타입의 오퍼레이터 (예를 들어 어떤 폴더에 데이터가 쌓여지기를 기다린다든지, 요청에 대한 응답이 확인되기를 기다린다든지).
- PythonOperator : 파이썬 코드를 돌리는 작업을 할 때 사용하는 기계
- BashOperator : bash 명령어를 실행시키는 작업을 할 때 사용하는 기계
- SqliteOperator : SQL DB 사용과 관련된 작업을 할 때 사용하는 기계
- SimpleHttpOperator : HTTP 요청(request)을 보내고 응답(response) 텍스트를 받는 작업을 할 때 사용하는 기계
- HttpSensor : 응답(response)하는지 확인할 때 사용하는 센서 기계
### <br/><br/><br/>

--------------------------------------------------------

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
### 수정 yaml
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
      - AIRFLOW_WEBSERVER_HOST=182.162.88.163
      - AIRFLOW_WEBSERVER_PORT_NUMBER=8088
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_SECRET_KEY=a25mQ1FHTUh3MnFRSk5KMEIyVVU2YmN0VGRyYTVXY08=
  airflow-worker:
    image: docker.io/bitnami/airflow-worker:2
    environment:
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USERNAME=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_WEBSERVER_HOST=182.162.88.163
      - AIRFLOW_WEBSERVER_PORT_NUMBER=8088
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_SECRET_KEY=a25mQ1FHTUh3MnFRSk5KMEIyVVU2YmN0VGRyYTVXY08=
  airflow:
    image: docker.io/bitnami/airflow:2
    environment:
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USERNAME=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_WEBSERVER_PORT_NUMBER=8088
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_SECRET_KEY=a25mQ1FHTUh3MnFRSk5KMEIyVVU2YmN0VGRyYTVXY08=
    ports:
      - '8088:8088'
volumes:
  jhshin_airflow_postgresql_data:
    driver: local
  jhshin_airflow_redis_data:
    driver: local

```
### 그럼 이렇게 로그가 조회가 될 것 이다.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/9f0ceadc-0ab5-4a18-a261-de77b5461194)
### <br/>

## DAG 실행 예제
### airflow 는 interactive 하게 현재 진행 중인 상태를 보여준다.
https://github.com/Shin-jongwhan/airflow/assets/62974484/57af569d-4053-4392-b131-dbda8b0d8e09
### <br/>

## 













### <br/><br/><br/>


