# 마이크로서비스 배포
- 배포는 소프트웨어를 운영에 반영하기 위해 사람(개발자/운영자/기획)이 해야 하는 단계

- 운영 환경의 4대 필수 기능
  - 서비스 관리 인터페이스 : 개발자가 서비스를 생성, 수정, 구성 할 수 있는 인터페이스 제공.
  - 런타임 서비스 관리 : 서비스 인스턴스가 항상 적정한 개수만큼 실행되도록 함. 서비스 인스턴스가 깨질 때, 재시동
  - 모니터링 : 서비스가 무슨 일을 하고 있는지, 로그 파일 및 지표를 개발자가 관찰할 수 있게 함. 운영환경에 문제가 있으면 개발자가 알아야 함 (알람)
  - 요청 라우팅 : 사용자 요청을 서비스로 보냄

- 네 가지 주요 배포 옵션
  - 언어에 특정한 패키지(JAR/WAR)로 서비스를 배포 : 단점이 많지만, 다른 옵션을 권장하는 이유라서 같이 살펴범
  - 서비스를 가상 머신으로 배포 : 서비스를 가상 머신 이미지로 묶어 배포 하는 방식. 서비스의 기술 스택을 캡슐화
  - 서비스를 컨테이너로 배포 : 컨테이너는 가상 머신보다 가벼움. (쿠버네티스)
  - 서비스를 서버리스 배포 기술로 배포 : 컨테이너보다 더 최근에 나온 기술. (AWS 람다) 


## 12.1 서비스 배포: 언어에 특정한 패키징 포캣 패턴
- 사전 필요
  - 필요한 런타임(JDK) 설치
  - WAR 파일 배포 시, 웹 컨테이너(톰캣) 설치
  - 자동 배포를 위한 파이프라인 구축

- 머신 하나에 여러 JVM 띄워서, JVM 당 하나의 서비스 인스턴스 가동
- 하나의 JVM에 여러 서비스 인스턴스 실행도 가능 (격리 안됨)

### 12.1.1 언어에 특정한 패키징 포맷 패턴의 장점
- 장점
  - 배포가 빠름
  - 리소스를 효율적으로 활용할 수 있음(같은 머신이나, 프로세스 내에 여러 인스턴스 실행) 

#### 배포가 빠르다
- 서비스를 복사해서 그냥 시동하면 되기 때문에 속도가 가장 빠름
- 오버헤드가 없어서 서비스도 빨리 시동되는 편

#### 리소스를 효율적으로 활용할 수 있다
- 여러 서비스 인스턴스가 머신과 OS를 공유하므로 리소스를 효율적으로 활용할 수 있음 (장점이자 단점)

### 12.1.2 언어에 특정한 패키징 포맷 패턴의 단점
- 단점
  - 기술 스택을 캡슐화할 수 없음
  - 서비스 인스턴스가 소비하는 리소스를 제한할 수 없음
  - 여러 서비스 인스턴스가 동일 머신에서 실행될 경우 서로 격리할 수 없음
  - 서비스 인스턴스를 어디에 둘지 자동으로 결정하기 어려움 

#### 기술 스택을 캡슐화할 수 없다
- 서비스별로 런타임 버전이 젖ㅇ해져 있고, 필요한 소프트웨어 패키지 버전이 상이할 수 있으므로 정확하게 구분해야 함
- 서비스마다 사용한 언어/프레임워크가 다양할 수 있고, 같은 언어/프레임워크라도 버전이 제각각일 수 있으니 많은 세부 정보를 운영팀과 공유

#### 서비스 인스턴스가 소비하는 리소스를 제한할 방법이 없다
- 인스턴스가 소비하는 리소스를 제한할 방법이 없음
  - 한 프로세스가 전체 CPU/메모리를 다 소모하면, 다른 서비스 인스턴스에게 영향을 줌 

#### 여러 서비스 인스턴스가 동일 머신에서 실행될 경우 서로 격리할 수 없다
- 하나의 인스턴스가 오작동하면 다른 인스턴스에도 영향을 기칠 수 있음

#### 서비스 인스턴스를 어디에 둘지 자동으로 결정하기 어렵다
- CPU, 메모리 같은 리소스는 한정되어 있고 각 서비스 인스턴스는 일정량의 리소스가 필요하기 때문에 너무 지나치지 않게, 머신을 최대한 효율적으로 활용하는 방향으로 서비스 인스턴스를 배정


