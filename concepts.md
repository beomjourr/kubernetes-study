> 하나의 서버에는 기본적으로 어떤 운영체제든 Host OS(내 컴퓨터에 직접 설치된 운영체제)가 올라감
> 

## VM (Virtual Machine)

- **구조**: 물리 서버 위에 **하이퍼바이저**(VMware, VirtualBox 같은 가상화 소프트웨어)를 두고, 그 위에 여러 개의 OS(게스트 OS)를 돌리는 방식
    - 하이퍼바이저란?
        - 가상화 관리자 (물리서버와 Hypervisor는 1:1 관계)
        - 물리적 하드웨어(CPU, 메모리, 저장소)를 여러 VM에게 나눠줌
        - 각 VM이 독립적으로 돌아가도록 관리
- **특징**
    - 각각의 VM이 **자체 OS**를 갖고 있음.
    - **무겁고, 부팅이 느림.**
    - 보안·격리 수준이 높음 (서로 완전히 분리된 독립된 컴퓨터처럼 동작).
    - 내가 윈도우를 사용하고 있더라도, 새로운 Guse OS 설치할때 Linux를 설치해서 쓸 수 있음 (예시)
- Host OS 없이도 쓰는 경우 있음
    - 어차피 가상화 해서 Guest OS들 쓸건데, Host OS 있어봤자
- 클라우드(EC2 등) 쓰면 Container보다 별도로 해야되는건 없음

## Container

> 애플리케이션을 실행하기 위해 필요한 모든 것(코드 + 라이브러리 + 설정)을 묶어서, 격리된 환경에서 실행시키는 기술
> 
- 호스트 OS의 커널을 공유하며 자원을 격리
- **구조**: 호스트 OS 위에 **컨테이너 엔진(Docker 같은 것)**을 두고, 애플리케이션과 필요한 라이브러리만 분리해서 실행.
- **특징**
    - OS 커널을 공유 → **가볍고 빠름**.
    - 실행과 종료가 빠르다 (초 단위).
    - 자원 효율이 좋음 (더 많은 컨테이너를 같은 서버에 돌릴 수 있음).
    - VM보다 격리가 약간 덜하지만, 운영환경에서 충분히 안전하게 사용 가능.
    - 한 컨테이너가 보안에 뚫려 OS영역에 접근하게 되면 다른 컨테이너가 위험해질 수도 있긴함 (보완은 가능)
    - 내가 윈도우를 사용하고 있다면,  Linux를 설치해서 쓸 수 없음 (예시)
- 클라우드(ECS/EKS) 써도 Docker 이미지를 만들거나, Kubernetes를 설정해주는 작업이 별도로 필요
- **쿠버네티스 환경에서 호스트 OS = Pod가 할당된 노드의 운영체제**
- 도커의 역할은 컨테이너 생성과 실행까지


**Kubernetes Cluster**

- Master와 Node들 (그래서 최소 2대 이상이 있어야 ‘정식 클러스터’)

 **Master (Control Plane Node)  - 관리 서버**

- 서버 1대
- 쿠버네티스의 전반적인 기능들을 컨트롤하는 역할
    - 스케줄러도 여기에 들어감
- 직접 애플리케이션을 실행하지 않고, **명령 내리고 관리만 함**.

**Node (Worker Node) - 일꾼 서버**

- 실제 파드(Pod, 컨테이너)가 실행되는 곳
- 나머지 서버들
- 한 마스터에 여러 노드들이 연결됨
- 자원 제공하는 역할
- 클러스터 전체 자원을 늘리고 싶다면 Node를 늘리면 됨

**Namespace**

- 쿠버네티스 object들을 독립된 공간으로 분리시켜주는 역할
- 다른 namespace에 있는 pod에는 연결 불가, Service도 연결 불가
- 한 Namespace에서는 같은 이름의 Pod를 생성할 수 없음

**Pod**

- 쿠버네티스 최소 배포단위
- Pod에는 여러 컨테이너들이 있음 (컨테이너 하나당 하나의 앱)
    - 함께 배포되고 함께 죽고 함께 사는 컨테이너들의 그룹 (프론트, 백, …)
