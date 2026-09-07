# 하드웨어 구조도

역할 기반 별칭이며 물리 포트 번호와 실제 주소를 생략했습니다. 선은 연결 관계로, 화살표 방향의 통신 제한이나 서버별 처리 속도를 뜻하지 않습니다. 내부 스위치의 10GbE 표기를 GPU 장비 NIC 속도로 일반화하지 않습니다.

```mermaid
flowchart TB
    EXT["기존 외부 L2 스위치<br/>1GbE 외부 접속"]
    INT["독립 내부 스위치<br/>10GbE fabric"]
    subgraph HOSTS["클라우드 물리 서버"]
        A["제어 서버 A<br/>OpenStack 제어·네트워크<br/>Ceph MON·MGR"]
        B["복합 서버 B<br/>OpenStack 제어·컴퓨트·네트워크<br/>Ceph MON·MGR·NVMe OSD"]
        C["복합 서버 C<br/>OpenStack 제어·컴퓨트·네트워크<br/>Ceph MON·MGR·NVMe OSD"]
        S["스토리지 서버<br/>Ceph NVMe OSD"]
    end
    GPU["별도 GPU 장비<br/>단일 노드 Kubernetes<br/>GPU 및 Local SSD"]
    EXT --- A
    EXT --- B
    EXT --- C
    EXT --- S
    EXT --- GPU
    INT --- A
    INT --- B
    INT --- C
    INT --- S
    INT --- GPU
```

- 서버별 CPU·RAM·NIC·SSD 보유 현황을 고려해 역할을 배분했습니다.
- 제어 서버 A의 추가 compute 역할은 후속 검토·설계 범위이므로 현재 compute로 그리지 않았습니다.
- Ceph MON/MGR와 OSD의 위치가 다르며, NVMe OSD는 복합 서버 B·C와 스토리지 서버에 분산됩니다.
- HDD 클러스터나 미편입 후보 서버는 이 도식에 포함하지 않았습니다.
- 서버별 VLAN 멤버십은 같지 않습니다. Octavia 관리 VLAN은 [네트워크 구조도](04-network.md)에서 별도로 표시합니다.
