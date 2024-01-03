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

## 보안
- actuator는 endpoint를 개방함으로 기본적으로 모든 endpoint에 대해서 disable 상태를 유지하고, 운영상 필요한 endpoint만 개방해야 한다.
- 또한 접근에 대해 인증이 된 권한을 보유한 사용자만 접근이 가능하도록 해야 한다.
- 서비스 운영에 사용되는 포트와 다른 포트를 사용해야 한다.
- 기본 경로인 ```/actuator``` 대신 경로를 변경하여 운영한다.

# Prometheus

# Grafana

# Ref
- https://velog.io/@roycewon/Spring-boot-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81Prometheus-Grafana-docker#spring-actuator