- Pod에 문제가 생겨서 재생성 되면 그 안에 있던 데이터가 날라감
    - 이 문제를 해결하기 위해 Volume을 만들어서 Pod에 연결하면 데이터는 이곳에 별도로 저장 할 수 있음
- 쿠버네티스 클러스터에서 Pod의 IP로 접근 가능 - 완전 외부에서는 접근 불가능

**Service**

- Pod들에게 외부로부터 연결이 가능하도록 IP를 할당 , 로드밸런싱 등
- Service의 IP로 접근하면 항상 연결되어있는 Pod에 접근 가능
    - Pod의 IP는 Pod가 재생성되면 바뀜 → Pod의 IP는 신뢰성이 떨어짐
    - Service의 IP는 사용자가 지우지 않는한, 삭제나 재생성되지 않음
- Pod에는 labels을, Sevice에는 selector를 동일하게 지정해서 연결함

**ResourceQuota / LimitRange**

- 한 Namepsace에서 사용할 수 있는 자원의 양을 제한시킬 수 있음 (Pod개수 제한, cpu/mem제한 등)

**ConfigMap / Secret**

- Pod 생성 시에 컨테이너 안에 환경변수 값을 넣어주거나 파일을 마운팅 시켜줄 수 있음 (세팅)
    - ConfigMap - 환경 설정 등
    - Secret - 비밀번호/토큰 같은 민감한 정보

**Controller**

- Pod 관리
- Pod를 만들어주고, Pod가 죽었을 때 다시 생성해줌



**Container**

- Pod 안에 있는 컨테이너끼리는 포트가 중복될 수 없다

**Label**

- Key : value 형태
- Service에서 selector에 Pod에 있는 labels의 키값과 동일하게 적으면 연결됨

**Node Schedule**

1. Pod를 만들 때 노드를 직접 지정하는 방법
    1. hostname: node1
2. 쿠버네티스의 스케줄러가 판단해서 지정
3. request, limits
    - Memory: 초과 시 Pod 종료 시킴
    - Cpu:  초과해도 Pod 종료시키진 않음 (느려질뿐)



## Volume
emptyDir

- 설정만 하면 파드와 함께 생성되고, 파드가 죽으면 같이 사라짐
- 용도 : 임시 파일, 캐시 등

hostPath

- 노드와 함께 생존
- mountPath랑 hostPath를 지정해줌
    - hostPath → host OS, 즉 volume 데이터들은 host OS에 저장되므로 파드가 죽어도 남아있음

PV/PVC 

- 클러스터 전체에서 영구 보존
- PV (PersistentVolume)
    - 관리자가 미리 준비해둔 스토리지 자원
- PVC (PersistentVolumeClaim)
    - 개발자가 작성한 스토리지 요청서

![image.png](attachment:f31d7188-789c-4c32-872c-710a40f2b15d:image.png)

- Pod -  PVC - PV
- pv-pvc 1:1로만 해야되는 것도 있음
- storageClass
    - 동적인 설정이 가능하게 해줌
    - pv를 설정 안해도 sc가 pv를 만들어달라고 할 수 있음
        - pod - psv -sc → (pv)
    - 클라우드 환경에서 많이 쓰임


## sevice
ClusterIP (기본값)

- 내부 전용: 클러스터 안에서만 접근 가능
- 용도: 내부 서비스끼리 통신(DB, API 등)
- **접근**: `service-name:port`

NodePort

- 외부 접근: 각 노드의 특정 포트로 접근 가능
- 포트 범위: 30000~32767
- **접근**: `NodeIP:NodePort`
- ClusterIP기능도 포함

LoadBalancer

- **클라우드 로드밸런서**: 외부 로드밸런서 자동 생성
- **고정 외부 IP**: 별도 IP 주소 할당
- **AWS/GCP/Azure 등 클라우드 환경 필요**
- **NodePort + ClusterIP 기능도 포함**

## Namespace, ResourceQuota, LimitRange
ResourceQuota
- ResourceQuota 생성 시 resources(requests, limits)을 지정하지 않으면 생성되지 않음 