## 12.2 서비스 배포: 가상 머신 패턴
- 서비스를 직접 배포 하는거보다 AMI(dlalwl)로 묶어 배포하는 방식이 낫다.
- 서비스의 배포 파이프라인은 VM 이미지 빌더를 실행해서 서비스 코드 및 소프트웨어 실행에 필요한 각종 라이브러리가 포함된 VM 이미지 생성

### 12.2.1 가상 머신 패턴의 장점
- 장점
  - VM 이미지로 기술 스택을 캡슐화함
  - 서비스 인스턴스가 격리됨
  - 성숙한 클라우드 인프라를 활용 

#### VM 이미지로 기술 스택을 캡슐화한다
- 서비스와 연관된 디펜던시를 모두 VM 이미지에 담을 수 있음
- 서비스를 가상 머신으로 묶는다는 것은 기술 스택이 캡슐화된 블랙 박스를 만드는 것과 같음

#### 서비스 인스턴스가 격리된다
- 각 서비스 인스턴스가 서로 완전히 동떨어져 동작하기 때문에 정해진 CPU/메모리 리소스가 가상 머신마다 배정되므로 다른 서비스에 있는 리소스에 영향을 주지 않음

#### 성숙한 클라우드 인프라를 활용한다
- AWS 등의 퍼블릭 클라우드는 물리 머신에 과부하를 유발하지 않는 방향으로 여러 VM을 스케줄링하며, VM 간 부하 분산 및 자동 확장 등 유용한 부가 기능 제공

### 12.2.2 가상 머신 패턴의 단점
- 단점
  - 리소스를 효율적으로 활용할 수 없음
  - 배포가 비교적 느림
  - 시스템 관리 오버헤드가 발생

#### 리소스를 효율적으로 활용할 수 없다
- 서비스 인스턴스마다 OS를 포함한 전체 가상 머신의 오버헤드가 있음
  - 퍼블릭 Iaas 가상 머신은 대부분 VM 크기가 한정되어 있어 VM을 십분 활용하기 어려움

#### 배포가 비교적 느리다
- VM 이미지는 대부분 크기가 커서 빌드 시간이 몇 분 정도 걸리고 네트워크를 통해 이동하는 데이터양도 많음
- VM 이미지에서 VM 인스턴스를 생성할 때도 네트워크 데이터양이 많기 때문에 다소 시간이 걸리는 편

#### 시스템 관리 오버헤드가 발생한다
- OS/런타임 패치를 해야 함.
  - 서버리스 사용 시, 이런 시스템 관리 필요 없음

## 12.3 서비스 배포: 컨테이너 패턴
- 컨테이너는 OS 수준에서 가상화한 메커니즘
- 프로세스 입장에서는 컨테이너는 마치 자체 머신에서 실행되는 것처럼 실행

### 12.3.1 서비스를 도커로 배포
- 서비스를 컨테이너로 배포하려면 반드시 컨테이너 이미지로 묶어야 함
  - 애플리케이션과 서비스 구동에 필요한 모든 소프트웨어로 구성된 파일 시스템 이미지

#### 도커 이미지 빌드
- 이미지를 빌드하는 방법이 기술된 도커파일(Dockerfile)을 생성
- 도커파일 기반으로 이미지 빌드

#### 도커 이미지를 레지스트리에 푸시
- 이미지를 레지스트리에 푸시하려면 두 커맨드를 실행
  - 1) tag 커맨드로 이미지 앞에 호스트명과 레지스트리 포트를 붙임
  - 2) 도커 push 커맨드로 태그를 붙인 이미지를 레지스트리에 업로드 

#### 도커 컨테이너 실행
- 서비스를 컨테이너 이미지로 패키징한 후에는 하나 이상의 컨테이너 생성 가능

- 의존성이 있을 때 도커 컴포즈 사용하면 간편

### 12.3.2 컨테이너 패턴의 장점
- 장점
  - 기술 스택의 캡슐화. 서비스 관리 API가 곧 컨테이너 API가 됨
  - 서비스 인스턴스가 격리됨
  - 서비스 인스턴스의 리소스를 제한할 수 있음

### 12.3.3 컨테이너 패턴의 단점
- 단점
  - 컨테이너 이미지를 직접 관리해야 하는 부담이 있음

## 12.4 FTGO 애플리케이션 배포: 쿠버네티스
- 쿠버네티스는 도커 오케스트레이션 프레임워크로, 도커를 기반으로 여러 머신을 하나의 서비스 실행 리소스 풀로 전화하는 소프트웨어 계층
- 서비스 인스턴스에 이슈가 있더라도, 항상 원하는 개수 만큼 실행되도록 유지

