# Istio Introduction

3월 3일 한국 쿠버네티스 유저그룹의 스터디 내용 정리입니다.

## Monolithic vs. Micro Service and Service Mesh

### Monolithic Service

- 고전적인 방식으로 각기 다른 역할을 하는 서비스 & 기능들을 하나의 프로젝트로 묶어서 관리되는 형태
- 서비스의 규모가 커지면 구조 자체의 복잡성이 증가.
  - 일부의 변경이 Side Effect로 인해 전체적인 안정성을 크게 저하
  - 기능의 변경에 대한 빌드 및 배포에 대한 시간 소요가 증가

### Micro Service

- 여러개의 독립된 서비스를 모아 하나의 시스템으로 제공
- 각 개별 서비스의 기능이 독립적으로 동작하기 때문에 기능/수정이 용이, 빌드 및 배포 시간이 짧음
- 단, 규모가 커지면 거대해진 Network로 인해 통신이 어렵거나 테스트가 복잡해짐 => 최종적으로 논리적 구조 자체가 복잡해질 위험이 큼

#### 구현방식

- REST API를 통한 Request/Response 구조
- Queue를 통한 Event Driven 구조
  - Resonse를 즉각 확인이 어려우나 호출이 많은 서비스에서 활용
- 초기에는 client library 방식이었으나, 언어 형평성/업그레이드 문제 존재
  - 이후 service proxy 방식의 MSA를 활용.
  - Sidecar를 통한 처리. 개발 언어 종속성 배제. => 업그레이드가 용이

#### Terminology

- **Service Discovery**: 서비스를 찾고 연결하고 로드 밸런싱을 해야 하는 기능
- **Circuit Breaker**: 고장에 대응하기 위한 용도, alert를 바탕으로 운영, propagation 되면 각 서비스가 알아서 이에 대한 제어 처리.
- **Sidecar**: Proxy Pattern을 바탕으로 한 MSA의 Design Pattern

### Service Mesh

- MSA에서 발생하는 관리 어려움을 해결하기 위한 아키텍처
- 서비스간 직접적인 통신보다 Sidecar 기능을 갖는 Proxy 를 통해서 통신하는 방식으로 동작
  - 트래픽을 네트워크 레벨에서 제어 가능
  - 독립된 proxy 모델을 통해 로깅 및 트레이싱에 유리
- 단, 규모가 커지면 proxy 자체가 증가하기 때문에 중앙에서 이들에 대한 통제를 해야 함.

## Istio

Service Mesh 에 대한 대표적인 구현체로, Data Plane과 Control Plane으로 나뉘어 있음. Google, IBM, Lyft가 2017년 5월 오픈소스로 공개한 프로젝트로 Google의 Open Usage Common Foundation으로 운영되는 프로젝트.

### Data Plane

실제 네트워크 트래픽을 처리하는 부분으로 Sidecar 형태로 구성된 proxy들을 가리키며, Control Plane을 통해 제어됨.

#### Envoy

Sidecar 패턴을 가장 잘 구현한 솔루션으로 Istio는 Envoy에 Control Plane을 붙여 확장한 솔루션

- 경량화 된 L7용 Proxy
- 지원 프로토콜: HTTP, gPRC, TCP
- 지원 기능: Circuit Breaker, Retry, Timeout

### Control Plane

Data Plane에 대한 컨트롤을 하는 부분으로 Pilot, Citadel, Galley,  Mixer로 구성.

#### Pilot

Traffic Manager 기능 지원

- Service Discovery
- Traffic Retry
- Circuit Breaker
- Timeout

#### Mixer

Service Mesh의 Access 제어 및 정책 관리. 모니터링 지표 수집

#### Citadel

인증 기능 관리 (TLS 등)

#### Galley

Istio Configuration 체크. k8s의 yaml을 Istio가 이해할 수 있는 형태로 변환.