- 파드를 먼적 만들고 ResourceQuota 를 생성하면 ResourceQuota 의 규칙이 무시될 수 있음
- ResourceQuota를 만들때는 namespace에 pod가 없는 상태일것
- 실무에서는 namespace보다 deployment에 더 많이 설정함







# Controller

![image.png](attachment:85676170-d990-4bba-ac0a-ba38c0aaaba2:image.png)

**Auto Healing**

- 

`terminationGracePeriodSeconds` 옵션을 명시적으로 설정하지 않으면 **기본값은 30초**입니다.

![image.png](attachment:4e6b91e9-1e54-44e9-beaa-9ebde886d929:image.png)

- Controller와 Pod는 label과 selector로 연결함 (서비스처럼)

## 기능

**temsplate**

- 새로 만들 땐 어떻게 만들까에 대한 정의
- 컨트롤러(Deployment, ReplicaSet 등)은 Pod를 직접 적지 않고, template 안에 정의된 내용을 복사해서 새 Pod를 생성

**replicas**

- 몇 개의 파드를 유지할 것인가
- replicas: 0 하면, 만들어져있는 파드들도 삭제됨

### Selector

- 어떤 Pod가 이 컨트롤러(Deployment/ReplicaSet)에 의해 관리될지를 정의하는 조건.
- matchLabels를 일반적으로 사용, matchExpressions는 잘 사용 안함
    - matchExpressions의 기능은, 사전에 어떤 오브젝트가 미리 만들어져있고, 그 오브젝트에 여러 라벨이 붙어있을떄 ,내가 원하는 오브젝트만 세밀하게 선택하기 위해 주로 사용
- matchLabels랑 matchExpressions도 혼합해서 사용 가능

주의사항

- slector에 있는 모든 내용이 template > labels 내용에 포함이 되어야 함

```jsx
# ❌ 경우 1: selector 라벨이 Pod에 없음
selector:
  matchLabels:
    type: web
    ver: v1
template:
  labels:
    type: web
    # ver: v1 누락! - 에러 발생

# ❌ 경우 2: 라벨 값이 다름  
selector:
  matchLabels:
    ver: v1
template:
  labels:
    ver: v2  # v1이 아닌 v2 - 에러 발생
    
    
# ❌ 경우 3: 같은 키 중복 (이건 Yaml 파일 자체가 잘못된 것)
selector:
  matchLabels:
    type: web
    ver: v1       # 이 조건이 문제!
template:
  metadata:
    labels:
      type: web
      ver: v1
      ver: v2     # 불가능! 같은 키에 두 개 값 설정 불가
```

### ReplicaSet

- 지정된 개수 (replicas)의 Pod를 유지
- template 기준으로 Pod 생성, 개수 유지

![image.png](attachment:343d9fea-2a13-42f7-9af1-83f46408fd68:image.png)

## Deployment

- 한 서비스 운영중인데, 이 서비스를 업데이트 해야돼서 재배포를 해야될 때 사용

![image.png](attachment:382435c0-85c2-4119-b0bb-98d6c9d1bde4:image.png)

대표적인 배포방식

### ReCreate

- 업데이트 시 기존 버전의 Pod를 모두 종료한 후 새 버전의 Pod를 생성
    - 업데이트할 Deployment에서 관리되는 Pod들 모두
- 업데이트 시 Deployment가 Pod를 먼저 삭제 (다운타임 발생) 후 새 파드 생성
    - ex) replicas: 3 → 0 → 3 (새 버전)
- 단점 : 다운타임 발생 (일시적인 정지가 가능한 서비스에서만 사용 가능)

### Rolling Update

- 업데이트 시 Deployment가 v2 Pod를 먼저 생성 후 v1 Pod 삭제 (점진적 교체)
    - ex) replicas : 3 (고정), 실제 파드 : 3개 → 4개 → 3개 → 4개 →
- 배포 중간에 추가적인 자원을 요구함
    - v2 Pod가 생성될 때 v1  Pod가 아직 떠있는 상태라 둘다 접근 가능할 수  있음