### 12.4.1 쿠버네티스 개요
- 도커가 실행되는 여러 머신을 하나의 리소스 풀로 취급하는 도커 오케스트레이션 프레임워크

- 주요 기능
  - 리소스 관리 : 여러 머신을 CPU, 메모리, 스토리지 볼륨을 묶어 놓은 하나의 리소스 풀로 취급
  - 스케줄링 : 컨테이너를 실행할 머신을 선택. 스케줄링은 기본적으로 컨테이너의 리소스 요건 및 노드별 리소스 상황에 따라 결정
  - 서비스 관리 : 마이크로서비스에 직접 매핑되는 서비스를 명명하고 비저닝함. 인스턴스를 적정 개수만큼 가동시키고 요청 부하를 골고루 분산함. 서비스 롤링 업데이트

#### 쿠버네티스트 아키텍처
- 쿠버티스 클러스터의 머신은 마스터, 노드 둘중 하나임
  - 마스터는 클러스터를 관장
  - 노드는 하나 이상의 파드를 실행하는 워커 (파드는 여러 컨테이너로 구성된 쿠버네티스의 배포 단위)

- 마스터는 다음 컴포넌트를 실행
  - API 서버 : kuberctl CLI에서 사용하는 서비스 배포/관리용 REST API
  - etcd : 클러스터 데이터를 저장하는 키-값 NoSQL DB
  - 스케줄러 : 파드를 실행할 노드를 선택
  - 컨트롤러 관리자 : 컨트롤러는 클러스터가 원하는 상태가 되도록 제어 (인스턴스 개수등)

- 노드는 다음 컴포넌트를 실행
  - 큐블릿(kubelet) : 노드에서 실행되는 파드를 생성/관리 함
  - 큐브 프록시(kube-proxy) : 여러 파드에 부하를 분산하는 등 네트워킹 관리를 함
  - 파드 : 애플리케이션 서비스

#### 쿠버네티스트 핵심 개념
- 파드(pod)
  - 쿠버네티스의 기본 배포 단위
  - IP 주소, 스토리지 볼륨을 공유하는 하나이상의 컨테이너로 구성
  - 파드는 컨테이너와 파드가 실행하는 노드, 둘 중 하나는 언제라도 깨질 수 있기 때문에 일시적 임

- 디플로이먼트(deployment)
  - 파드의 선언형 명세
  - 항상 파드 인스턴스(서비스 인스턴스)를 원하는 개수만큼 실행시키는 컨트롤러
  - 롤링 업데이트/롤백 기능이 탑재된 버저닝을 지원

- 서비스(service)
  - IP 주소와 이 주소로 해석되는 DNS 명이 할당된 서비스는 TCP/UDP 트래픽을 하나 이상의 파드에 고루 분산
    - IP 주소, DNS명은 오직 쿠버네티스트 내부에서만 접근 가능 

- 컨피그맵(ConfigMap)
  - 애플리케이션 서비스에 대한 외부화 구성이 정의된 이름-값 쌍의 컬렉션
  - 파드 컨테이너의 데피니션은 컨테이너 환경 변수를 정의하기 위해 컨피그맵을 참조
  - 민감 정보는 시크릿이라는 컨피그맵 형태로 저장 (TLS)

### 12.4.2 쿠버네티스 배포: 음식점 서비스
- 쿠버네티스 오브젝트는 YAML 파일로 정의하는 것이 가장 쉬움

#### 쿠버네티스 서비스 생성
- 파드가 실행되면 쿠버네티스 디플로이먼트는 파드가 계속 실행되도록 관리


- 파드 IP 주소는 동적 할당되기 때문에 HTTP 요청을 원한느 클라이언트 입장에서는 쑬모가 없음
  - 클라이언트 쪽 디스커버리 메커니즘을 부착하고 서비스 레지스트리를 설치하면 됨 (서비스 디스커버리 메커니즘은 쿠버네티스에 기본 내장)

- IP 주소 및 DNS명이 할당된 서비스는 하나 이상의 파드 클라이언트에 안정된 끝점을 제공하는 오브젝트
  - 대상 파드를 선택하는 셀럭터(selector)가 핵심 

- 서비스는 내부용 로드밸런싱

### 12.4.3 API 게이트웨이 배포
- NodePort 서비스는 광역 클러스터 포트를 통해 (특정 포트) 클러스터의 모든 노드에 접근할 수 있음
  - 포트 번호는 30000 ~ 32767 중 선택
  - 해당 포트로 접근하면 모든 노드에 접근 가능

