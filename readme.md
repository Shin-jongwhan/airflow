### 230901
# airflow 정리 공간
### airflow 는 DAG (Directed Acyclic Graph) 로 endpoint 가 있는 작업의 연결을 해주는 framework 이다.
### apache 재단에서 관리한다. 에어비앤비가 복잡해지는 워크플로우를 관리하기 위해서 시작한 프로젝트이다.
### <br/><br/><br/>

## docker bitnami/airflow 로 airflow 구축하기
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

### docker volume 생성
```
docker volume create jhshin_airflow_postgresql_data
docker volume create jhshin_airflow_redis_data
```
#### 
### <br/>

### yaml 파일 일부 수정
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
  airflow-worker:
    image: docker.io/bitnami/airflow-worker:2
    environment:
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USERNAME=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_WEBSERVER_HOST=airflow
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

### airflow 웹서버 실행 확인
### 먼저 telnet 으로 접속 되는지 확인하였다
```
telnet [IP address] [port]
```
### 그 다음 웹으로 접속해보았다.
#### ![image](https://github.com/Shin-jongwhan/airflow/assets/62974484/617a766d-8677-4be8-b90f-eb9e2e41fc76)
### <br/><br/><br/>
