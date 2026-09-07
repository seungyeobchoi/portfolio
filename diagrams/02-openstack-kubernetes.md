# OpenStack–Kubernetes 내부 구조도

관리 Kubernetes와 사용자에게 제공할 workload Kubernetes는 서로 다른 클러스터입니다. 아래는 논리 구성으로 VM의 개별 물리 호스트 배치를 나타내지 않습니다.

```mermaid
flowchart TB
    U["사용자"]
    subgraph OPENSTACK["OpenStack 가상화 기반"]
        API["OpenStack API<br/>Nova · Neutron · Cinder · Glance"]
        subgraph MGMT["관리 Kubernetes / VM 기반"]
            P["포털 / 자원 신청"]
            CP["관리 control plane"]
            W["관리 worker 2대"]
            OPS["ops worker<br/>CAPI/CAPO 관련 관리 구성"]
        end
        PDB["별도 Portal DB VM"]
        subgraph WORK["시험용 workload Kubernetes / VM 기반"]
            VIP["사설 Octavia API VIP"]
            WCP["control plane 1대"]
            WW["worker 1대"]
            CSI["Cinder CSI"]
        end
    end
    CEPH["Ceph NVMe<br/>공유 블록·이미지 저장소"]
    U --> P
    P --> PDB
    P -->|"1차 볼륨 신청·제공"| API
    P -.->|"Kubernetes 상품 통합 진행 중"| OPS
    OPS -->|"클러스터 자원 관리"| API
    API -->|"VM·네트워크 구성"| WORK
    OPS -->|"관리 API 연결"| VIP
    VIP --> WCP
    WCP --- WW
    CSI -->|"볼륨 요청"| API
    API -->|"스토리지 연동"| CEPH
```

시험용 두 노드 Ready와 Cinder CSI 볼륨 수명주기를 검증했습니다. 포털 통합의 점선은 서비스 전체가 아직 진행 중임을 나타냅니다. API와 Ceph를 연결한 선은 서비스 연동 관계이며 실제 데이터가 모든 OpenStack API를 차례로 통과한다는 뜻은 아닙니다.

Octavia의 사설 API VIP 네트워크와 Amphora 관리망은 서로 다릅니다. 이 도식에는 서비스 API 경로만 표시했습니다.
