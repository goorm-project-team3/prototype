# 3팀 기획안 — GooR(oo)M
[프로토타이핑](https://goorm-project-team3.github.io/prototype/)

## 1. 프로젝트 개요

| 항목 | 내용 |
|---|---|
| 프로젝트명 | GooR(oo)M (문서 표기: GooRoom) |
| 한 줄 정의 | 강사·학생이 한 화면에서 강의하고 실습하는 학습 특화 Web IDE |
| 컨셉 키워드 | 강의룸 · 커서 공유 · 포커스 모드 · 강사 노트 · 이해도 반응 |
| 팀 구성 | 4인 (FE 2 / BE 2) |
| 기간 | 2026.5.1 ~ 6.5 (약 5주) |
| 배포 형태 | Web (반응형), FE: Vercel / BE: Render / DB: Render PostgreSQL |

### 1.1 배경 & 동기

> "실시간 강의를 듣는 와중에도 코드는 GitHub에서 확인하고, 강의 내용은 Notion에 정리하며, 실제 실습은 VS Code에서 이루어지는 — 이러한 분산된 도구들로 인한 불편감을 겪었다."

부트캠프 강의 환경에서 발생하는 마찰:

- 강사 측: 화면 공유 도구 + 코드 에디터 + 채팅창 + 필기 도구를 번갈아 가며 관리. 학생 이해도를 실시간으로 파악하기 어렵다.
- 학생 측: 강사 화면 따라가면서 + 본인 코드 작성하면서 + 노트 정리까지 — 창 세 개를 동시에 오간다. 강사가 어느 파일의 몇 번째 줄을 보는지 놓치는 경우가 빈번하다.

### 1.2 목표

- 정성 목표
  - "강사는 한 화면에서 강의·시연·노트를 끝내고, 학생도 한 화면에서 따라가기·실습·노트를 완결한다"는 학습 경험을 만든다.
  - 풀스택 사이클(요구사항 → 설계 → 구현 → 배포)을 팀 단위로 1회 완주하며, WebSocket 실시간 기능을 실서비스 배포까지 경험한다.
- 정량 목표 (MVP)
  - 강의룸 입장 → 강사 커서 확인 → 포커스 모드 토글까지 핵심 플로우를 30초 이내에 완료할 수 있는 UI 동선
  - 커서 공유 지연 500ms 이내 (LAN 기준)
  - 동시 접속 15명 규모 강의룸 안정 동작

## 2. 타겟 사용자 & 문제 정의

### 2.1 페르소나

| 페르소나 | 역할 | 핵심 페인 포인트 |
|---|---|---|
| Owner (강사) | 강의 진행, 코드 시연 | 화면 공유 + 에디터 + 채팅 + 필기 4창 분산, 학생 이해도 파악 어려움 |
| User (학생) | 강의 수강, 코드 실습 | 강사 화면 + 실습 에디터 + 노트 3창 분산, "강사가 어디 보는지" 파악 어려움 |

### 2.2 차별화 포인트 (왜 또 만드는가)

| 비교 대상 | 그쪽이 잘하는 것 | GooRoom이 다르게 하는 것 |
|---|---|---|
| VS Code + Zoom 화면 공유 | 강력한 에디터 + 영상 | 설치 없이 브라우저만으로, 강사 커서 공유·포커스 모드·이해도 반응이 에디터 안에 통합 |
| VS Code Live Share | 실시간 협업 편집 | 학습 특화 기능(포커스 모드·이해도 반응·강사 노트) 없음, 설치 필요 |

→ 요지: "화면 공유 도구 + 에디터 + 노트"를 각각 쓰는 게 아니라, 세 기능이 하나의 브라우저 탭 안에서 실시간으로 연결된 학습 환경이라는 포지셔닝.

## 3. 권한 체계

### 3.1 역할 기본값

| 권한 | 역할 | 커서 가시성 (기본값) | 편집 (기본값) | 파일·폴더 CRUD (기본값) |
|---|---|---|---|---|
| Owner | 강사·강의 진행자 | 본인 커서만 | O | O (강사 노트·학생 관리 포함) |
| User | 일반 학생 | Owner 커서 + 본인 | O | X |

### 3.2 Owner 강의룸 설정 패널 (동적 권한 제어)

| 설정 항목 | 대상 역할 | 기본값 | 활용 시나리오 |
|---|---|---|---|
| 커서 공유 | User | User: ON (Owner→User 단방향) | 강의 중 집중이 필요하면 User 커서 공유 OFF |
| 코드 편집 | User | 둘 다 ON | 강사 시연 중 학생 입력 잠금 → User 편집 OFF |
| 파일·폴더 CRUD | User | 둘 다 OFF | 멘토링 시 멘티(Editor)에게 파일 생성 허용 → Editor CRUD ON |

Owner는 강의룸 설정에서 역할별 세부 권한을 ON/OFF 가능하다. 이를 통해 강의 상황에 맞게 권한을 동적으로 조절할 수 있다.

커서 공유 원칙 (기본값 기준):

- Owner → User: 강사 커서를 모든 학생이 볼 수 있음 → 강의 모드
- User ↔ User: 다른 학생 커서는 보이지 않음 → 집중도 유지

## 4. 핵심 기능 — MoSCoW

| 분류 | 기능 | 비고 |
|---|---|---|
| Must | 회원가입·로그인 (JWT) | Spring Security + BCrypt |
| Must | 강의룸(프로젝트) · 폴더 · 파일 CRUD | 파일 트리 사이드바 |
| Must | 코드 에디터 (Monaco) | 다중 언어 신택스 하이라이트, 저장 |
| Must | Owner 커서·셀렉션 실시간 공유 | WebSocket(STOMP) 브로드캐스트 |
| Must | 포커스 모드 (강사 팔로우) | User가 토글 → Owner 파일·스크롤 위치 자동 추적 |
| Must | 강사 공유 노트 | Owner가 작성, User는 읽기 전용 (마크다운) |
| Must | 개인 노트 | 본인만 보이는 마크다운 메모 |
| Must | 실시간 채팅 (강의룸 단위) | 학생 → 강사 질문 채널 |
| Should | 이해도 반응 버튼 (✅ / ❓) | User가 클릭 → Owner 화면에 실시간 집계 |
| Should | 라인 포인터 (Owner 특정 라인 강조) | "지금 이 줄 보세요" — 모든 User 화면에 하이라이트 |
| Should | 반응형 레이아웃 | 1280px 이상 데스크탑 우선 |
| Should | 2단계 권한 (Owner / User) | 위 § 3 참조 |
| Should | 강의룸 권한 설정 패널 (Owner 전용) | 커서 공유 · 편집 · CRUD를 역할별로 ON/OFF. § 3.2 참조 |
| Could (Stretch) | 학생 코드 스냅샷 | Owner가 버튼 클릭 → 각 User의 현재 에디터 코드를 썸네일로 일괄 조회 |
| Won't | 코드 실행·컨테이너·디버거·Git 연동·음성 채팅 | MVP 범위 밖 |

## 5. 사용자 시나리오 (핵심 플로우)

### 5.1 UI 레이아웃 — VS Code 기반 골조

디자인 원칙: VS Code의 검증된 멀티존 레이아웃을 베이스 골조로 채택. Vapor UI(구름 디자인 시스템) 컴포넌트·토큰으로 구현.

```text
┌──────────────────────────────────────────────────────────────────────┐
│ [Top Bar] GooRoom · LectureRoom1 · 멤버(○○○○) · Owner · Profile     │
├──┬────────────────┬───────────────────────────────────┬─────────────┤
│ 📁 │ FILE EXPLORER  │ App.tsx × index.tsx × +          │ Chat ▼      │
│ 👥 │ ▾ src/         ├───────────────────────────────────┤             │
│ 📝 │ App.tsx        │ 1 // ← Owner 커서 (주황색)        │ 이해도 반응 │
│    │ index.tsx      │ 2 import React from 'react';     │ ✅ 8  ❓ 3   │
│    │ ▾ public/      │ 3 function App() {               │ ─────────   │
│    │ index.html     │ 4   return ...;                  │ 강사 노트   │
│    │                │ 5 }                              │ [마크다운]  │
│    │                │ (Owner 셀렉션 영역 하이라이트)   │             │
├──┴────────────────┴───────────────────────────────────┴─────────────┤
│ [Status Bar] User · 포커스 ON ▶ · 온라인 12명 · WS Connected         │
└──────────────────────────────────────────────────────────────────────┘
```

Zone 구성:

| Zone | 역할 | 주요 컴포넌트 |
|---|---|---|
| Top Bar | 컨텍스트 표시 | 강의룸명, 멤버 아바타(온라인), 권한 뱃지 |
| Activity Bar | 패널 전환 | Explorer / Members / Notes |
| Side Panel | 선택된 카테고리 | 파일 트리, 멤버 목록, 노트 목록 |
| Editor Area | 코드 편집 메인 | Monaco 에디터, Owner 커서(주황) 오버레이, 탭 바 |
| Right Panel | 학습 보조 정보 | Chat / 이해도 반응 집계 / 강사 노트 (탭 전환) |
| Status Bar | 실시간 상태 | 권한 뱃지, 포커스 모드 토글, 온라인 수, WS 연결 |

VS Code와의 의도적 차이:

- Editor Area에 Owner 커서 오버레이 — 학생 화면에서 강사 커서가 다른 컬러(주황)로 항상 표시
- Status Bar에 포커스 모드 토글 — 한 번 클릭으로 강사 팔로우 ON/OFF
- Right Panel에 이해도 반응 집계 — 학습 특화 실시간 피드백 채널
- 강사 공유 노트 패널 — 강의 중 바로 정리하는 Notion 대체 공간

### 5.2 [강의 모드] Owner → Users 다수

1. Owner가 강의룸 생성 → 링크로 학생 초대 (User 권한 자동 부여)
2. Owner가 코드 작성 시작 → 모든 User 화면의 Editor Area에 Owner 커서(주황색) 실시간 표시
3. User들이 포커스 모드 ON → Owner가 파일 이동·스크롤 시 User 화면도 자동으로 따라감
4. Owner가 강사 공유 노트에 핵심 설명 작성 → Right Panel 노트 탭에 실시간 반영
5. 학생들이 ✅ / ❓ 반응 → Owner의 Right Panel에 집계 표시 → 이해도 보고 진도 판단

## 6. 시스템 아키텍처

```text
┌─────────────────────────────────────────────────────────┐
│ Browser                                                 │
│ React + Vite + TypeScript                               │
│ ├─ Vapor UI (구름 디자인 시스템) + Tailwind             │
│ ├─ Monaco Editor (코드 편집기)                          │
│ ├─ React Query (서버 상태)                              │
│ ├─ Zustand (클라 상태: 활성 파일·포커스 모드·커서)      │
│ └─ STOMP.js (WebSocket 클라이언트)                      │
└──────────┬──────────────────────────────┬───────────────┘
           │ HTTPS (REST)                 │ WSS (STOMP)
           ▼                              ▼
┌──────────────────────────┐   ┌──────────────────────────────┐
│ Spring Boot Backend      │   │ Spring WebSocket (STOMP)     │
│ ├─ Spring Security+JWT   │   │ ├─ /topic/room/{id}/cursor   │
│ ├─ JdbcTemplate          │   │ ├─ /topic/room/{id}/focus    │
│ ├─ REST API              │   │ ├─ /topic/room/{id}/chat     │
│ └─ 권한 인터셉터         │   │ └─ /topic/room/{id}/reaction │
└──────────┬───────────────┘   └──────────────┬───────────────┘
           │                                  │
           ▼                                  ▼
┌────────────────────────────────────────────┐
│ RDB (개발: H2 / 운영: Render PostgreSQL)   │
└────────────────────────────────────────────┘
```

배포:

- FE: Vercel (GitHub 연동 자동 배포, PR 프리뷰 URL 자동 생성)
- BE: Render (Docker 이미지 자동 빌드·배포)
- DB: Render PostgreSQL (운영) / H2 (개발)
- CI/CD: GitHub Actions (lint·test·build) + Vercel/Render 자동 배포

핵심 WebSocket 토픽 설계:

| 토픽 | 발행 주체 | 구독 범위 | 전송 데이터 |
|---|---|---|---|
| /cursor | Owner, Editor | Owner→전체, Editor→Owner만 | 파일 ID, 라인, 컬럼, 셀렉션 범위 |
| /focus | Owner | 포커스 모드 ON User들 | 파일 ID, 스크롤 위치 |
| /chat | 모든 권한 | 강의룸 전체 | 메시지, 발신자, 타임스탬프 |
| /reaction | User, Editor | Owner (집계) | 반응 타입 (UNDERSTOOD / CONFUSED) |

## 7. 데이터 모델 (ERD 가안)

| 테이블 | 주요 컬럼 | 비고 |
|---|---|---|
| users | id, email, password(BCrypt), nickname, created_at |  |
| rooms | id, name, owner_id(FK users), created_at, updated_at |  |
| room_members | room_id, user_id, role(OWNER / EDITOR / USER), joined_at | 복합 PK |
| folders | id, room_id, parent_folder_id(nullable), name, created_at | 자기 참조 |
| files | id, folder_id, name, content(TEXT), language, updated_at, updated_by |  |
| chat_messages | id, room_id, user_id, content, created_at | 인덱스: (room_id, created_at) |
| shared_notes | id, room_id, content(TEXT), updated_at, updated_by | 강사 공유 노트. 강의룸당 1개 |
| user_notes | id, room_id, user_id, content(TEXT), updated_at | 개인 노트. (room_id, user_id) 복합 |
| reactions | id, room_id, user_id, type(UNDERSTOOD / CONFUSED), created_at | 이해도 반응 |
| room_settings | room_id(PK), cursor_editor, cursor_user, edit_editor, edit_user, crud_editor, crud_user | Owner가 설정하는 역할별 권한 ON/OFF. 각 컬럼 boolean |

> 정확한 컬럼·인덱스·외래키 제약은 Sprint 1 첫 2일 동안 BE 팀이 ERD를 확정해 Notion에 박제한다.

## 8. API 카테고리 (REST + WebSocket)

| 분류 | REST 엔드포인트 | WebSocket 토픽 |
|---|---|---|
| 인증 | POST /auth/signup, POST /auth/login, POST /auth/refresh | — |
| 강의룸 | GET/POST/PATCH/DELETE /rooms[/{id}] | — |
| 멤버·권한 | GET/POST/DELETE /rooms/{id}/members | — |
| 파일·폴더 | GET/POST/PATCH/DELETE /rooms/{id}/files, /folders | — |
| 채팅 | GET /rooms/{id}/chat?cursor=... (히스토리) | /topic/room/{id}/chat |
| 커서·포커스 | — | /topic/room/{id}/cursor, /focus |
| 이해도 반응 | GET /rooms/{id}/reactions (집계 조회) | /topic/room/{id}/reaction |
| 강사 공유 노트 | GET/PATCH /rooms/{id}/notes/shared | — |
| 개인 노트 | GET/PATCH /rooms/{id}/notes/personal | — |
| 강의룸 권한 설정 | GET/PATCH /rooms/{id}/settings (Owner 전용) | — |

> API 명세서는 SpringDoc(Swagger UI)으로 자동 생성. BE PR 머지 시 자동 갱신. Sprint 1 첫 주에 1차 스펙 동결 → FE는 MSW/mock 데이터 작성 기준으로 사용.

## 9. 기술 스택 & 선택 근거

| 레이어 | 채택 | 선택 근거 |
|---|---|---|
| FE 프레임워크 | React 18 + TypeScript + Vite | 강의 커리큘럼 일치, 타입 안전성, 빠른 빌드 |
| 에디터 | Monaco Editor (@monaco-editor/react) | VS Code 동일 엔진, Owner 커서 오버레이에 Monaco Decoration API 직접 활용 |
| 상태 관리 | React Query (서버) + Zustand (클라) | 포커스 모드·커서 상태 등 클라 전용 상태는 Zustand, 서버 데이터는 React Query |
| UI 라이브러리 | Vapor UI (구름 공식 디자인 시스템) + Tailwind | 부트캠프 주최사 구름의 공식 디자인 시스템, VS Code 레이아웃 골조 위에 구현 |
| WebSocket 클라 | STOMP.js + SockJS | Spring WebSocket과 짝, 토픽별 커서·포커스·채팅·반응 분리 처리 |
| BE 프레임워크 | Spring Boot 3 (Java) | 강의 커리큘럼 일치 |
| DB 접근 | JdbcTemplate (강의 범위) | 학습 단계상 SQL 가시성 우선 |
| 인증 | Spring Security + JWT | 표준적, 무상태로 확장 용이 |
| 실시간 | Spring WebSocket(STOMP) | Spring 생태계 일관성, 토픽 단위 브로드캐스트 |
| DB | 개발 H2 / 운영 Render PostgreSQL | JdbcTemplate 그대로, 드라이버만 교체 |
| 배포 (FE) | Vercel | GitHub 푸시 시 자동 빌드, PR 프리뷰 URL |
| 배포 (BE) | Render + Docker | Docker 학습 목표 유지 |
| CI/CD | GitHub Actions + Vercel/Render 자동 배포 | Actions = lint·test·build 게이트 |
| 협업 툴 | Discord / Notion / Figma / GitHub | 활동 계획서 참조 |

## 10. 팀 구성

- 팀 구성: 4인 (FE 2 / BE 2)

## 11. 일정 (마일스톤)

| 기간 | 단계 | FE 주요 작업 | BE 주요 작업 |
|---|---|---|---|
| 5/4 ~ 5/7 | 기획 멘토링 | 기획안·질문지 사전 공유 | 동일 |
| 5/11 | 기획 심사 (10분 발표 + 5분 QnA) | — | — |
| 5/12 ~ 5/20 (Sprint 1) | MVP 골격 | 라우팅·레이아웃·Monaco 통합·Owner 커서 오버레이 POC·목업 UI·강의룸 | DB/ERD 확정·인증 API·rooms/files CRUD·WebSocket 기본 셋업·API 명세 1차 |
| 5/21 ~ 5/25 (Sprint 2) | 학습 특화 기능 통합 | 포커스 모드·React Query 연동·채팅 UI·강사 노트·이해도 반응 UI | 커서 브로드캐스트·포커스 이벤트·반응 집계·노트 API |
| 5/25 (일) | 🛑 Feature Freeze | 신규 기능 동결 | 동일 |
| 5/26 ~ 6/2 | 안정화 & 배포 | 버그·반응형·Vercel 배포 검증·발표자료 초안(5/27~) | Render 배포·Docker 최적화·모니터링 |
| 6/3 ~ 6/4 | 발표 준비 | 장표·시연 영상·QnA 예상 답변 | 동일 |
| 6/5 | 최종 심사 | — | — |

> 버퍼 전략: Sprint 1 첫 2일 학습·POC 배정 (WebSocket STOMP + Monaco Decoration API 첫 도입 대비). Feature Freeze 후 Should(이해도 반응·라인 포인터) 구현 우선.

## 12. 리스크 & 대응

| 리스크 | 가능성 | 대응 |
|---|---|---|
| 커서 공유 지연 > 500ms (WebSocket 첫 도입) | 중상 | Sprint 1 첫 2일 STOMP POC. 지연 과도 시 폴링 fallback 검토. |
| Monaco Decoration API 학습 비용 | 중 | Sprint 1 POC에 포함. 학습 자료 풍부, Owner 커서 단색 오버레이부터 단계적 구현 |
| BE API 지연으로 FE 블로킹 | 중 | API 명세 선합의 → MSW/mock으로 FE 선개발, 실 API 완료 후 연동 |

## 13. 프로젝트 관리 계획

### 13.1 히스토리 관리

- GitHub-Flow: `main` ← `develop` ← `feature/<이슈번호>-<설명>`, 직접 push 금지
- PR 게이트: 1명 이상 리뷰 Approve + CI 통과 → Squash & Merge
- 칸반 보드: GitHub Projects (`Backlog / Todo / In-Progress / Review / Done`)
- 커밋 컨벤션 8종: `FEAT / FIX / REFACTOR / STYLE / DOCS / CHORE / TEST / DESIGN`
- CI: FE `pnpm lint · typecheck · build` / BE `./gradlew test · build`

### 13.2 문서화

| 문서 | 위치 | 갱신 주기 |
|---|---|---|
| 활동 계획서 / 기획안 | Notion | 변경 시 즉시 |
| API 명세서 | SpringDoc(Swagger UI) 자동 생성 | BE PR 머지 시 자동 |
| ERD | Notion | DB 변경 PR 동시 갱신 강제 |
| 회의록 | Notion | 회의 직후 |

### 13.3 회의 & 커뮤니케이션

- 데일리 스크럼 (평일 19:00, 15분)
- Discord 응답 24시간 내 원칙 · 모든 의사결정 회의록 박제

## 14. 기대 효과 & 학습 목표

### 14.1 사용자 측면

- 도구 분산 해소: GitHub·Notion·VS Code 세 창 → GooRoom 한 탭으로
- 강의 흐름 추적: Owner 커서 공유 + 포커스 모드 → 강의를 놓치지 않게
- 즉시 적응 가능한 UI: VS Code 기반 골조 → 개발자 학습 곡선 거의 0

### 14.2 팀 학습 목표

- 공통: 풀스택 사이클 1회 완주 / GitHub-Flow + PR 리뷰 문화 / 실서비스 배포(Vercel + Render + Docker)
- FE: Monaco Decoration API(커서 오버레이) / React Query + Zustand / STOMP 클라이언트 / Vapor UI 디자인 시스템
- BE: Spring Security + JWT / Spring WebSocket(STOMP) 커서·포커스·반응 멀티 토픽 / JdbcTemplate 기반 SQL

### 14.3 차별점 정리

> "강의를 실시간으로 따라가며 코드를 실습하고, 강의 노트를 한 화면에서 정리하는 학습 특화 Web IDE"
