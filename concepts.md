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



# Pod 상세

# Pod LifeCycle

![image.png](attachment:4e3b2887-3902-4cf8-827b-273f39da138a:image.png)

![image.png](attachment:17e8e613-5574-454e-8b3e-d0e8af8b58b1:image.png)

**Phase**

- Pod의 메인 상태
    - Pending, Running, Succeded, Failed, Unknown
        - Unknown: 노드와 파드 간 통신장애상태

**Pending 상태**

- 파드의 최초 상태는 Pending 상태
- **initContainer**
    - 본 컨테이너가 기동되기 전에 초기화 시켜야될 내용이 있는 경우, initContainer로 초기화 스크립트를 넣을 수 있음
    - 이 스크립트가 본 컨테이너보다 먼저 실행돼서 성공됐거나, 아예 시도를 하지 않으면 : true
    - 실패했으면 : False

**Running 상태**

- Pod가 **노드에 스케줄되고**, **최소 하나의 컨테이너가 시작**되면 Running
- Pod의 상태가 Running이더라도, 내부 컨테이너 상태가 Running 상태가 아닐 수도 있기 때문에 파드 뿐만 아니라 컨테이너의 상태도도 확인해줘야 함
- 컨테이너가 모두 Running일수도 있지만, Waiting상태 + CrashLoopBackOff(기동 중 문제발생하여 재시작) 가 Reason으로 나올 수도 잇음
    - 이 때는 ContainerReady : False, Ready: False

**Failed or Succeded 상태**

- 모든 컨테이너가 Terminated 상태가 되고, 이 중 하나라도 Error면  Pod는 Failed 상태
    - 
- Job이나 CronJob으로 실행된 Pod가 자신의 작업을 끝마치면 Failed나 Succeded 상태가 됨
    - Container중 하나라도 Error면 Falied
    - Container들 모두 Completed면 Succeded
    - 두 케이스 모두 ContainerReady랑 Ready는 모두 False

![image.png](attachment:e5e3fe0f-1626-4578-914b-b9109ed6d1a6:image.png)

**ReadinessProve**

- **App 구동 순간**에 트래픽 실패를 없앰
    - App이 구동되기 전까지는 Service와 연결되지 않게 해서, 새 Pod가  Running상태지만 아직 App이 구동되지 않아서 사용자가 접근했을 때 문제가 될 상황을 방지해줌 - 기존 파드로 연결되도록함
    - Pod가 Running 상태인데, App이 아직 Booting 중이라, 사용자가 접근했는데, 정상화면을 볼 수 없는 문제를 방지해줌

**LivenessProbe**

- **App 장애시** 지속적인 트래픽 실패를 없앰
    - 예를 들어 톰캣은 돌고있지만, 그 위에 띄워진 앱에 문제가 생겨 502같은 에러가 발생했을 떄, Pod는 Running이지만 안쪽에는 장애상태 → LivenessProbe 설정을 해주면 App에 문제가 있을 때 Pod를 재실행하도록 해줌

## QoS (Quality of Service)

![image.png](attachment:066ee814-6088-4404-b052-670406eda559:image.png)

> 메모리 부족 시 삭제 순서
- BestEffort Pod → Burstable Pod → Guaranteed Pod
> 

**Quaranteed - 최고 등급**

- 모든 Container에 Request와 Limit가 설정
- Request와 Limit에는 Memory와 CPU가 모두 설정
- 각 Container 내의 Memeory와 CPU의 Request와 Limit의 값이 같음

**Burstable - 중간 등급**

- **Quaranteed와 BestEfffort 사이**
    - requests < limits 이거나, request만 설정되어있거나, 파드에서 하나는 컨테이너가 다 설정되어있지만, 나머지 하나의 컨테이너는 **BestEfffort**일때
- **Quaranteed는 아니고, 뭔가 하나씩 없는**
- **Burstable 등급에 여러 파드가 있는 경우 먼저 삭제되는 순서?**
    - OOM (Out of Memory) Score
        - Request 대비 실제 메모리 사용량 %가 더 높은 Pod가 먼저 제거됨

**BestEfffort - 최하 등급**

- 어떤 Container 내에도 Request와 Limit가 미설정

# Node Schduling

> 파드는 기본적으로 스케줄러에 의해 노드에 할당된다
> 

![image.png](attachment:3cf40551-4d55-48a2-9823-54cb9f5988b8:image.png)

**NodeName**

