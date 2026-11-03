# 검증 결과와 진행 범위

[README](../README.md#구조도와-상세-문서) · 프로젝트 소개: [클라우드](cloud-platform.md) · [AI 챗봇](ai-chatbot.md) · [FDP 연구](fdp-research.md)

작업·검증 기록에서 확인한 구축·구현 성과와 실행 결과를 정리했습니다. Windows Standard(10월 28일)·Development(31일), 원격 Linux 사용자 시험(11월 2일)과 Registry 격리 검증을 각각의 환경·범위로 기록합니다. 과거 실패·당시 미완료와 후속 성공을 날짜별로 구분하며, 문서 정리일을 현재 운영 재검증일로 사용하지 않습니다. AI 데이터 기반의 기존 기록은 해당 실행 범위로 유지합니다.

## 확인한 구축과 실행 결과

| 대상 | 시점 | 확인한 결과 | 적용 범위 |
|---|---|---|---|
| 클라우드와 볼륨 포털 기반 | 2026-07-31까지 | OpenStack·Ceph와 Cinder Volume 신청·제공 기반 구축 | 첫 번째 볼륨 제공 단계 |
| 볼륨 포털 기능 | 검증 일자 미기재 | 테스트 계정의 자원 신청·생성·삭제·스냅샷·복원과 관리자 계정의 동작 점검 | 기능 검증 완료, 실제 사용자 도입 사례 없음 |
| Octavia | 2026-08-14 | 관리망 연결, 시험용 LB 생성과 ACTIVE·ONLINE 상태, 삭제 후 임시 자원 정리 | 해당 관리 경로와 시험용 자원 |
| CAPI/CAPO pilot | 2026-08-17 | control plane 1대·worker 1대 Ready, Cinder CSI 볼륨 생성·사용·삭제 | 시험용 클러스터 기능 검증 |
| Provisioner와 Finalizer | 2026-08-26 | 각각 단일 replica 실행 검증 | 해당 worker와 수정 사항 |
| 포털 worker 다중 replica | 2026-08-28 | Provisioner·Finalizer·FBS·Dispatcher 각각 replicas·Ready·Available 2, legacy monolith 0; replica-two 검증 통과 | 당시 정확히 2 replica 구성 |
| GPU Kubernetes | 2026-09-01 | 기본 클러스터, 단일 노드 Pod·Service·DNS 통신, static/dynamic Local PV 검증 | 챗봇 실행 기반의 기능 검증 완료 |
| AI 데이터 기반 | 2026-10-12~13 | 내부 DNS 해석, etcd 3멤버 TLS·건강성·관리 인증, 두 Patroni용 계정의 제한된 키 접근 검증 | 10월 12~13일 당시 etcd 구성요소와 VM 내부 시험 경로; 이후 DB 실증은 아래 날짜별 후속 기록으로 구분 |
| G5 데이터 기반 마감 | 2026-10-21 | HA 시나리오·DGX 인증/SQL 후 임시 DB·계정·HBA 정리, 양쪽 PG·etcd·Octavia 최종 대조 | 합의된 G5 범위 완료. 백업/PITR·전체 AI 서비스 인수는 후속 |
| Standard 사용자 종단 | 2026-10-28 | CP 인증·제품 배포 후 Windows 접속·앱·권한 거절·RO 볼륨·자연 만료 자동 갱신·정상 삭제 | 시험 계정·단일 Standard. 다른 환경·장기 안정성·현재 운영 상태는 별도 |
| Development 사용자 종단 | 2026-10-31 | 설정 적용 후 Windows 접속·앱·권한 거절 15검사, 정상 삭제·접속 회수와 시험 자원 정리 대조 | 시험 계정·단일 2노드 Development. 볼륨 재시험·자연 만료 갱신·HA·다른 환경·현재 운영은 미검증 |
| Linux 사용자·callback 대체 | 2026-11-02 | 원격 Linux cold OIDC·앱/권한 13검사·자연 만료 갱신, 별도 포트 대체 8검사·정상 삭제·접속 회수·임시 설정 원복 | 시험 계정·원격 서버·Windows 브라우저/SSH 포워딩·수동 keyring·임시 이름 매핑. 독립 Cloud 잔류 자원 조회·WSL·일반 환경 미검증 |
| Registry 격리 통합 | 2026-11-01~02 | 인증·준비 상태 오류/복구, 실제 Registry의 업로드·읽기·내용 정합성·쓰기 거절 및 시험 자원 정리 | 합성 작은 HTTP API fixture. 운영 인증 전환·실제 CRI pull·Kaniko·성능·영구 복구 미검증 |
| FDP 연구 | 석사 연구 | 커널 인터페이스 구현, 실제 SSD의 데이터 배치 확인과 WAF·TPM 비교 실험 완료 | 연구 환경의 커널 인터페이스와 배치 전략 |

네트워크와 자원의 기능 검증은 전체 서비스의 무중단·장기 안정성 또는 처리량 향상 검증과 구분합니다.

## 포털 worker의 변경과 검증

8월 26일 기록에서는 Finalizer 초기화 과정의 기존 factory 호출을 제거한 뒤, 동일 Pod가 여섯 차례 연속 Ready/Running을 유지하고 재시작 횟수가 늘지 않는지 확인했습니다. Pod에 마운트된 소스와 수정본의 일치, 변경 전후 CAPI 리소스 그래프의 spec 유지도 확인했습니다. Provisioner와 Finalizer는 각각 단일 replica 실행 범위에서 검증했습니다.

후속 기록에는 FBS 단일 replica 활성화·상태 조정(reconciliation), Dispatcher 활성화·2 replica 구성 작업, 표준 클러스터 생성 E2E 관련 소스 변경이 있습니다. 2026-08-28 실행 출력에서는 Provisioner·Finalizer·FBS·Dispatcher 각각의 replicas·Ready·Available이 모두 2이고 legacy monolith는 0인 상태를 확인했습니다. 모든 replica-two 단계가 종료되었으며 최종 결과와 불변성 검증 모두 통과했습니다.

이 검증은 당시 정확히 2 replica 구성에 한정되며, 해당 기록에서 production 사용은 미승인 상태였습니다. 현재 배포 상태, production 준비 완료나 포털 요청부터 클러스터 준비·삭제·자원 회수까지의 전체 E2E 성공을 증명하지 않습니다.

## 서비스 통합 검증 이력과 원인 분석

2026-09-30 통합 검증에서는 생성 단계 성공 후 Finalizer 단계 실패를 확인했고, 전체 요청은 실패로 기록됐습니다. 당시 Finalizer의 replicas·Ready·Available은 각각 2였습니다. 따라서 worker의 replica 상태와 서비스 요청의 성공 여부를 별도로 구분합니다.

후속 소스 분석에서는 API 도달성 사전조건이 연결 상태를 검사하는 경로를 확인했습니다. 이는 오류가 발생한 검사 경로에 대한 분석이며, 네트워크 장애의 근본 원인을 확정하거나 현재 장애 상태를 나타내는 결과는 아닙니다.

<a id="standard-e2e-20261011"></a>
## 2026-10-11 Standard 생성·삭제 종단 재검증

9월 30일 실패와 이후 수정·복구 이력을 보존한 뒤, 10월 11일에는 수정된 당시 버전에서 별도 새 Standard 1개를 재검증했습니다. control plane 1대·worker 2대이며, 생성·마무리 작업이 최초 세대에서 각각 한 번 성공하고 결과가 반영됐습니다. 00:32 KST에 3노드 Ready와 API·CNI·CCM·CSI·스토리지 건강을 확인했고 UI ACTIVE·성공 상태가 추가 작업이나 상태 역행 없이 유지됐습니다.

00:37 KST까지 같은 시험 클러스터의 정상 삭제, CAPI 그래프와 하위 자원 부재, 진행 작업·예약 0, 기존 클러스터 식별자·원본 볼륨·조회한 Cloud 자원 ID·Nova/Neutron 사용량의 사전 기준 일치를 확인했습니다. 최종 31개 대조는 모두 통과했고 operator 복구·수동 상태 변경·재배정 없이 완료했습니다.

이는 **당시 버전의 단일 Standard 생성·상태 유지·정상 삭제·자원 회수 시험**입니다. 기존 볼륨 재연결·파일 읽기·쓰기 거절 시험은 이 재검증에서 반복하지 않았습니다. 모든 프로필·동시/대규모 요청·모든 장애 복구나 후속 변경본의 사용자 E2E를 이 기록만으로 보장하지 않습니다. 이후의 실제 사용자 시험은 [10월 28일 결과](#standard-user-e2e-20261028)로 구분합니다.

<a id="standard-user-e2e-20261028"></a>
## 2026-10-28 배포 후 Standard 사용자 종단 검증

대상은 기존 시험 계정의 Windows 클라이언트와 새 Standard 1개(control plane 1대·worker 2대)입니다. 당시 10GiB 루트 디스크·단일 API LB 구성을 대조했습니다. 사용자 도입이나 모든 환경의 운영 인수를 뜻하지 않습니다.

### 구현·격리 검사와 실제 배포

10월 21일에는 사용자 로그인 뒤 API의 내부 인증 DNS 해석 실패로 kubectl이 401에서 중단됐습니다. 관리망 출발 issuer 확인과 사용자 CP 출발 인증 경로가 다름을 확인한 뒤 CP 한정 DNS bootstrap·공개 인증 관측·최초 신뢰/현재 보안/가용성 분리 코드를 준비했습니다. 22일의 추가 DB 구조·회귀/격리 검사·프론트 canary는 당시 코드 전체의 운영 배포나 사용자 복구 완료가 아니었습니다.

28일에는 호환 reader를 먼저 적용하고 backend·provisioner·finalizer·publisher의 실제 모듈 대응과 기동을 대조했습니다. 새 요청용 CP DNS 정책·준비 증거와 최신 프론트 이미지도 적용하고 일반/관리자 UI를 확인했습니다. 아래 결과는 이 배포 후 새 요청의 실행이며 과거 실패 작업을 재큐하거나 상태를 수동 성공으로 바꾼 기록이 아닙니다.

### 실제 시험 결과

| 단계 | 확인한 결과 | 구분할 범위 |
|---|---|---|
| 생성·준비 | 최초 세대의 생성·마무리 성공, 3노드 Ready·요청 ACTIVE, CP 인증·준비 증거와 두 relay ACK | ACK만으로 사용자 명령 성공을 대신하지 않고 아래 실제 경로를 별도 시험 |
| 접속·인증 | Portal 서명 ZIP 다운로드·독립 공개키 검증·등록, 실제 로그인·PKCE·keyring, 동시 kubectl 성공 | 시험 계정·Windows·해당 클러스터 |
| 앱·권한 | 앱 Ready와 DNS·exec·log·port-forward 성공. nodes·시스템 Secret·RBAC·검사용 자원·PVC·특권 Pod의 시험한 금지 요청 거절 | 모든 API·모든 권한 조합 검증은 아님 |
| 읽기 전용 볼륨 | 사용자 UI 연결, 실제 Pod의 디렉터리 읽기·EROFS 쓰기 거절, 정상 해제와 관련 자원 부재·원본 볼륨 available/첨부 없음 복귀 | 파일 내용·이름·해시를 출력하지 않았으며 전체 파일 내용·해시 보존 검증이나 RW 지원 시험이 아님 |
| token 갱신 | 사용자 요청 강제 갱신과 별도로 300초 ID token 자연 만료 뒤 강제 옵션·재로그인 입력 없이 자동 갱신·kubectl 재접속 | 취소 전달·전체 캐시 초기화 경합·모든 인증 장애 복구와 별도 |
| 정상 삭제·정리 | CREATE/DELETE 완료, CAPI 하위 자원·정확히 대조한 Cloud VM/LB/망/전용 정책의 부재, 접속 허가 만료. 원본 볼륨 상태·공통 입구·중계·경로 보존 | 이름 검색만으로 정리를 추정하지 않음. 현재 시험 클러스터는 삭제돼 접속 대상이 아님 |
| 시험 세션 종료 | 시험 Pod의 UID 조건 삭제·부재, 해당 로그인 세션·소유 keyring 항목·하위 프로세스 종료 | 전역 설정·다른 캐시·기존 키 세션을 정리한 것이 아님 |

별도 공개 코드 저장소의 준비 상태 파일은 10월 18일 기록입니다. 당시 다운로드 통합 미검증 표시와 이번 Windows 단일 시험은 시점과 대상을 구분합니다. Ubuntu/WSL 고유 연동의 미검증 경계는 유지합니다.

### 실패 이력과 남은 검증

직전 시험의 초기 권한 설치 검증은 실패했습니다. 후속 동일 정책의 server dry-run 성공만으로 최초 실패를 전파 지연으로 확정하지 않았습니다. 최초 설치에만 짧은 엄격 재확인을 추가했지만, 최종 시험은 첫 시도에서 통과했으므로 재확인이 성공에 필요했다고 단정하지 않습니다. 이전 실패·소비 이력은 보존했습니다.

Development 2노드·Ubuntu/WSL·계정 정지의 실제 영향 범위와 시험·일반 신규 tenant의 최초 망 연결·장기 로그/이미지 증가 및 업그레이드 여유는 후속입니다. 10GiB는 이번 부팅·앱·CSI 시험을 통과한 구성이며 전체 프로필의 최종 기본값이나 장기 충분성을 뜻하지 않습니다. 이 결과는 당시 저장된 검증이며 현재 운영 상태·전체 Kubernetes 서비스 완료·AI 전체 E2E로 확대하지 않습니다.

당시 후속으로 남긴 Development의 별도 시험은 아래 10월 31일 기록을 따릅니다. Standard의 볼륨·자연 만료 갱신 결과를 Development에 적용한 것은 아닙니다.

<a id="development-user-e2e-20261031"></a>
## 2026-10-31 Development 사용자 종단 검증

시험 계정의 Windows 클라이언트와 새 Development 1개(control plane 1대·worker 1대)를 대상으로 생성부터 정상 삭제까지 확인했습니다. 두 VM은 같은 물리 호스트에 배치됐으므로 호스트 장애에 대한 HA 검증이 아닙니다.

### 준비·실제 적용과 사용자 시험의 구분

초기 사전 확인·설정 변경의 서버 dry-run은 준비 기록입니다. 이후 별도 재개 구간에서 backend의 접수 flag 두 개를 실제 적용하고 배포 설정과 기존 Python 소스 36개의 일치를 확인했습니다. 새 Python 기능 구현·DB 구조·전역 Cloud 정책 변경으로 계산하지 않습니다.

| 단계 | 확인한 결과 | 구분할 범위 |
|---|---|---|
| 생성·준비 | 실제 UI 신청, CP·worker 2노드와 단일 API LB 확인, 접속 준비와 두 relay ACK | ACK를 사용자 접속 성공으로 대신하지 않음 |
| 접속·앱·권한 | 서명 ZIP 다운로드·독립 공개키 검증·등록, OIDC·PKCE·keyring·동시 캐시 kubectl, 앱 Ready·DNS·exec·log·port-forward와 시험한 금지 요청 거절 등 15검사 | 시험 계정·Windows·해당 Development. 모든 API·모든 권한 조합 검증 아님 |
| 정상 삭제·접속 회수 | 정상 UI DELETE, 생성·삭제 요청 모두 DELETED, 대조한 CAPI 하위 5종 모두 0, 경로 tombstone·유효 ACK 0·접속 파일 제공 거절 | 과거 준비 상태 행은 보존된 기록이며 현재 잔류 자원으로 해석하지 않음 |
| Cloud 자원 정리 | 사용자 OpenStack 조회의 시험 VM·LB·network·subnet·전용 보안그룹·Amphora compute 부재와 공통 입구·공유 router 경로 보존 대조 | 시험 대상 범위의 정리. 별도 port inventory와 원본 볼륨 상태는 조회하지 않음 |
| 시험 세션 종료 | 시험 Pod·해당 keyring·로그인 세션·하위 프로세스 정리 | 전역 캐시·다른 사용자 자료 정리가 아님 |

이번 Development 시험은 원본 볼륨을 사용·변경하지 않았고 볼륨 재시험과 token 자연 만료 갱신은 포함하지 않았습니다. 이 두 항목의 실제 결과는 10월 28일 Standard 범위로만 유지합니다. 당시 10GiB 루트 디스크 여유는 CP 약 3.25GiB·worker 약 3.14GiB였지만 장기 이미지·로그 증가나 업그레이드 여유를 보장하지 않습니다.

Ubuntu/WSL 실제 OIDC·keyring·클라이언트, 계정 정지·남은 lifecycle·일반 신규 tenant의 최초 망·모든 장애·장기 안정성과 현재 운영 상태는 미검증입니다. Linux 환경 확인과 세션 해제 방식 선택은 준비 단계이며 실제 Linux 사용자 접속 성공이 아닙니다. AI 전체 E2E나 전체 서비스 인수 완료로 확대하지 않습니다.

위 10월 31일의 Linux 미완료·자연 만료 제외는 당시 Windows 시험의 범위입니다. 이후 Linux에서 확인한 항목은 아래 별도 결과를 따르며, Windows 시험을 다시 수행했다고 계산하지 않습니다.

<a id="linux-client-e2e-20261102"></a>
## 2026-11-02 원격 Linux 사용자 시험과 callback 대체

대상은 시험 계정과 원격 Linux 서버 한 대, 별도 Development 시험 클러스터입니다. CLI·기존 암호화 keyring의 수동 해제는 Linux에서, 인증 브라우저는 Windows에서 SSH 포워딩으로 연계했습니다. 임시 인증 이름 매핑을 사용하고 각 구간 종료 시 원본 hash로 복구했습니다. Linux 데스크톱·WSL·일반 배포 환경 전체의 성공이 아닙니다.

### 실제 성공과 시도 이력

최초 두 시도는 keyring 입력 대기 중 종료돼 kubectl·브라우저 인증 전에 중단됐습니다. 입력 대기 시간과 도구 제한은 대조했지만 초기 예외 원인 전체를 확정하지 않았습니다. 세 번째는 cold 로그인 후 시험 도구가 keyring 속성을 정확히 두 개로 가정해 cache 확인·정리를 끝내지 못했습니다. 표준 Generic 속성·응답 형식 호환을 보정하면서 필수 소유 속성·서명·사용자 검증은 유지했고 소유 cache를 복구·정리했습니다. 공식 kubelogin·Portal 제품·IdP·PAM을 교체한 것이 아닙니다.

최종 정상 시험과 이후 callback 대체 시험은 별개입니다.

| 구간 | 실제 확인한 결과 | 정리·한계 |
|---|---|---|
| 18:28 Linux 정상 경로 | 서명 파일 검증·등록 후 cold kubectl·OIDC/PKCE·keyring 소유 cache, 동시 캐시 명령 2개, 앱·권한 13검사, 300초 ID token 자연 만료 후 동시 갱신 명령 2개 | 앱 Pod·소유 cache·로그인 세션·하위 프로세스·시험 provider와 브라우저 정리. 토큰 반출 없음 |
| 22:57 callback 대체 | 별도 새 Development에서 기본 callback 8000 점유 중 공식 도구가 18000을 사용, cold 로그인·서명 사용자 cache 등 8검사 | 소유 소켓·cache·로그아웃·provider·브라우저 정리. 앱·캐시 동시 명령·자연 만료 갱신은 이 구간에서 반복하지 않음 |
| 각 클러스터 정상 삭제 | 정상 Portal UI DELETE 뒤 생성/삭제 요청 DELETED, CAPI 하위 5종 각각 0, route tombstone·유효 ACK 0·실제 다운로드 guard 거절·사용자 UI DELETED | 독립 OpenStack 관리자 API/CLI의 잔류 자원 목록은 조회하지 않음. CAPI 0을 Cloud 자원 전체 부재 조회로 표현하지 않음 |
| 임시 환경 원복 | 각 시험의 소유 hosts 행 제거·원본 hash 일치, 소유 cache·세션 종료 | 기존 keyring·등록 파일·공개 support는 보존. 다른 사용자·전역 설정 정리가 아님 |

두 callback 포트 동시 점유·취소·추가 네트워크/저장 실패, 계정 정지·일반 신규 tenant 망, 볼륨 재시험·HA·장기 10GiB·현재 운영과 전체 서비스 인수는 미검증입니다. 임시 이름 매핑의 성공은 정식 DNS·공개 HTTPS 개통을 증명하지 않습니다.

<a id="registry-isolated-20261101-02"></a>
## 2026-11-01~02 Registry 인증·준비 상태의 격리 검증

실제 Kubernetes의 임시 Pod·합성 인증서와 계정으로 시험했습니다. 운영 Registry·Secret·소비자 workload를 전환한 배포 검증이 아닙니다.

| 시험 | 확인한 결과 | 구분할 한계 |
|---|---|---|
| Nginx 인증·접근 경계 | 비특권·read-only root·token mount 없는 시험 Nginx에서 합성 계정의 허용/거절, backend 전달 시 Authorization 제거 확인 | 초기 backend는 합성 응답 서버. 실제 Registry 프로토콜과 별도 |
| TLS와 준비 상태 | 잘못된 시험 cert/key에서 TLS 실패에도 HTTP 생존·기존 준비 응답이 200인 한계 발견. 준비 검사를 CA/이름 검증 HTTPS와 backend 응답으로 보정한 뒤 정상 Ready→오류 NotReady→복원 Ready, 재시작 0 확인 | 이전 인증서 자동 유지·자동 복구나 운영 인증서 갱신 전체의 보장 아님 |
| 실제 Registry API | 같은 임시 Pod의 고정 Registry 2.8.3와 Nginx에서 작은 blob 2개의 단계적 PATCH/PUT·manifest 게시, 읽기 GET/HEAD와 실제 bytes/digest 일치 | HTTP API의 작은 합성 fixture. 독립 CRI/containerd pull·Kaniko build·대용량/동시성 성능 미검증 |
| 역할·삭제 동작 | 읽기 계정의 POST/PATCH/PUT/DELETE가 401. 게시자의 manifest 삭제는 backend의 405 UNSUPPORTED, 진행 업로드 취소는 204 | manifest 삭제 미지원과 인증 거절을 같은 원인으로 처리하지 않음 |
| 네트워크·정리 | Registry backend는 Pod 내부 loopback만 수신, Pod 외부 직접 접근 거절·시험 proxy 경유 확인. 시험 port-forward 종료·정확한 UID 자원 삭제와 부재 확인 | 운영 DNS/endpoint 연결·운영 데이터 변경 없음. 실제 Registry 통합 구간의 임시 자원 4개 정리 |

운영 Registry 인증 전환·소비자 image pull·자격 증명 교체, 영구 데이터 복구와 장기 sizing은 후속입니다. 자동 교체 방식 선택과 연결 경로 조사는 구현·타이머 활성화·운영 성공으로 합산하지 않습니다. AI 모델·Embedding·전체 AI E2E가 완료됐다는 근거도 아닙니다. [건강 검사와 프로토콜 검증의 판단](design-decisions.md#registry의-생존-준비-상태와-프로토콜-검증을-분리)을 참조하세요.

## 서비스 통합 확인 범위

| 대상 | 확인한 이력·범위 | 미확인 범위 |
|---|---|---|
| 포털의 Kubernetes 제공 | 기존 Windows Standard·Development 기록 유지; 11월 2일 원격 Linux의 별도 Development 로그인·앱/권한·자연 만료 갱신과 callback 대체·정상 삭제·접속 회수 | Linux의 독립 Cloud 잔류 자원 조회·볼륨 재시험, WSL·일반 Linux/신규 tenant·계정 정지·HA·장기 용량·모든 장애 및 현재 운영 상태 |
| AI 챗봇 | G5 완료: DB HA 시나리오·합성 벡터·DGX Pod의 DNS/TLS/인증/SQL·접근 제한·임시 자원 정리 | Portal 요청부터 모델 응답까지 전체 E2E, PgBouncer·앱 연동·성능·백업/PITR 및 전체 운영 인수 |

단일 GPU 물리 노드 장애에 대한 고가용성은 제공하지 않습니다.

## AI 데이터 기반의 인증·권한 검증

2026-10-12~13 기록에서는 3멤버 etcd의 기동·자동 시작 설정, 관리 TLS를 통한 endpoint 상태와 건강성, 멤버 목록·알람을 확인했습니다. 관리 인증서를 통한 계정 접근과 비상 비밀번호의 정상·오류 인증도 검증했습니다.

2026-10-13에는 두 Patroni용 계정에 독립 비밀번호와 하나의 제한 역할을 적용했습니다. 각 계정의 허용 prefix 읽기 성공, 잘못된 비밀번호 거부, 범위 밖 읽기 및 관리 목록 접근 거부를 확인했습니다. 이어 승인된 두 시험 키에서 조건부 생성·값/revision 대조·조건부 삭제·부재 확인을 통과했습니다. 마지막 플레이북은 ok 33, changed 4, failed 0으로 종료했습니다.

시험은 etcd VM 한 대에서 자기 endpoint로 실행했습니다. HTTP gateway의 CN 인증서 거부를 확인한 뒤 비상 비밀번호 검사를 etcdctl gRPC 경로로 보정한 이력도 포함합니다. 따라서 실제 DB VM과 Patroni gateway 접속, 범위 밖 쓰기 거부, 장애 전환까지 성공했다고 판단하지 않습니다. [그림과 상세 설명](ai-data-foundation-flow.md)을 참조하세요.

<a id="database-validation-20261018-20"></a>
## 2026-10-18~20 DB 시나리오와 당시 후속 경계

위의 10월 12~13일 계정 시험과 별도로, 이후 PostgreSQL·Patroni와 DB 접속점을 실제 시험했습니다. 다음은 저장된 실행 기록의 범위이며 현재 상태를 다시 조회한 결과가 아닙니다.

| 날짜·시나리오 | 확인한 결과 | 한계·미완료 |
|---|---|---|
| 10월 18일 소량 합성 벡터 | 저장·L2 검색 결과의 양쪽 일치, 데이터·인덱스 복제, HNSW 생성과 실제 Index Scan | 작은 합성 자료에서 세션의 순차 스캔을 억제한 시험; 기본 planner 선택·성능·검색 품질·실제 앱 검증 아님 |
| 10월 19일 계획 전환·정상 재부팅 | 역할 변경·복귀, 같은 DB 접속점의 TLS 읽기·새 쓰기와 Replica 반영, 데이터·인덱스 보존 | 승인된 시나리오와 관측 시점에 한정 |
| 10월 19일 짧은 watchdog 시험 | watchdog reset 사건과 재부팅·DB 접속 복귀 확인 | 기대한 다른 노드의 자동 승격은 검증 미통과. 다음 종료 유지 시험의 성공으로 이 결과를 덮지 않음 |
| 10월 20일 Primary 종료 유지 | 수동 승격 없이 다른 노드 Primary·동일 접속점 TLS 읽기, 원래 노드 Replica 재합류, 새 쓰기·복제 확인 | 해당 종료 유지 시나리오; 관측 상한을 정확한 중단·승격·재합류 시간으로 해석하지 않음 |
| 10월 20일 Primary 강제 재부팅 | 실제 재부팅, 접속 실패 구간 뒤 같은 접속점의 재접속, 역할·복제·데이터/인덱스 보존과 새 쓰기·Replica 반영 | 동일 Primary로 복구. 다른 노드 자동 승격의 증거가 아님. 관측 오류 원문이 없어 실패 계층을 확정하지 않음 |
| 10월 20일 DGX 일반 Pod 경로 | 기본 DNS·TCP 3회와 실제 출발 경로 확인, 관측 종료·임시 Pod 삭제 | 접근 제한과 DB TLS·인증·SQL 검증은 후속 |

강제 재부팅의 연속 관측을 조기 종료했으며, 프로세스 종료를 확인하고 이후 쓰기·복제 검증을 완료했습니다. 예정된 전체 관측 시간을 채우거나 정상 종료한 것으로 기록하지 않습니다. 장기 안정성·무중단·백업 복구·실제 AI 앱 통합을 뜻하지 않습니다.

<a id="g5-closeout-20261021"></a>
## 2026-10-21 G5 완료와 최종 경계

앞선 표의 D1 이후, D2에서 일반 DGX Pod와 제어 클라이언트의 strict TLS·클라이언트 인증·합성 9행 조회가 성공했다. 실제 출발 주소에 맞춰 LB/보안그룹을 두 단일 주소로 제한한 뒤 양쪽의 조회를 재검증했다. D2 Pod·암호문은 정리했으며, 21일에는 검증 DB·계정·전용 HBA 규칙 제거와 양쪽 부재를 확인했다.

최종 관측은 PostgreSQL VM B Primary/VM A Replica, TLS 비동기 streaming, 역할에 맞는 watchdog, etcd 세 멤버 health 및 동일 리더·알람 없음이다. Octavia listener/Primary member는 ONLINE이고 Replica member ERROR 및 LB/pool DEGRADED는 Primary 전용 검사에서 나온 예상 집계였다. DB 자체의 Replica 건강성과 구분했다.

합의한 구축·A~D 시험·정리·최종 건강 확인을 완료했다. [사례 문서](g5-postgresql-ha.md)에 시험별 증거, 관측 시간의 한계, 커널/드라이버·부팅 보정과 DGX 신뢰 경계를 정리했다.

- 비동기 복제의 무손실 RPO, 모든 장애 조합·장기 가용성을 보장하지 않는다.
- etcd 멤버 상실, 물리 compute 장애, 실제 pg_rewind 수행, 허용 주소 밖의 차단 접속은 미검증이다.
- LB monitor의 CA/서버명 검증 예외는 DB 클라이언트 strict TLS와 구분한다.
- 백업/PITR, GPU 모델 canary, PgBouncer·앱 DB 권한 및 전체 서비스 종단은 후속이다.

<a id="access-observation-20261018-20"></a>
## 2026-10-18~20 접속·관측 구현과 실제 적용

다음 표는 당시의 완료·미완료 기록입니다. 이후 제품 배포와 실제 사용자 성공은 [10월 28일 결과](#standard-user-e2e-20261028)를 따릅니다.

| 범위 | 확인한 기록 | 남은 검증 |
|---|---|---|
| 접속 파일 등록 | Windows PowerShell 5.1/7 각각 22개, Ubuntu 24.04 등록 25개와 메모리 시험 키의 서명·검증·등록 5개 검사; 별도 공개 배포처와 운영 공개키 게시 | 합성 파일·격리 검사. Portal 다운로드·사용자 kubectl E2E·WSL 고유 연동은 미완료 |
| 상태·Usage·접속 준비·회수 연결 | 10월 18~19일 API/Worker·화면 연결 코드와 격리 시험 | 로컬 구현·격리 성공을 실제 역할 UI나 새 클러스터 사용자 시험으로 확대하지 않음 |
| 실제 관측 기능 적용 | 10월 19일 finalizer 소스·두 관측 기능 적용. 프로젝트 3개 중 2개 READY snapshot, 1개는 자격증명 참조 조회 실패로 UNAVAILABLE | 당시 활성 workload 관측 대상 0개. 새 workload 관측 성공은 미확인 |
| 실제 제어 서비스 적용 | 10월 20일 00:25 KST까지 중계 전용 runtime·설정/CSR 준비, 제어 서비스 두 개 각각 Ready·실제 DB TLS·비활성 등록 상태 확인 | 중계 leaf·신원 등록, 제어망·unit 기동·ACK, 제품 다운로드·역할 UI·사용자 종단 미완료 |

10월 11일의 성공은 그때의 소스·설정에 한정됩니다. 이후 적용한 관측·접속·회수 계약과 미배포 후보의 E2E를 대신하지 않습니다. 공개 등록기의 [10월 18일 준비 기록](https://github.com/seungyeobchoi/portal-kubectl-access/blob/main/release-status.json)도 게시·제품 연결·사용자 시험을 별도 항목으로 구분합니다.

## 설계 후보와 후속 제안

아래 항목은 각 기록 당시의 후보·제안 상태를 보존한 것입니다. AI 구성과 제어 서버 A의 후속 상태는 [2026-10-16 기준 보완](#2026-10-16-기준-설계-상태-보완)을 따릅니다.

- AI 모델·검색 구성: 2026-08-26 기록의 vLLM·Qwen3, pgvector·HNSW, RAG·Rule Engine·MCP는 설계 후보입니다. 최종 채택, 실제 서빙과 응답 품질·처리량·지연은 확인되지 않았습니다.
- 제어 서버 A의 추가 compute 역할: 설계 제안으로 구분하며 도식의 구축 범위에 포함하지 않았습니다.
- FDP의 PLI별 큐 처리와 배치 정책 개선: 후속 연구 제안입니다. 완료한 WAF·TPM 실험과 원인에 대한 해석은 [연구 문서](fdp-research.md)에 구분했습니다.

## 2026-10-16 기준 설계 상태 보완

2026-10-16은 기존 후속 결정을 공개 문서에 정리한 기준일이며, 해당 결정을 이날 새로 내렸거나 새 실행 검증을 수행했다는 뜻은 아닙니다. 위의 2026-08-26 후보 기록과 기존 실행 결과는 당시 범위로 유지합니다.

| 대상 | 후속 확정한 설계 | 구분할 구현·검증 범위 |
|---|---|---|
| PostgreSQL·pgvector | GPU 장비 밖의 AI PostgreSQL과 pgvector를 데이터·벡터 저장 구성으로 채택 | 실제 애플리케이션 DB 연동, 벡터 검색과 성능 검증 완료를 뜻하지 않음 |
| RAG | CPU RAG Query와 별도 CPU Embedding Serving, 독립 Deployment 경계 | 모델·서비스 API·검색과 권한 검사 담당 및 실제 응답 품질·처리량·지연 검증은 별도 |
| MCP | MCP Tool Gateway를 독립 Deployment로 분리하고 학사 rule/calculator·Search connector를 내부 도구 모듈로 구성 | 외부 Search API 제공자·세부 호출 계약·도구 연동 검증은 별도 |
| 애플리케이션 구성 | 상시 Deployment 8종과 필요 시 실행하는 운영 Job, 역할별 PgBouncer 경유 DB 접근 | 설계 확정을 배포·실제 서빙·전체 E2E 성공으로 확대하지 않음 |

vLLM·Qwen3 등 모델·추론 엔진과 HNSW 세부 설정은 최종 채택으로 표시하지 않습니다. [전체 구조와 처리 흐름](ai-chatbot-architecture.md)에서 확정 설계·TBD·구현 연기·공개 생략을 구분합니다.

제어 서버 A의 compute 역할도 당시 제안 단계 이후에 추가됐습니다. 2026-10-11 기록에서는 제어 서버 A·복합 서버 B·복합 서버 C에 etcd VM을 한 대씩 배치했고 세 VM 모두 Nova ACTIVE임을 확인했습니다. 이는 향후 세 번째 호스트 확보를 전제로 한 계획이 아니라 해당 시점의 실제 VM 배치 확인입니다. 이 결과만으로 호스트 장애 시 합의 유지나 PostgreSQL 자동 장애 전환을 검증한 것은 아닙니다. [하드웨어 그림의 후속 설명](../diagrams/01-hardware.md#2026-10-16-기준-compute-역할-보완)을 참조하세요.

## 검증 방법과 역할

설정과 실제 출력, 코드와 계층별 로그를 대조하고, 한 단계의 성공을 전체 서비스 완료로 판단하지 않습니다. 조회 실패 하나만으로 원인을 확정하지 않고 연결 관계와 실행 결과를 함께 살핍니다.

본인은 클라우드·포털과 AI 챗봇의 서비스 구조, 인프라 설계·통합, 문제 해결과 검증을 담당합니다. AI 챗봇의 LLM 관련 설계는 동료 1명이 담당하며, 설계 판단과 변경 허용 범위는 직접 검토·결정하고, 포털 API/Worker 구현·점검 도구·로그 분석에는 Codex의 지원을 받았습니다. [개발·검증 방식](cloud-platform.md#codex-development-validation)에서 본인 판단과 지원 역할을 구분합니다.

공개 문서에는 실제 주소·계정·인증자료·원본 운영 출력·비공개 원문 링크·실행 가능한 운영 설정을 포함하지 않았습니다. 구조도는 역할과 설계 판단을 설명하기 위한 요약입니다.
