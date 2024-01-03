# 모니터링 대시보드 (feat. SpringActuator, Prometheus, Grafana)

# SpringActuator
운영 중인 애플리케이션을 ```HTTP```나```JMX```를 이용해 모니터링하고 관리할 수 있는 기능을 제공한다.

또한 프로메테우스, 그라파나와 같은 다양한 모니터링 시스템과 쉽게 연동할 수 있다.

> JMX : 애플리케이션, 시스템, JVM 등을 모니터링하고 관리하기 위한 도구를 제공하는 자바 기술.

```build.gradle``` 에 하기 코드를 삽입하여 의존성을 추가한다.
```
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```
## 주요 엔드 포인트
- ```beans```
  - 애플리케이션의 모든 Spring Bean 목록을 표시한다.
- ```health```
  - 애플리케이션의 현재 상태 정보를 표시한다.
- ```env```
  - 스프링의 모든 환경변수 정보를 표시한다.
- ```metrics```
  - 애플리케이션의 각종 지표를 제시한다. (CPU Usage, Connection Pool, etc..)
- ```refresh```
  - 애플리케이션의 설정 정보를 갱신할 수 있다.
- 그 외
  - https://docs.spring.io/spring-boot/docs/3.0.5/reference/html/actuator.html#actuator.endpoints

## 보안
- actuator는 endpoint를 개방함으로 기본적으로 모든 endpoint에 대해서 disable 상태를 유지하고, 운영상 필요한 endpoint만 개방해야 한다.
- 또한 접근에 대해 인증이 된 권한을 보유한 사용자만 접근이 가능하도록 해야 한다.
- 서비스 운영에 사용되는 포트와 다른 포트를 사용해야 한다.
- 기본 경로인 ```/actuator``` 대신 경로를 변경하여 운영한다.

# Prometheus
Prometheus는 메트릭 수집, 시각화, 알림, 서비스 디스커버리 기능을 모두 제공하는 오픈 소스 모니터링 시스템이다.

