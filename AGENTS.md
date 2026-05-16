# AGENTS.md

이 파일은 코딩 에이전트가 이 프로젝트에서 안전하고 일관되게 작업하기 위해 필요한 최소한의 컨텍스트를 제공한다.
짧게 유지한다. 자세한 기술 문서는 `docs/`에 두고, 에이전트의 작업 방식이 바뀌어야 할 때만 이 파일을 수정한다.

## 프로젝트 목적

`k8s-data-project-crawling`은 Airflow와 Kubernetes를 사용해 데이터 크롤링 작업을 실행하기 위한 프로젝트다.

주요 책임:

- 재사용 가능한 크롤러와 데이터 처리 코드를 구현한다.
- Airflow DAG는 얇게 유지하고 오케스트레이션에 집중한다.
- Kubernetes manifest는 환경별 차이를 관리할 수 있고 재현 가능하게 작성한다.
- 테스트, DAG import, manifest 검증을 빠르게 실행할 수 있는 로컬 검증 명령을 제공한다.

## 작업 규칙

- 수정하기 전에 현재 작업, 저장소 상태, 관련 파일을 먼저 확인한다.
- 새로운 구조나 의존성을 도입하기보다 기존 프로젝트 패턴을 우선한다.
- 변경 범위는 요청된 작업에 맞게 좁게 유지한다.
- 작업에 필요하지 않은 파일, 생성 파일, secret, 로컬 env 파일, dependency lock 파일은 수정하지 않는다.
- 동작이 바뀌면 관련 테스트를 추가하거나 수정한다.
- 설정, 명령, 아키텍처, 컨벤션이 바뀌면 관련 문서를 업데이트한다.
- 명령이 실패하면 코드를 바꾸기 전에 에러를 읽고 원인을 먼저 파악한다.

## 저장소 구조

프로젝트가 커질 때 아래 구조를 목표로 한다:

```text
.
├── AGENTS.md
├── README.md
├── TASKS.md
├── docs/
│   ├── architecture.md
│   ├── workflow.md
│   ├── conventions.md
│   ├── airflow.md
│   ├── kubernetes.md
│   ├── crawling.md
│   ├── data-schema.md
│   └── troubleshooting.md
├── src/
│   ├── crawlers/
│   ├── pipelines/
│   ├── storage/
│   └── utils/
├── airflow/
│   ├── dags/
│   ├── plugins/
│   └── tests/
├── k8s/
│   ├── base/
│   ├── overlays/
│   │   ├── local/
│   │   ├── dev/
│   │   └── prod/
│   └── jobs/
├── scripts/
├── tests/
└── .github/
    └── workflows/
```

## 문서 규칙

- `README.md`: 사람이 읽는 프로젝트 개요, 설치 방법, 빠른 시작 문서.
- `TASKS.md`: 현재 목표, 진행 중인 작업 목록, 우선순위, 완료 기준.
- `docs/architecture.md`: 시스템 경계, 데이터 흐름, 주요 설계 결정.
- `docs/workflow.md`: 로컬 개발, 검증, 릴리스, 배포 흐름.
- `docs/conventions.md`: 네이밍, 스타일, 파일 위치, 기여 규칙.
- `docs/airflow.md`: DAG 작성 규칙, 스케줄, 재시도, 변수, 테스트.
- `docs/kubernetes.md`: manifest 구조, overlay, resource, secret, 검증 방식.
- `docs/crawling.md`: 크롤러 패턴, 요청 예절, 재시도, 파싱, 에러 처리.
- `docs/data-schema.md`: 입력/출력 계약, 저장 경로, 스키마 버전.
- `docs/troubleshooting.md`: 반복적으로 발생하는 에러, 원인, 검증된 해결 방법.

문서가 아직 없다면, 해당 지식이 앞으로도 유지될 필요가 있을 때만 새로 만든다.

## 코드 위치

- 재사용 가능한 Python 애플리케이션 로직은 `src/` 아래에 둔다.
- 크롤러 구현은 `src/crawlers/` 아래에 둔다.
- 변환 또는 적재 로직은 `src/pipelines/` 아래에 둔다.
- 저장소 클라이언트와 영속화 코드는 `src/storage/` 아래에 둔다.
- 공통 helper는 `src/utils/` 아래에 둔다.
- Airflow DAG 정의는 `airflow/dags/` 아래에 둔다.
- Kubernetes 리소스는 `k8s/` 아래에 둔다.
- 반복 실행되는 로컬 또는 CI 명령은 `scripts/` 아래에 둔다.
- 테스트는 `tests/` 아래에 두되, 이미 도메인별 테스트 디렉터리가 정해져 있으면 그 구조를 따른다.

## Airflow 지침

- DAG 파일은 task를 오케스트레이션해야 하며, 무거운 비즈니스 로직을 담지 않는다.
- task callable은 Airflow 밖에서도 import하고 테스트할 수 있게 유지한다.
- DAG parse 시점에 네트워크 호출이나 무거운 연산이 실행되지 않게 한다.
- schedule, retry, timeout, catchup 동작은 명시적으로 작성한다.
- 환경별 값을 하드코딩하지 말고 문서화된 Airflow variable 또는 environment variable을 우선 사용한다.

## Kubernetes 지침

- base manifest는 재사용 가능하게 유지하고, 환경별 차이는 overlay에 둔다.
- 실제 secret은 커밋하지 않는다.
- manifest를 적용하기 전에 dry-run 검증을 우선한다.
- 예약 실행 또는 장시간 실행 workload에는 가능한 경우 resource request와 limit을 설정한다.
- job 이름, label, selector는 manifest 전반에서 일관되게 유지한다.

## 검증

가장 빠르고 관련성 높은 검증을 먼저 실행한다. 프로젝트 script가 있다면 ad hoc 명령보다 script를 우선 사용한다.

프로젝트가 성숙해지면 아래 명령을 권장한다:

```bash
./scripts/test.sh
./scripts/validate.sh
python -m pytest
airflow dags list
kubectl apply --dry-run=client -k k8s/overlays/local
```

위 명령이 아직 존재하지 않는다면 결과를 만들어내지 않는다. 작업에 필요하면 script를 만들고, 그렇지 않으면 검증 명령이 아직 없다고 보고한다.

## Git 및 리뷰

- 변경 전 `git status --short --branch`를 확인한다.
- working tree에 이미 있는 사용자 변경사항은 보존한다.
- 명시적으로 요청받지 않는 한 history rewrite, reset, 변경사항 폐기는 하지 않는다.
- 작업 요약에는 변경한 파일, 실행한 검증, 남은 위험을 포함한다.

## 에이전트 인수인계

작업이 크거나 중간에 끊길 수 있다면 다음 세션이 이어받을 수 있게 충분한 컨텍스트를 남긴다:

- 현재 목표.
- 변경한 파일.
- 실행한 명령과 결과.
- 알려진 blocker.
- 다음에 할 구체적인 작업.
