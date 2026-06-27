# Moles - Claude 작업 규칙서

## 프로젝트 개요

Unity 3D 기반의 SF 멀티 슈팅 게임. 절차적으로 생성된 방(Room)을 탐색하며 적을 처치하는 구조. 1인칭 시점.

- Unity 프로젝트 경로: `MOLES/`
- 커스텀 스크립트 경로: `MOLES/Assets/Scripts/`
- 임포트 에셋 경로: `MOLES/Assets/Imported_Assets/`

---

## 아키텍처 구조

### 핵심 시스템

| 클래스 | 역할 |
|---|---|
| `GameManager` | 싱글턴. 룸 목록 관리, 플레이어 위치 지정, 문 상태 제어 |
| `RoomCreator` | 절차적 방 위치 배열 생성 |
| `Room` | 개별 방. 적 소환, 벽 배치, 클리어 판정, 문 위치 관리 |
| `ElevatorRoom` | 스테이지 시작/종료 엘레베이터 방 |
| `EnemyCommon` | 모든 적의 공통 베이스 클래스 (MonoBehaviour 상속) |
| `MeleeAttackEnemy` | 근접 공격 적. EnemyCommon 상속 |
| `PlayerController` | 플레이어 이동 및 입력 처리 |
| `PlayerGun` | 총기 발사 로직 |
| `DoorControl` | 문 개폐 제어 |

### 클래스 상속 구조

```
MonoBehaviour
  └── EnemyCommon          (적 공통 기반)
        └── MeleeAttackEnemy   (근접 공격 적)
        └── (추후 다른 적 타입 추가 예정)
  └── Room
        └── ElevatorRoom
```

### 주요 참조 관계

- `GameManager.instance` — 전역 싱글턴, 씬 전체에서 접근
- `Room` → `EnemyCommon` 리스트 (`EnemiesInRoom`)
- `GameManager` → `Room` (`CurrentPlayRoom`)
- `EnemyCommon.myRoom` → 자신이 속한 `Room`
- 플레이어 오브젝트 이름: `"Player"` (GameObject.Find로 탐색)
- Resources 경로: `"PlayerSet"`, `"Trunckarce"` (적), `"ItemBio1"` (드롭 아이템)

---

## C# 코딩 규칙

### 기본 원칙

- 주석은 한국어로 작성 (기존 코드 스타일 유지)
- 새 코드에 주석은 WHY가 비자명할 때만 추가, WHAT 설명 주석은 쓰지 않음
- `Debug.Log`는 개발 중에만 사용, 완성된 기능에선 제거
- `[SerializeField]` 대신 `public` 사용 중 (기존 스타일 유지)

### 적 클래스 작성 규칙

- 새 적 타입은 반드시 `EnemyCommon`을 상속
- `EnemyCommon.Start()`는 `protected virtual` — 오버라이드 시 `base.Start()` 호출
- 적 사망 시 반드시 `myRoom.SetEnemyNum(-1)` 호출 (룸 클리어 판정에 사용)
- `TargetDetected`, `PlayerInRoom` 플래그를 통해 적 활성화 여부 결정

### 룸 시스템 규칙

- 적 소환은 `Room.SummonEnemy()`에서 처리 (Awake에서 호출)
- 문 위치는 1~8 정수로 표현 (`allDoorList`, `myDoorList`)
- 룸 클리어 조건: `EnemyNum <= 0` → `RoomClear()` → `GameManager.EnableAllDoors()`
- 플레이어가 룸 진입 시 `GameManager.setCurrentPlayRoom()` 호출

### NavMesh

- 모든 이동하는 적은 `NavMeshAgent` 컴포넌트 필요
- `EnemyCommon.Start()`에서 `GetComponent<NavMeshAgent>()` 취득

---

## 작업 시 주의사항

### 하지 말 것

- `Imported_Assets/` 내 파일 수정 금지 (외부 패키지)
- `Enemy.cs` 직접 수정 금지 — 구버전(`Enemy_OldVersion.cs`도 존재). 새 로직은 `EnemyCommon` 또는 그 하위 클래스에 작성
- `GameManager`에 DontDestroyOnLoad가 적용되어 있으므로 씬 전환 시 중복 생성 주의
- 주석처리된 코드 블록 복원 금지 — 의도적으로 비활성화된 것들임

### 파일 인코딩

- 일부 `.cs` 파일의 한국어 주석이 깨진 문자로 표시될 수 있음 (EUC-KR 인코딩 문제). 파일 수정 시 UTF-8 BOM 없이 저장

### 씬 구조

- 실제 씬 파일(`.unity`)은 수정하지 않고 스크립트만 수정
- Inspector에서 할당되는 `public` 필드는 코드로 찾지 말 것 (이미 씬에 연결되어 있음)

---

## 현재 개발 상황 (2026-06-27 기준)

- 스테이지 시작/종료 구조 완성 (`ElevatorRoom` 기반)
- 엘레베이터 룸 제작 완료
- 점프 공격 구현 중간 단계 완료
- 적이 공격 목표까지 이동 후 Roar 단계 진입하는 흐름 구현 중
- `MeleeAttackEnemy`가 주력 적 클래스

## 추후 개발 방향

- 적 종류 추가 시 `EnemyCommon` 상속으로 확장
- 문 상태는 `GameManager.ShutdownAllDoors()` / `EnableAllDoors()`로 일괄 제어