- 스케줄러와 상관없이 해당 노드에 할당됨
- 운영 상황에서는 노드가 추가되고 삭제되면서 노드명이 바뀌기 때문에 NodeName은 잘 사용안함

**NodeSelector**

- 파드에 키:값을 달면 같은 라벨을 가진 노드에 할당됨
    - 만약 두 노드 이상이면, 더 자원이 많은 노드에 할당됨
    - 만약 매칭이되는 노드가 없으면 아무 노드에도 할당되지 않음

**NodeAffinity**

- NodeSelector를 보완해서 사용할 수 있음
- matchExpressions
    - key를 설정하면 스케줄러가 해당 key가 있는 노드들 중 자원이 많은 쪽으로 배치
- Required vs Preferred
    - Required는 엄격
        - **NodeSelector랑 비슷**
    - Preferred
        - 같은 key를 가진 노드를 선호하지만, 없다면 다른 노드에 할당

**PodAffinity**

- Pod가 다른 Pod의 위치를 보고 노드를 선택
    - 다른 Pod의 라벨 기준

|  |  |  |
| --- | --- | --- |
|  |  |  |
|  |  |  |

**Taint, Toleration**

- 파드에 Toleration이 있다고 해서, Taint가 있는 노드에 무조건 배치되는게 아니라, Pod가 해당 노드에 스케줄링 됐을 때 노드에 배치될 수 있는 조건이 되는것임.
    - 즉, Toleration있어도 다른 노드에 배치될 수 있음
    - 원하는 노드에 배치하려면 ndeSelector 옵션을 추가해야함
- PreferNoSchedule
    - 다른 파드들이 이 노드에 스케줄 되지 않지만, 정 할당될 곳 없으면 할당됨
- NoSchedule (effect)
    - 다른 파드들이 이 노드에 스케줄 되지 않음
- NoExecute
    - NoExecute 효과를 단 Taint를 노드에 설정하면, 노드에 안맞는 기존 파드가 삭제됨
        - 이미 노드에 파드가 있는 상태에서도 삭제됨 (NoSchedule은 안됨)
        - 파드가 삭제되지 않도록 하려면 Pod를 만들 때, Toleration (effect: NoExecute)설정을 추가해야됨
        - 이미 생성된 파드에는 **Toleration을 나중에 추가할 수 없음**

- NodeAffinity, PodAffinity, Taint의 NoSchedule 공통점
    - 파드가 최초 노드를 선택할때만 고려되는 조건
    - 이미 파드가 노드에 안착된 상태에서는 노드 라벨 등이 바뀐다고 해서 파드가 삭제되지 않음
    - but, NoExecute는 삭제됨



- Kubernetes Cluster의 IP 대역대는 10 또는 20인가?
    - Pod는 20번대, Service는 10번대
- 내부망에 연결되어 있는 기기들이 cluster 안에 접근 불가
    - 연결하고 싶으면 NodePort Service를 만들어야함
    - 서버(노드)의 IP와 Port를 알면 안에있는 서비스에 접근 가능해짐
- Cloude환경에서는, 쿠버네티스에서 LoadBalancer 타입의 서버를 만들었을때, NodePort가 생성되면 서 이 Port에 로드밸런서가 연결되고,  외부망에 있는 사람들은 cloude (로드벨런서) IP를 알면 접근이 가능함
    - 실제로는 클라우드 프로바이더가 로드밸런서 생성/관리

![image.png](attachment:27e08296-1ef0-4c41-bb43-efc0868b4266:image.png)

**Pod가 Service나 Pod에 접근하는 방법**

- 만약 동시 배포될 때, Pod A가  Pod B로 접근하고 싶지만, IP는 생성 시 동적으로 할당 되기 때문에 미리 알 수가 없다면
    - Headless Service, DNS Server가 필요함

**Pod가 외부 특정 사이트에 접근해서 데이터를 가져와야 할 때**

- 접근 주소를 변경해야 되는 상황에서는
    - 서비스의 ExternalName을 이용해서 외부연결을 Pod의 수정없이 변경할 수 있음

![image.png](attachment:eb4ccccf-8a6c-4d6f-80dd-31aa8ac0a343:image.png)

DNS Server에는 서비스 도메인 이름과 IP가 저장되어있어서, 예를 들어 파드가 server1에 대한 도메인 이름을 지레아면 해당 IP를 알려줌