대표적인 기능으로는 아래와 같다.
- 풀 방식의 메트릭 수집, 시계열 데이터 저장
- PromQL을 활용하여 저장된 시계열을 쿼리 및 집계
- 서비스 디스커버리
- 데이터 시각화
## 아키텍처
![image](https://github.com/jekyllPark/prometheus-grafana-tutorial/assets/114489012/9224d3d8-729c-425f-acca-df5995630e30)

먼저 Prometheus는 시계열 데이터를 저장한다. Exporter는 다양한 서비스/시스템의 메트릭을 수집한다. 예를 들어 Node Exporter는 설치된 머신의 CPU, Memory 등의 메트릭 정보를 수집하게 된다. Client Library는 애플리케이션 코드를 계측하기 위해 쓰인다. Pushgateway는 앞에 두 개의 컴포넌트가 수집하기 어려운 배치 데이터 등을 수집할 때 사용된다. Prometheus는 설정 파일(prometheus.yml)에 작성된 "Job"을 통해서 이들이 수집하는 메트릭을 가져와서 저장한다.

또한, Alertmanager를 통해서 특정 메트릭이 임계치가 넘어가거나 경계에 잡혔을 때 이메일, 슬랙 등을 통해서 알림을 보내줄 수 가 있으며 UI 기능이 있어 데이터를 시각화할 수 있다. 하지만 자체적인 시각화 기능은 약한 편이며 보통 Grafana라는 오픈 소스 대시보드 툴로 Prometheus UI를 대체하는 편이다.

그리고 서비스 디스커버리 기능을 제공한다. 마이크로 서비스가 대중적으로 유행하는 지금 상황에서 인스턴스는 다이나믹하게 스케일 인/아웃이 된다. 이를 수동으로 관리하기는 불가능에 가깝다. Prometheus는 다행히 여러 서비스 디스커버리와 통합할 수 있다. 가령, 쿠버네티스 서비스 디스커버리와 통합하여, 쿠버네티스 클러스터에 존재하는 모든 노드와 팟들의 메트릭을 수집할 수가 있다.

마지막으로 Prometheus는 애초에 "스케일 아웃"을 고려하지 않고 설계되었다. 그래서 데이터가 많으면 많을수록 이를 어떻게 해결해야 할까라는 고민 때문에, 도입하기가 쉽지 않았다. 하지만, Prometheus 클러스터링을 위한 Thanos, Cortex등 여러 오픈 소스가 개발되면서 이 문제가 상당 수 해결되었다.

## 적용 방법
! 당연히 프로메테우스가 우선 OS에 설치가 되어있어야 한다.

Prometheus가 metric을 폴링할 수 있도록 엔드포인트를 제공해주어야 한다.

build.gradle에 ```prometheus``` 의존성을 추가해준다.
```
implementation 'io.micrometer:micrometer-registry-prometheus'
```

이후, actuator의 ```exposure``` 설정 또한 변경해주어야 한다.
```
management:
  endpoints:
    web:
      exposure:
        include: "prometheus"
```

정상 여부 확인을 위해, 엔드포인트 ```/actuator/prometheus```를 통해 확인할 수 있다.

도커를 통해 각 모니터링 서버를 관리하기 위해 ```docker-compose```를 아래와 같이 구성한다.
```
version: '3'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
```
![image](https://github.com/jekyllPark/prometheus-grafana-tutorial/assets/114489012/a9a0e594-d94d-4b0a-bc64-176fcf4c47de)

별도의 설정이 없다면 프로메테우스는 metrics를 ```self scrape```하기 때문에 볼륨 마운트를 통해 prometheus.yml 파일을 설정해준다.
```
-- prometheus.yml
scrape_configs:
  - job_name: 'tutorial'
    metrics_path: '/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:8080']
```
```docker-compose```에 볼륨 마운트를 추가해준다.
```
version: '3'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```
> 만약 connection refused를 반환받을 경우, localhost가 아닌 본인의 아이피로 수정해주면 잘 된다.

이후 도커를 재실행하면 프로메테우스 내 metrics 타겟이 설정된 것을 확인할 수 있다.

![image](https://github.com/jekyllPark/prometheus-grafana-tutorial/assets/114489012/0d96fd5c-82c9-4f79-8f81-f5fd68f8a397)

# Grafana
Prometheus를 통해 주기적으로 메트릭을 수집하였지만, 이를 확인하려면 매번 PromQL을 통해 지표를 질의해야 하고, 시각화에 아쉬움이 존재한다.

이를 해소하기 위해 Grafana를 통해 대시보드를 제공 받을 수 있다.

Grafana는 시계열 메트릭 데이터를 시각화 하는데 가장 최적화된 대시보드를 제공해주는 오픈소스 툴킷이다.

다양한 DB를 연결하여 DB의 데이터를 가져와 시각화 할 수 있으며, 그래프를 그리는 방법도 간단히 마우스 클릭으로 완료할 수 있다.

또한 시각화한 그래프에서 특정 수치 이상으로 값이 치솟을 때, 알림을 전달 받을 수 있는 기능도 제공한다.

## 적용 방법
프로메테우스와 마찬가지로 도커를 통해 구성하기 위해 ```docker-compose```를 수정해준다.
```
version: '3'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
       - ./prometheus.yml:/etc/prometheus/prometheus.yml

# 추가
	grafana:
    image: grafana/grafana:latest
    container_name: grafana
    user: "$UID:$GID"
    ports:
      - "3000:3000"
    volumes:
      - ./grafana-data:/var/lib/grafana
    depends_on:
      - prometheus
```
> 주의할 점으로는, 도커 컨테이너가 내려가더라도 Grafana에서 설정한 대시보드 혹은 데이터 소스가 사라지지 않게 볼륨 마운트를 꼭 설정해주어야 한다.

도커 재실행 후, 3000 포트로 접근 시 하기와 같이 페이지가 반환되면 정상 실행된 것이다.
![image](https://github.com/jekyllPark/prometheus-grafana-tutorial/assets/114489012/69ded7e3-3c43-493a-ae47-20a19920d537)

```add data source```를 통해 Prometheus의 포트를 등록해주고, 

![image](https://github.com/jekyllPark/prometheus-grafana-tutorial/assets/114489012/471d1e9c-dac7-4407-8196-72fde38b7288)

Grafana Labs를 통해 원하는 Dashboard를 import 해주면 시각화된 메트릭을 볼 수 있다.

![image](https://github.com/jekyllPark/prometheus-grafana-tutorial/assets/114489012/c3e5e38d-5dde-4bd9-8230-1348db8ee3b6)


# Ref
- https://velog.io/@roycewon/Spring-boot-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81Prometheus-Grafana-docker#spring-actuator
- https://gurumee92.tistory.com/220
- https://medium.com/finda-tech/grafana%EB%9E%80-f3c7c1551c38