- LoadBalancer는 클라우드에 특정한 부하 분산기를 자동으로 구성하는 서비스

### 12.4.4 무중단 배포
- 쿠버네티스는 파드를 롤링 업데이트 함
  - 파드마다 신 버전이 요청 처리 준비가 완료되기 전에는 구 버전을 절대 중지하지 않음 (readinessProbe로 헬스체크) 

- 디플로이먼트는 롤아웃이라는 이력을 관리해서 커맨드로 이전 버전으로 쉽게 롤백 가능

### 12.4.5 배포와 릴리스 분리: 서비스 메시
- 운영 환경은 트래픽이 높기 때문에 스테이징 환경을 정확하게 동일한 레플리카로 맞추기 어려움 (환경을 완전히 일치 시키기 어려움)

- 새 버전을 확실하게 시작하려면 배포와 릴리스를 따로 분리
  - 배포 : 운영 환경에서 실행
  - 서비스 릴리스 : 최종 사용자에게 서비스를 제공

- 다음 5단계를 거쳐 서비스를 운영에 배포
  - 1. 최종 사용자 요청을 서비스에 라우팅하지 않고 새 버전의 서비스를 프로덕션에 배포
  - 2. 프로덕션에서 새 버전을 테스트
  - 3. 소수의 최종 사용자에게 새 버전을 릴리스
  - 4. 모든 웅영 트래픽을 소화할 때까지 점점 더 많은 사용자에게 새 버전을 릴리스 함
  - 5. 어딘가 문제가 생기면 곧장 구 버전으로 되돌림. 새 버전이 정확히 잘 동작한다는 확신이 들면 구 버전을 삭제


- 서비스 메시는 한 서비스와 다른 서비스, 외부 애플리케이션의 모든 통신을 중재하는 네트워킹 인프라
  - 테스트 사용자는 A버전으로 서비스, 최종 사용자는 B 버전의 서비스로 나누어 라우팅 하는 것도 가능 (이스티오)


#### 이스티오 서비스 메시 개요
- 이스티오는 마이크로서비스를 연결, 관리, 보안하는 오픈 플랫폼
- 이스티오 기능
  - 트래픽 관리 : 서비스 디스커버리, 부하 분산, 라우팅 규칙, 회로 차단기 등
  - 보안 : 전송 계층 보안(TLS)을 이용한 서비스 간 통신 보안
  - 텔레메트리 : 네트워크 트래픽 관련 지표 수집 및 분산 추적
  - 정책 집행 : 쿼터 및 사용률 제한 정책 적용 

- 컨트롤 플레인 : 데이터 플레인이 트래픽 라우팅하도록 구성하는 등의 관리 역할. 파일럿과 믹서로 구성
  - 파일럿 : 하부 인프라에서 배포된 서비스 관련 정보 추출 (서비스와 파드). 엔보이 프록시가 미리 정의된 라우팅 규칙에 따라 트래픽을 라우팅 하도록 구성
  - 믹서 : 엔보이 프록시에서 텔레메트리를 수집하고 정책을 집행하는 역할- 데이터 플레인 : 서비스 인스턴스별 엔보이 프록시로 구성

- 엔보이 프록시 : 엔보이를 변형한 것으로 저수준 프로토콜(TCP) 부터 고수준 프로토콜(HTTP/HTTPS) 까지 다양한 프로토콜을 지원하는 고성능 프록시

- 쿠버네티스 사용 시, 서비스 파드 내부의 컨테이너가 바로 엔보이 프록시

#### 이스티오로 서비스를 배포
- 이스티오 요건
  - 쿠버네티스 서비스 포트는 `<프로토콜>[접미어]` 포맷의 이스티오 명명 관례를 따라야함. ㅇㅣㄱ명 포트는 이스티오가 TCP 포트로 간주해서 규칙 기반의 라우팅을 적용하지 않음
  - 이스티오로 분산 추적을 하려면 파드에 app이라는 라벨을 붙여 서비스를 식별
  - 여러 버전의 서비스를 동시에 실행하려면 쿠버네티스 디플로이먼트명에 버전을 넣어야 함

#### 서비스 메시 (다시 한번 되새기자)
- 서비스 메시는 한 서비스와 다른 서비스, 외부 애플리케이션의 모든 통신을 중재하는 네트워킹 인프라
- 블루그린, 카나리 배포

## 12.5 서비스 배포: 서버리스 패턴

