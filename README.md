# 클라우드 플랫폼 엔지니어 포트폴리오

**OpenStack·Ceph·Kubernetes를 연결하고, 제약 조건 안에서 변경 범위를 정해 장애와 복구를 검증합니다.**

보유 서버의 사양과 통신 경로를 분석해 OpenStack·Ceph 기반 사설 클라우드를 구축했습니다. Kubernetes 기반 포털에서 사용자별 자원 신청과 볼륨 관리를 구현하고, 테스트 계정과 관리자 계정으로 동작을 검증했습니다. 석사 연구에서는 FDP SSD의 데이터 배치 정보를 전달하는 Linux 커널 인터페이스를 구현했습니다.

장비 역할과 네트워크·스토리지 배치, 서비스 연계와 변경 허용 범위는 직접 판단했습니다. API/Worker 구현·점검 도구·로그 분석에는 Codex의 지원을 받았습니다. AI 챗봇의 LLM 관련 설계는 동료 1명이 담당합니다.

<a id="주요-경험"></a>
## 대표 문제 해결

| 사례 | 본인이 판단한 내용 | 확인한 결과와 범위 |
|---|---|---|
| [Octavia 관리망 연결](docs/cloud-platform.md#octavia-관리망-연결-문제-해결) | 외부 스위치 권한 제약을 분석하고 기존 관리망·VXLAN을 유지하는 내부 VLAN 경로 선택 | 시험용 LB 생성·ACTIVE/ONLINE·삭제 후 정리 확인. 처리량 개선을 측정한 결과는 아님 |
| [Kubernetes 생성·정리](docs/cloud-platform.md#kubernetes-제공-서비스로-확장) | 생성 단계와 서비스 완료를 분리하고 상태·API·실제 자원으로 결과 대조 | 2026-10-11 단일 Standard 생성·상태 유지·정상 삭제, 보존 대조 31항목 통과. 최신 접속·관측 변경본 E2E와 별도 |
| [PostgreSQL HA와 watchdog](docs/g5-postgresql-ha.md) | 리더 조정·연결 대상 선택·watchdog의 책임과 장애별 성공 조건 분리 | 계획 전환·종료 유지 시 자동 승격·재합류·DGX Pod TLS/SQL·정리 확인. 모델 응답·앱 전체 E2E는 후속 |
| FDP 커널 인터페이스 | 응용 프로그램부터 NVMe 드라이버까지 I/O 경로 확장, 플래그 충돌 분석·수정 | 실제 장치의 데이터 배치와 WAF·TPM 비교 실험 |

<a id="프로젝트와-담당-범위"></a>
## 프로젝트와 검증 범위

| 프로젝트 | 시작·구축 시점 | 확인한 성과와 범위 |
|---|---|---|
| [사설 클라우드와 볼륨 포털](docs/cloud-platform.md) | 2026.05 ~ 2026.07.31 1차 구축 | OpenStack·Ceph 통합, 테스트·관리자 계정의 볼륨 신청·관리 기능 검증. 실제 사용자 도입 사례 없음 |
| [Kubernetes 제공 서비스](docs/cloud-platform.md#kubernetes-제공-서비스로-확장) | 2026.08 시작 | 당시 버전의 단일 Standard 생성·삭제 검증. 후속 사용자 종단·모든 프로필·동시 요청까지 확인한 것은 아님 |
| [GPU 기반 AI 챗봇](docs/ai-chatbot.md) | 2026.08 시작 | 2026-10-21 데이터 기반 시험·정리 완료. GPU 모델과 AI 애플리케이션 통합은 후속 |
| [FDP SSD 커널 인터페이스](docs/fdp-research.md) | 석사 연구 | **커널 구현·실장치 실험 완료** — Linux·ext4 인터페이스와 실제 SSD의 데이터 배치·WAF·TPM 실험 |

## 코드와 검증 바로 보기

- **[접속 파일 등록기 공개 코드](https://github.com/seungyeobchoi/portal-kubectl-access)**: [PowerShell](https://github.com/seungyeobchoi/portal-kubectl-access/blob/main/clients/windows/Register-PortalKubeconfig.ps1) · [Python](https://github.com/seungyeobchoi/portal-kubectl-access/blob/main/clients/ubuntu/register_portal_kubeconfig.py) · [검증 범위와 제공 준비 상태](https://github.com/seungyeobchoi/portal-kubectl-access/blob/main/release-status.json)
- **[날짜별 검증 결과](docs/validation.md)**: [Standard 생성·삭제](docs/validation.md#standard-e2e-20261011) · [DB 장애 시험·정리](docs/validation.md#g5-closeout-20261021)
- **[개발·검증 방식](docs/cloud-platform.md#codex-development-validation)**: 본인 판단, Codex 지원, 구현·격리 시험·실제 적용·종단 시험의 구분

공개 코드는 서명 검증·접속 파일 등록 도구입니다. Windows/Ubuntu의 합성 파일·격리 검사를 확인했으며 Portal 다운로드 연결·실제 사용자 kubectl E2E·WSL 고유 연동은 미완료입니다.

<a id="구조도와-상세-문서"></a>
## 대표 사례와 상세 문서

- [클라우드 구축과 사용자 자원 제공](docs/cloud-platform.md): Octavia·Kubernetes 문제 해결, 역할과 구현 범위, 기존 구조도
- [PostgreSQL HA 구축과 장애 검증](docs/g5-postgresql-ha.md): watchdog 보정, 장애별 결과, DGX 데이터 경로와 정리
- [AI 챗봇 소개](docs/ai-chatbot.md) · [전체 구조와 처리 흐름](docs/ai-chatbot-architecture.md): 완료된 데이터 기반과 후속 AI 서비스 설계
- [kubectl 인증·접속·복구 흐름](docs/kubectl-command-flow.md): 선택한 접속 방식과 남은 사용자 시험

### FDP SSD 커널 인터페이스 연구

1. **프로젝트 소개**: [FDP SSD 파일시스템 인터페이스 연구](docs/fdp-research.md)
2. **설계 판단·검증**: [구현과 문제 해결](docs/fdp-research.md#구현과-문제-해결) → [실제 장치 실험](docs/fdp-research.md#실제-장치-실험) → [결과와 해석의 구분](docs/fdp-research.md#결과와-해석의-구분)
3. **상세 구조와 결과 그래프**: [연구 포트폴리오 PDF](pdf/fdp-research.pdf) 3~6쪽

## PDF 포트폴리오

- [클라우드 인프라 포트폴리오 5쪽](pdf/cloud-portfolio.pdf)
- [FDP 연구 포트폴리오 6쪽](pdf/fdp-research.pdf)

클라우드 PDF는 2026-10-21에 공개 문서와 맞춰 갱신했습니다. 실제 실행 날짜와 적용 범위는 연결된 검증 기록을 따릅니다.

## 기록과 공개 범위

저장된 기록과 공개 문서를 재구성한 포트폴리오이며 현재 운영 상태를 직접 재검증한 보고는 아닙니다. 실제 주소·계정·인증정보·원본 운영 로그·인계문을 공개하지 않습니다. PDF에서도 사진·이름·연락처·소속과 작성자 메타데이터를 제외했습니다.