- 다운타임은 없음
- 업데이트 당시 자원 부족 시 배포가 멈출 수 있음
    - 자원이 부족한 상태에서 업데이트시에는 ReCreate가 더 안전할 수도 있음

### Blue/Green

- `Blue` = 현재 운영 중인 버전, `Green` = 새로 배포할 버전
- Deployment뿐만 아니라 Replica를 관리하는 모든 컨트롤러에서 할 수 있음
- Controller가 v2 Pod만들어서 Service에 연동할 때 일시적으로 자원사용량 2배(파드 2개씩 떠있어서)지만 일시적이어서 서비스에 영향은 없고, 다운타임 없음
- 만약 v2에서 문제가 발생하면 라벨만 v1으로 바꾸면 돼서 롤백이 용이함
    - 문제 없다면 v1은 삭제하면 완료

### Canary (새(bird) 이름)

> Rolling Update : 잠깐 두 버전이 섞여서 서비스됨
Blue/Green : 한 번에 완전히 100% 바뀜, 섞임 없음
> 

![image.png](attachment:361d16a9-48bc-46e5-a70d-59d70e6a9910:image.png)

- Deployment에 정의하는 selector, replicas, template는 직접 파드를 만들어서 관리하는게 아니고, RepicasSet을 만들고 거기에 값을 지정하기 위함
    - 즉 ReplicaSet를 거쳐서 파드를 재어

**revisiosnHistoryLimit**

- 새 버전을 만들 떄 0인 replicaSet을 몇개까지 남겨둘 것인가
    - 오래된 순으로 삭제됨
- 이 옵션 추가 안하면 기본 10개

- Rolling Update가 진행될 때 ReplicaSet은 같은 라벨을 가진 Pod v1에 연결될 수도 있지 않았을까?
    - Deployment가 ReplicaSet을 만들때 라벨셀렉터 말고 추가 라벨셀렉터를 만들기때문에 연결 안됨

![image.png](attachment:c6b32ab9-65c0-41f3-9a44-dd901186b9c5:image.png)

**DemonSet**

- 모든 노드에 파드를 하나씩 배치하는 컨트롤러
- 노드 당 하나보다 더 많은 파드를 만들 수는 없지만, 노드에 파드를 안만들 수는 있음
    - DemonSet의 template에 nodeSelector 옵션을 추가해서 특정 노드들에만 파드를 만들게 설정 가능
- replicas 설정 없음 (노드 수만큼 자동)
- 스케줄링 무시하고 모든 노드에 배치
- 주요 용도
    - 로그 수집기 (Fluentd, Filebeat)
    - 모니터링 에이전트 (Prometheus Node Exporter)
    - 네트워킹 (CNI 플러그인)
    - 보안 에이전트 (Falco)

**Job**

- 한 번만 실행
- 성공하면 파드 종료
    - 파드가 삭제되는건 아니고, 자원을 사용하지 않는 상태
    - 그 이후에 파드 직접 삭제
- 실패하면 재시도 (설정에 따라)

**CronJob**

- 정히진 시간에 반복 실행 (스케줄링)
- Linux cron 문법 사용
- 내부적으로 Job을 사용
- 주요 용도
    - 데이터 백업, 예약 메일, 주기적인 업데이트 확인 등

```jsx
"0 2 * * *"    = 매일 새벽 2시
"*/30 * * * *" = 30분마다
"0 0 * * 0"    = 매주 일요일 자정
```

![image.png](attachment:4cac4029-0c9a-4b2a-98ab-315cf951d784:image.png)

**Allow (default)**

- 자신의 차례떄는 Job이 만들어지고 Pod가 생성
- 겹쳐서 실행 가능

Forbid

- 2분이 됐을 때까지 1분에 생성한 Pod가 종료되지 않으면, 다음과정 스킵됨
    - Pod가 종료됐을 때 해당되는 시간의 스케줄이 돌음

**Replace**

- 새로운 실행이 시작되면, 이전 실행 중이던 Job(과 그 파드들)을 갖에로 종료시키고 새 Job으로 교체