- DNS 매커니즘 상 DNS Server에 해당 DNS Server에 도메인 정보 없다면 상위 DNS Server를 찾게 됨
    - kubernetes Cluster → Internal Network → External Network
    - pod가 접근할 IP주소를 몰라도 DNS Server를 이용해 알아낼 수 있음

![image.png](attachment:7c5c4eac-69ae-4d84-81ff-fce692a782c1:image.png)

**Cluster IP**

- Service의 이름이 도메인으로 자동 등록되기 때문에, Pod에서는 Service의 이름으로 호출 가능
- 또한 쿠버네티스에 설치된 CoreDNS를 통해 Service의 이름으로 Service IP를 확인 가능
- Service FQDN
    - <service-name>.<namespace-name>.svc.cluseter.local
    - 같은 namespace 상에서는 <service-name>만, 타 namespace를 호출 시 <service.name> . <namespace-name>까지 입력 필요
- Pod FQDN : <pod-ip> . <namespace-name> . pod . cluster . local (실 사용 불가)

**Headless**

- Headless Service를 만들면 Service의 IP는 할당되지 않음
    - yaml에서 clusterIP: None
- 그래서 DNS에 Service 호출 시 Service IP는 없고, 해당 Service에 연결된 Pod의 IP들만 반환함
- 또한 Headless Service를 통해 Pod를 Domain 이름으로 호출 가능
- Pod FQDN
    - <pod-name>.<service-name>.<namespace-name>.svc.cluster.local

> 단순히 서비스에만 연결할 때는 ClusterIP로 서비스 만들어도 문제없음
근데 Pod가 다른 Service에 연결된 Pod에 접근하고 싶다면, Headless Service로 만들어야함
> 

**EndPoint**

- 쿠버네티스는 서비스와 파드가 연결될때(labels, selector) Endpoint를 만들어서 관리함
    - Service이름과 동일한 이름의 Endpoint와 , 안에는 Pod의 IP,port정보를 가지고 있음
- 사용자는 Pod와 Service에 label selector없이 직접 Endpoint를 만들어서 연결할 수 있음

**ExternalName**

- externalName 속성에 도메인 정보를 넣을 수 있음
- DNS cache가 내부 외부정보를 찾아서 IP주소를 알아냄



# StatefulSet

![image.png](attachment:c566b6c8-5cc9-4024-9a4d-d1bb26ec52b9:image.png)

**Stateless Application**

- 주로 Web Server (APACHE, NGINX, …)
- stateless는 앱이 여러개 배포되더라도 똑같은 서비스 역할을 함
    - 볼륨이 반드시 필요하진 않음

**StatefulSet Application**

- 주로 Database
- StatefulSet은 여러개 배포되더라도 각각 본인의 역할이 있음
    - Primary
    - 메인
    - Secondary
    - Arbiter
        - Primary가 죽으면 감지해서, Secondary가 Primary역할을 하게 해줌
- 각 앱마다 볼륨을 별도로 써야함

### ReplicaSet vs StatefulSet

![image.png](attachment:890cd447-e258-4a01-b6c7-6f9a642ddc96:image.png)

|  | ReplicaSet | StatefulSet |
| --- | --- | --- |
| 이름 | Random 이름으로 생성 | 순차적인 Index 이름으로 생성 |
| replicas를 3으로 늘리면 | 동시에 랜덤이름으로 생성됨 | 순차적으로 index 이름으로 생성됨 |
| 하나의 파드가 삭제되면 | 새이름으로 생성 | 기존이름으로 생성됨 |
| replicas를 0으로 바꾸면 | 동시삭제됨 | 순차삭제 (Index가 높은 Pod부터) |
- statefulSet + Headless Service
    - **StatefulSet**: Pod 이름이 예측 가능
    - **Headless Service**: 각 Pod에 직접 도메인 생성
    - 즉, 특정 Pod를 골라서 직접 접근할 수 있음

# Authentication

> Kubernetes의 모든 작업은 API Server를 통해 진행됨
그래서 여기에 접근하기 위한 인증이 필요
> 

![image.png](attachment:68adaec7-3e80-40ef-8e0d-716e40fce1d9:image.png)

**X509 Client Certs**

- 외부에서 Cluster로 API를 보내는 방법으로는 Client key와 crt파일을 이용해서 API Server에 https로 접근
- 그리고 Cluster 관리자가 kubectl을 이용해서 내부 서버에 Proxy를 만들고 http로 접근