### 12.5.1 AWS 람다를 이용한 서버리스 배포
- AWS 람다는 애플리케이션을 파일로(ZIP, JAR) 묶고 업로드 하면 트래픽에 따라 인스턴스를 자동 실행. (사용자는 비용만 지불)
- 한계도 있지만 개발조직에서 서버, 가상 머신, 컨테이너 관련 부분을 신경 쓸 필요가 없다는 점에서 매우 강력함

### 12.5.2 람다 함수 개발
- RequestHandler 인터페이스 사용
- 자바 람다는 ZIP또는 JAR 파일로 패키징

### 12.5.3 람다 함수 호출
- 람다 함수 호출 방법
  - HTTP 요청
  - AWS 서비스에서 생성된 이벤트
  - 스케줄링된 호출
  - API를 직접 호출 

#### HTTP 요청 처리
- AWS API 게이트웨이가 HTTP 요청을 람다 함수로 라우팅 (API 게이트웨이가 프록시 역할)

#### AWS 서비스에서 생성된 이벤트 처리
- AWS 서비스에서 생성된 이벤트를 람다 함수가 처리하도록 트리거
  - S3 버킷에 객체가 생성됨
  - DynamoDB 테이블의 데이터 항목이 생성, 수정, 삭제됨
  - 키네시스 스트림에서 메시지를 읽을 준비가 됨
  - SES를 통해 이메일을 수신

- AWS 람다는 다른 AWS 서비스와 완벽하게 연계됨

#### 람다 함수 스케줄링
- 리눅스크론 같은 스케줄러로 람다 함수를 주기적으로 호출

#### 웹 서비스를 요청하여 람다 함수 호출
- 웹 서비스를 요청할 때 람다 함수명과 입력 이벤트 데이터를 지정하고, 람다 함수를 동기/비동기 호출


### 12.5.4 람다 함수의 장점
- 장점
  - 다양한 AWS 서비스와의 연계 : DynamoDB, 키네시스 등 풍부한 AWS 서비스의 이벤트를 소비할 수 있고, AWS API 게이트웨이를 통해 HTTP 요청을 처리할 수 있음
  - 시스템 관리 업무가 많이 경감됨 : 시스템에 신경 쓸 필요 없이 애플리케이션 개발에만ㅁ 전념할 수 있음
  - 탄력성 : AWS 람다는 애플리케이션 부하 처리에 필요한 개수만큼 인스턴스를 실행함
  - 사용량만큼 과금 : AWS 람다는 실제로 요청을 처리하기 위해 소비한 리소스만큼 비용을 지불

### 12.5.5 람다 함수의 단점
- 단점
  - 긴-꼬리 지연 : AWS 람다는 코드를 동적 실행하므로 AWS가 애플리케이션 인스턴스를 프로비저닝하고 애플리케이션을 시동하기까지 시간이 걸림
  - 제한된 이벤트/요청 기반 프로그래밍 모델 : AWS 람다는 처음부터 실행 시간이 긴 서비스를 배포할 용도는 아님

## 12.6 REST 서비스 배포: AWS 람다 및 AWS 게이트웨이
- 각 요청 핸들러 클래스마다 람다 함수 하나씩 존재

### 12.6.1 음식점 서비스를 AWS 람다 버전으로 설계

#### FindRestaurantRequestHandler 클래스
- AWS SDK의 RequestHandler 인터페이스가 루트, 그 하위 추상 클래스들은 에러 처리, 디펜던시 주입을 담당
- AbstractHttpHandler는 요청 처리 도중 발생한 예외를 붙잡아 내부 서버 오류 응답(500)을 반환
- AbstractAutowiringHttpRequestHandler는 요청 처리에 필요한 디펜던시 주입
- FindRestaurantRequestHandler는 음식 점 정보를 검색 후, HTTP 응답에 해당하는 값 반환

#### AbstractAutowiringHttpRequestHandler 클래스로 디펜던시 주입
- `SpringApplication.run()` 으로 `ApplicationContext`를 생성 후, 최초로 들어온 요청을 처리하기 전에 디펜던시를 자동와이어링 함

#### AbstractHttpHandler 클래스
- 이 클래스 주요 임무는 요청 처리 시 발행한 예외를 붙잡아 에러 코드 500을 던지는 일

### 12.6.2 ZIP 파일로 서비스 패키징
- 그레이들 태스크를 이용하여 ZIP 파일로 빌드

### 12.6.3 서버리스 프레임워크로 람다 함수 배포
- 람다 함수를 쉽게 사용할 수 있게 도와 주는 오픈소스 서버리스 프로젝트가 있음
  - 서버리스가 대신 람다 함수를 배포하고 이 함수들로 요청을 라우팅하는 API 게이트웨이를 생성/구성