- 사용자가 kubernetes API Server로 접근하려면, kubeconfig(쿠버네티스 설치 시 생성) 파일에 있는 인증서(Client key, Client crt)를 복사해서 가져오면 됨
- kubectl 설치 시에 kubeconfig 파일을 통채로 복사하는 과정도 있음
    - 이걸로 kubectl에서 API Server에 접근할 수 있는 권한이 생김
- kubectl에 accept-hosts 옵션을 통해서 Proxy를 열어두면 외부에서도 http로 접근 가능
    - kubectl이 인증서를 가지고 있기때문(kubeconfig 복사해온)

**kubectl (API Server와 통신하는 클라이언트)**

- **kubectl config 명령어를 통해 여러 클러스터에 접근 가능함**
    - 각 클러스터의 kebeconfig 인증서가 kubectl에도 있어야함
- kubeconfig
    - 파일 안에는 clusters 항목으로 클러스터 등록 가능
        - name, url, CA
    - users라는 항목으로 사용자 등록 가능
        - name, crt, key
    - contexts라는 항목으로 clusters와 users를 연결 가능
        - nmae, cluster, user

**Service Account**

- Pod가 API Server에 접근할 때 사용하는 계정
- k8s 클러스터에서 namespace 만들면 디폴트라는 이름의 ServiceAccount가 자동으로 만들어짐
    - ServiceAccount에는 Secret이 달려있는데, 여기에는 CA crt정보와 token값이 들어있음
    - 파드를 만들면 이 ServiceAccount가 연결되고, Pod는 token값을 가지고 API Server에 접근 가능
- k8s 1.23이전 버전에서는 자동으로 Secret 생성됐으나, 이후 버전에서는 수동으로 생성해야함

# Authorization

![image.png](attachment:73bfc3d3-527b-4251-8846-b2369fbffbad:image.png)

**RBAC (Role Based Access Control)**

- 역할 기반으로 권한을 부여
- **"누가 무엇을 할 수 있는지"** 권한을 관리하는 시스템
- 핵심 개념: 권한 = Role + RoleBinding
    - Cluster 자원 (Node, PV, namespace 등)에 접근하고 싶다면 ClusterRole 필요
        - Namespace 내 자원 (Pod, Service 등)에 접근하고 싶다면 Role 필요
    - **Role**
        - 네임스페이스 내에서 할 수 있는 작업들을 정의
        - ‘무엇을 할 수 있는가’를 정의
        - 여러개를 만들 수 있음
        
        ```jsx
        apiVersion: rbac.authorization.k8s.io/v1
        kind: Role
        metadata:
          namespace: default
          name: pod-reader
        rules:
        - apiGroups: [""]          # core API group
          resources: ["pods"]      # 대상 리소스
          verbs: ["get", "list"]   # 허용된 동작
        
        - apiGroups: ["apps"]
          resources: ["deployments"]
          verbs: ["get", "list", "create", "update"]
        ```
        
    - RoleBinding
        - Role과 ServiceAccount를 연결
        - ‘누가 이 역할을 가질 것인가’
        
        ```jsx
        apiVersion: rbac.authorization.k8s.io/v1
        kind: RoleBinding
        metadata:
          name: read-pods-binding
          namespace: default
        subjects:              # 권한을 받을 주체들
        - kind: User
          name: jane
          apiGroup: rbac.authorization.k8s.io
        - kind: ServiceAccount
          name: my-sa
          namespace: default
        roleRef:               # 연결할 Role
          kind: Role
          name: pod-reader
          apiGroup: rbac.authorization.k8s.io
        ```
        
    - ClusterRole
        - 클러스터 전체 권한
    - ClusterRoleBinding

RoleBinding할 떄 내 namespace안의 Role이 아닌, ClusterRole을 지정할 수 있는데

- 이때는 SA가 클러스터 자원에는 접근하지 못하고, 자신의 namespace안에있는 자원에만 접근 가능
    - 이러면, 그냥 namespace안에 Role만드는거랑 무슨 차이냐?
    - 모든 namespace마다 똑같은 role를 부여하고 관리하면, 만약 role을 변경해야 될 때 모든 role을 수정해야되는데, clusterRole 하나만 만들어두고, 모든 namespace에 있는 RoleBinding이 ClusterRole 하나만 보고 있다면 한번에 관리가 가능해짐
        - 즉, 모든 네임스페이스에서 같은 권한을 만들어서 관리해야될 때 유용