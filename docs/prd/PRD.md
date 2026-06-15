# Product Requirements Document — Project Management Web App

**작성일:** 2026-06-15  
**버전:** 0.3  
**상태:** 검토 중  
**변경:** PM 플로우 & 상태 정의 추가, MS Teams/Meet + Google Docs 연동, 인증 라이브러리 확정 (Microsoft.Identity.Web), ORM 명시

---

## 1. 제품 개요

팀과 개인이 프로젝트, 태스크, 일정, 리소스를 한 곳에서 관리할 수 있는 웹 기반 프로젝트 관리 도구.

### 핵심 목표
- 프로젝트 진행 상황의 가시성 확보
- 팀원 간 협업 및 커뮤니케이션 중앙화
- 데드라인 추적 및 리소스 배분 최적화

---

## 2. 프로젝트 관리 기본 플로우

### 전체 흐름 개요

```
[1. 프로젝트 개시]
    PM이 프로젝트 생성 → 목표·기간 설정 → 팀원 초대 & 역할 배정
         ↓
[2. 계획 (Planning)]
    작업 분해: Epic → Story → Task → Sub-task
    우선순위·담당자·마감일 설정 → 마일스톤 등록 → 스프린트 구성
         ↓
[3. 실행 (Execution)]
    팀원이 태스크 착수 (Todo → In Progress)
    진행 중 댓글·파일 공유 → MS Meet 미팅 링크로 실시간 논의
    Google Docs 문서 연결 → 요구사항·설계서 공동 작업
         ↓
[4. 검토 (Review)]
    담당자가 완료 처리 (In Progress → In Review)
    리뷰어가 확인 후 승인(Done) 또는 반려(In Progress 복귀 + 코멘트)
         ↓
[5. 모니터링 (Monitoring)]
    PM이 대시보드·간트·번다운 차트로 전체 진행 상황 파악
    마일스톤 달성률·지연 태스크 알림 수신
         ↓
[6. 완료 & 회고 (Closure)]
    모든 태스크 Done → 마일스톤 달성 → 프로젝트 Complete 처리
    스프린트 리뷰·회고 기록 → 프로젝트 Archive
```

---

### 상태 정의 (Status Definitions)

#### 프로젝트 상태

| 상태 | 설명 | 전이 가능 대상 |
|------|------|----------------|
| `Draft` | 생성 직후, 계획 수립 중 | Planning |
| `Planning` | 태스크·마일스톤 구성 중 | Active, Cancelled |
| `Active` | 실행 진행 중 | On Hold, Completed |
| `On Hold` | 일시 중단 (외부 요인) | Active, Cancelled |
| `Completed` | 모든 작업 종료 | Archived |
| `Archived` | 읽기 전용 보관 | — |
| `Cancelled` | 취소됨 | — |

```
Draft → Planning → Active ⇄ On Hold
                     ↓
                Completed → Archived
         (모든 경로) → Cancelled
```

---

#### 태스크 상태

| 상태 | 설명 | 전이 가능 대상 |
|------|------|----------------|
| `Todo` | 생성됨, 미착수 | In Progress, Cancelled |
| `In Progress` | 담당자가 작업 중 | In Review, Todo, Cancelled |
| `In Review` | 검토 요청됨 | Done, In Progress |
| `Done` | 완료 승인 | (재오픈 시 In Progress) |
| `Cancelled` | 불필요하여 취소 | — |

```
Todo → In Progress → In Review → Done
                ↑_______________|   (반려 시 복귀)
```

---

#### 마일스톤 상태

| 상태 | 설명 |
|------|------|
| `Upcoming` | 기한 전, 미달성 |
| `In Progress` | 연결 태스크 일부 완료 중 |
| `Achieved` | 기한 내 모든 연결 태스크 Done |
| `At Risk` | 연결 태스크 중 지연 발생 (자동 감지) |
| `Missed` | 기한 초과, 미달성 |

---

#### 스프린트 상태 (Phase 2)

| 상태 | 설명 |
|------|------|
| `Draft` | 백로그에서 태스크 구성 중 |
| `Active` | 스프린트 진행 중 (1개만 Active 허용) |
| `Completed` | 스프린트 종료, 미완료 태스크는 백로그 복귀 |

---

## 3. 사용자 유형 (Personas)

| 역할 | 설명 |
|------|------|
| **Project Manager** | 프로젝트 생성, 팀 구성, 전체 일정 관리 |
| **Team Member** | 할당된 태스크 확인·수행·업데이트 |
| **Viewer / Stakeholder** | 읽기 전용 대시보드 조회 |
| **Admin** | 워크스페이스 설정, 사용자 권한 관리 |

---

## 3. 핵심 기능 (Core Features)

### 3.1 워크스페이스 & 조직 관리
- 워크스페이스 생성 및 멤버 초대 (이메일 / 링크)
- 역할 기반 접근 제어 (RBAC): Admin / Manager / Member / Viewer
- 워크스페이스 단위 설정 (타임존, 언어, 알림 등)

### 3.2 프로젝트 관리
- 프로젝트 생성·수정·보관·삭제
- 프로젝트 상태: `계획중 / 진행중 / 완료 / 보류`
- 프로젝트별 멤버 배정 및 역할 지정
- 프로젝트 카테고리 / 태그 분류

### 3.3 태스크 관리
- 태스크 생성·수정·삭제·복제
- 태스크 계층 구조: Epic > Story > Task > Sub-task
- 태스크 속성
  - 제목, 설명 (마크다운 지원)
  - 담당자 (멀티 어사인 지원)
  - 우선순위: Urgent / High / Medium / Low
  - 상태: `Todo / In Progress / In Review / Done / Cancelled`
  - 시작일 / 마감일
  - 예상 소요 시간 (Story Point 또는 시간 단위)
  - 레이블 / 태그
  - 첨부파일
- 태스크 간 의존성 설정 (Blocked by / Blocking)
- 태스크 댓글 & 멘션 (`@username`)
- 활동 로그 (변경 이력)

### 3.4 뷰 (View)
| 뷰 | 설명 |
|----|------|
| **Board (칸반)** | 상태별 컬럼으로 드래그 앤 드롭 |
| **List** | 테이블 형식, 정렬·필터 지원 |
| **Gantt / Timeline** | 날짜 기반 일정 시각화, 의존성 표시 |
| **Calendar** | 월/주/일 뷰로 마감일 확인 |
| **Dashboard** | KPI 요약, 진행률, 번다운 차트 |

### 3.5 일정 & 마일스톤
- 마일스톤 생성 및 태스크 연결
- 프로젝트 타임라인 자동 계산
- 마감 임박 알림 (D-7, D-3, D-1)
- 일정 충돌 감지

### 3.6 팀 협업
- 실시간 댓글 및 스레드
- 파일 첨부 (드래그 앤 드롭, 최대 50MB)
- 멘션 알림 (`@`)
- 태스크 공유 링크
- 활동 피드 (프로젝트/워크스페이스 단위)

### 3.7 알림 (Notifications)
- 인앱 알림 센터
- 이메일 알림 (일별/주별 다이제스트 옵션)
- 알림 유형: 태스크 할당, 댓글 멘션, 마감 임박, 상태 변경, 초대

### 3.8 검색 & 필터
- 전체 텍스트 검색 (태스크, 댓글, 파일명)
- 고급 필터: 담당자, 우선순위, 상태, 레이블, 날짜 범위
- 저장된 필터 (개인 / 팀 공유)
- 최근 검색 기록

### 3.9 리포트 & 분석
- 프로젝트 진행률 (완료 태스크 / 전체)
- 번다운 차트 (Sprint Burndown)
- 팀원별 태스크 부하 현황
- 사이클 타임 / 리드 타임 분석
- PDF / CSV 내보내기

### 3.10 인증 & 보안

**인증 라이브러리:** `Microsoft.Identity.Web` (NuGet) + `ASP.NET Core Identity`

| 항목 | 내용 |
|------|------|
| **이메일 + 비밀번호** | ASP.NET Core Identity (bcrypt 해싱, 계정 잠금 정책) |
| **Microsoft 로그인** | Microsoft.Identity.Web → Azure AD / Microsoft 365 계정 (Teams/Meet 연동 토큰 동시 획득) |
| **Google 로그인** | `Microsoft.AspNetCore.Authentication.Google` → Google OAuth 2.0 (Docs 연동 토큰 동시 획득) |
| **GitHub 로그인** | `AspNet.Security.OAuth.GitHub` |
| **2FA** | TOTP (Google Authenticator 호환), 백업 코드 |
| **JWT** | Access Token (15분) + Refresh Token (7일, Redis 저장) |
| **세션 관리** | 디바이스 목록 조회, 특정 세션 강제 로그아웃 |
| **감사 로그** | 로그인·권한 변경·삭제 이력 (Admin 조회) |

> Microsoft.Identity.Web을 선택한 이유: Microsoft 계정으로 로그인 시 Microsoft Graph API 접근 토큰을 자동으로 함께 발급받아 별도 OAuth 절차 없이 MS Teams 미팅 생성에 재사용 가능.

---

### 3.11 MS Teams / Microsoft Meet 연동

#### 연동 목적
태스크·마일스톤 단위에서 MS Teams 미팅을 생성·참여하고, 미팅 결과(회의록)를 태스크에 자동 연결.

#### 기능 상세

| 기능 | 설명 |
|------|------|
| **미팅 링크 생성** | 태스크/마일스톤 상세에서 "MS Meet 링크 생성" 버튼 → Microsoft Graph API `POST /onlineMeetings` 호출 → 생성된 joinUrl 저장 |
| **미팅 링크 표시** | 태스크 상세 패널에 참여 링크 고정 표시, 클릭 시 Teams 앱/웹 오픈 |
| **Teams 채널 알림** | 프로젝트에 Teams 채널 연결 → 태스크 상태 변경·마감 임박 알림을 Teams Webhook으로 발송 |
| **미팅 일정 등록** | 미팅 생성 시 참여자 캘린더에 자동 이벤트 추가 (`POST /events`) |
| **회의록 연결** | 미팅 종료 후 Teams 회의록 URL을 해당 태스크 첨부파일로 저장 |

#### 기술 구현
- **인증:** Microsoft.Identity.Web으로 로그인 시 `Calendars.ReadWrite`, `OnlineMeetings.ReadWrite` 스코프 요청
- **API 클라이언트:** `Microsoft.Graph` NuGet 패키지 (Graph SDK)
- **토큰 캐시:** Redis에 사용자별 Graph 토큰 캐싱 (만료 시 자동 갱신)

---

### 3.12 Google Docs 연동

#### 연동 목적
태스크·프로젝트에 Google Docs 문서를 연결하거나 신규 생성하여, 요구사항 정의서·설계서·회의록을 별도 탭 없이 인앱에서 확인.

#### 기능 상세

| 기능 | 설명 |
|------|------|
| **문서 연결** | 태스크/프로젝트에 Google Docs URL 붙여넣기 → 제목·썸네일 자동 파싱 후 저장 |
| **문서 생성** | "새 Google Docs 생성" 버튼 → Drive API로 문서 생성 → 태스크 이름을 초기 제목으로 설정 → 링크 자동 저장 |
| **인앱 미리보기** | 첨부된 Docs를 iframe embed (`/preview` URL)로 슬라이드오버 내 표시 |
| **권한 공유** | 문서 생성 시 프로젝트 멤버를 Commenter 권한으로 자동 공유 (`drive.files.permissions`) |
| **문서 목록** | 프로젝트 Overview에서 연결된 Docs 문서 목록 표시 |

#### 기술 구현
- **인증:** Google OAuth 로그인 시 `https://www.googleapis.com/auth/drive.file` 스코프 요청 (앱이 생성한 파일만 접근)
- **API 클라이언트:** `Google.Apis.Drive.v3` NuGet 패키지
- **토큰 저장:** 사용자별 Google Refresh Token DB 암호화 저장 (AES-256)

---

## 4. 추가 기능 (Nice-to-Have)

| 기능 | 우선순위 | 설명 |
|------|----------|------|
| 스프린트 관리 | 높음 | 스프린트 생성, 백로그 관리, 스프린트 리뷰 |
| 타임 트래킹 | 중간 | 태스크별 실제 소요 시간 기록 |
| 자동화 (Automation) | 중간 | 규칙 기반 트리거 (예: 상태 변경 시 담당자 알림) |
| 템플릿 | 중간 | 프로젝트·태스크 템플릿 라이브러리 |
| 외부 연동 (Integration) | 낮음 | Slack, Jira (MS Teams / Google Docs는 핵심 기능으로 격상) |
| AI 기능 | 낮음 | 태스크 요약, 일정 제안, 리스크 감지 |
| 모바일 앱 | 낮음 | React Native 또는 PWA |
| 오프라인 지원 | 낮음 | PWA + Service Worker |

---

## 5. 비기능 요구사항

| 항목 | 목표 |
|------|------|
| 응답속도 | 페이지 로드 < 2초 (LCP), API 응답 < 300ms (p95) |
| 가용성 | 99.9% Uptime (월 다운타임 < 44분) |
| 확장성 | 워크스페이스당 최대 500명, 태스크 100,000개 |
| 보안 | HTTPS 필수, XSS/CSRF/SQL Injection 방어, OWASP Top 10 준수 |
| 접근성 | WCAG 2.1 AA 수준 |
| 브라우저 지원 | Chrome, Firefox, Safari, Edge 최신 2버전 |

---

## 6. 기술 스택 (확정)

### 프론트엔드
- **Framework:** Next.js 14+ (App Router)
- **UI Library:** shadcn/ui + Tailwind CSS
- **State:** Zustand (클라이언트) + TanStack Query (서버 상태)
- **실시간:** WebSocket (SignalR 허브 연동)
- **드래그앤드롭:** @dnd-kit

### 백엔드 (REST API)
- **Framework:** ASP.NET Core 8 (Web API)
- **ORM:** Entity Framework Core 8 + `Npgsql.EntityFrameworkCore.PostgreSQL`
  - Code-First 마이그레이션, Fluent API로 관계 정의
  - 복잡 쿼리는 Dapper 병행 (리포트 집계 등)
- **실시간:** ASP.NET Core SignalR
- **인증 라이브러리:**
  - `ASP.NET Core Identity` — 사용자 저장소, 비밀번호 해싱, 잠금
  - `Microsoft.Identity.Web` — Microsoft 365 / Azure AD OAuth + Graph API 토큰 관리
  - `Microsoft.AspNetCore.Authentication.Google` — Google OAuth + Drive API 토큰 관리
  - `AspNet.Security.OAuth.GitHub` — GitHub OAuth
  - JWT Bearer (Access 15분 / Refresh 7일, Redis 저장)
- **외부 API 클라이언트:**
  - `Microsoft.Graph` — MS Teams 미팅 생성, 캘린더 이벤트
  - `Google.Apis.Drive.v3` — Google Docs 생성·공유
- **Queue:** Hangfire + PostgreSQL 스토리지 (알림·이메일·Webhook 비동기 처리)
- **파일 스토리지:** S3-compatible (AWS S3 / MinIO)

### 데이터베이스
- **메인 DB:** PostgreSQL 16
  - Full-Text Search (`tsvector`) → 별도 검색 엔진 불필요 (초기)
  - JSONB → 태스크 커스텀 필드 유연 저장
- **캐시 / 세션:** Redis 7

### 인프라
- **배포:** Docker + Docker Compose (초기) → Kubernetes (확장)
- **CI/CD:** GitHub Actions
- **모니터링:** Sentry (에러 추적), Prometheus + Grafana (메트릭)

---

## 7. 화면 목록 (Screen Inventory)

> **레이아웃 원칙**
> - 인증 후 전체 화면은 **좌측 사이드바 + 상단 헤더 + 콘텐츠 영역** 고정 쉘 사용
> - 태스크 상세는 **슬라이드오버 패널** (현재 뷰 컨텍스트 유지, URL 변경 포함)
> - 전역 검색은 **커맨드 팔레트** (`Cmd+K / Ctrl+K`) 오버레이로 처리

---

### A. 퍼블릭 (비인증)

| 경로 | 화면 | 주요 요소 |
|------|------|----------|
| `/` | 랜딩 페이지 | 기능 소개, CTA(회원가입/로그인), 요금제 |
| `/login` | 로그인 | 이메일+비밀번호, Google/GitHub OAuth, 2FA 입력 |
| `/signup` | 회원가입 | 이름, 이메일, 비밀번호, 약관 동의 |
| `/verify-email` | 이메일 인증 | 인증 코드 입력, 재전송 버튼 |
| `/forgot-password` | 비밀번호 찾기 | 이메일 입력 → 발송 안내 |
| `/reset-password` | 비밀번호 재설정 | 새 비밀번호 + 확인 입력 |
| `/invite/[token]` | 초대 수락 | 워크스페이스 정보 표시, 수락/거절 |

---

### B. 온보딩 (최초 로그인)

| 경로 | 화면 | 주요 요소 |
|------|------|----------|
| `/onboarding` | 워크스페이스 설정 마법사 | ① 이름·로고 ② 팀원 초대 ③ 첫 프로젝트 생성 |

---

### C. 홈 & 내 작업

| 경로 | 화면 | 주요 요소 |
|------|------|----------|
| `/home` | 홈 대시보드 | 오늘 마감 태스크, 최근 프로젝트, 활동 피드, 요약 통계 |
| `/my-tasks` | 내 태스크 | 전체 프로젝트를 아우르는 내 담당 태스크 목록, 프로젝트·상태·기간별 그룹핑, 필터 |

---

### D. 프로젝트

| 경로 | 화면 | 주요 요소 |
|------|------|----------|
| `/projects` | 프로젝트 목록 | 카드/리스트 토글, 상태·즐겨찾기 필터, 정렬 |
| `/projects/[id]/overview` | 프로젝트 개요 | 진행률, 마일스톤 타임라인, 최근 활동, 멤버 목록 |
| `/projects/[id]/board` | 칸반 보드 | 상태 컬럼, 카드 드래그앤드롭, 인라인 태스크 생성 |
| `/projects/[id]/list` | 리스트 뷰 | 테이블, 컬럼 커스텀, 정렬·필터, 인라인 편집 |
| `/projects/[id]/timeline` | 간트 차트 | 날짜 바 드래그, 의존성 화살표, 마일스톤 마커 |
| `/projects/[id]/calendar` | 캘린더 | 월/주 뷰, 마감일 표시, 드래그로 날짜 변경 |
| `/projects/[id]/reports` | 리포트 | 진행률 차트, 번다운, 담당자별 부하, CSV/PDF 내보내기 |
| `/projects/[id]/members` | 프로젝트 멤버 | 멤버 목록, 역할 변경, 초대, 제거 |
| `/projects/[id]/settings` | 프로젝트 설정 | 이름·설명·상태·카테고리, 위험 영역(보관·삭제) |

> `/projects/[id]` 접근 시 `/projects/[id]/overview` 로 리다이렉트

---

### E. 태스크 상세 (슬라이드오버 패널)

URL: `/projects/[id]/board?task=[taskId]` 형태로 딥링크 지원

| 탭 | 내용 |
|----|------|
| **상세** | 제목, 설명(마크다운), 담당자, 우선순위, 상태, 날짜, 레이블, Story Point |
| **서브태스크** | 체크리스트 형태, 인라인 추가 |
| **첨부파일** | 드래그앤드롭 업로드, 미리보기 |
| **댓글** | 마크다운 + 멘션, 스레드 |
| **활동** | 변경 이력 타임라인 |

---

### F. 알림

| 경로 | 화면 | 주요 요소 |
|------|------|----------|
| `/notifications` | 알림 센터 | 전체/읽지않음 탭, 유형별 필터, 일괄 읽음 처리 |

---

### G. 워크스페이스 설정 (Admin 전용)

| 경로 | 화면 | 주요 요소 |
|------|------|----------|
| `/settings/general` | 일반 설정 | 워크스페이스명, 로고, 타임존, 언어 |
| `/settings/members` | 멤버 관리 | 초대(이메일/링크), 역할 변경, 제거 |
| `/settings/roles` | 역할 & 권한 | RBAC 권한 매트릭스 편집 |
| `/settings/security` | 보안 | 감사 로그, 활성 세션 목록, 강제 로그아웃 |
| `/settings/notifications` | 알림 기본값 | 워크스페이스 기본 알림 정책 |
| `/settings/integrations/teams` | MS Teams 연동 | Teams 채널 Webhook 등록, 알림 이벤트 선택 |
| `/settings/integrations/google` | Google 연동 | Google 계정 연결·해제, Drive 권한 범위 확인 |
| `/settings/integrations/slack` | Slack 연동 | Slack Workspace 연결 (Phase 3) |
| `/settings/billing` | 요금제 & 결제 | 현재 플랜, 사용량, 카드 관리 |

---

### H. 내 프로필 & 계정

| 경로 | 화면 | 주요 요소 |
|------|------|----------|
| `/profile` | 프로필 | 이름, 아바타, 소속, 시간대 |
| `/profile/security` | 계정 보안 | 비밀번호 변경, 2FA 설정/해제, 연결된 OAuth, 디바이스 목록 |
| `/profile/notifications` | 알림 설정 | 유형별 인앱/이메일 수신 여부, 다이제스트 빈도 |

---

### I. 전역 UI 컴포넌트 (화면 아님, 오버레이)

| 컴포넌트 | 트리거 | 기능 |
|----------|--------|------|
| **커맨드 팔레트** | `Ctrl+K` | 전역 검색, 빠른 태스크 생성, 페이지 이동 |
| **알림 드롭다운** | 헤더 벨 아이콘 | 최근 5건 미리보기, 전체보기 링크 |
| **태스크 생성 모달** | `+` 버튼 / 단축키 | 프로젝트 선택 후 빠른 태스크 생성 |

---

### 화면 수 요약

| 그룹 | 화면 수 |
|------|---------|
| 퍼블릭 (비인증) | 7 |
| 온보딩 | 1 |
| 홈 & 내 작업 | 2 |
| 프로젝트 뷰 | 8 |
| 태스크 상세 (패널) | 1 |
| 알림 | 1 |
| 워크스페이스 설정 | 9 |
| 내 프로필 & 계정 | 3 |
| **합계** | **32** |

---

## 8. 개발 우선순위 (Phase)

### Phase 1 — MVP (1~2개월)
- [ ] 인증: 이메일 + Microsoft OAuth (Microsoft.Identity.Web) + Google OAuth
- [ ] 워크스페이스 & 프로젝트 CRUD + 상태 머신 구현
- [ ] 태스크 CRUD — 계층 구조, 담당자, 우선순위, 상태 전이
- [ ] 칸반 보드 뷰 + 리스트 뷰
- [ ] 기본 인앱 알림 (SignalR)

### Phase 2 — 협업 & 외부 연동 (2~3개월)
- [ ] 댓글 & 멘션 & 파일 첨부
- [ ] MS Teams 미팅 링크 생성 (Microsoft Graph API)
- [ ] Teams Webhook 채널 알림
- [ ] Google Docs 연결 & 생성 (Drive API)
- [ ] Google Docs 인앱 미리보기 (iframe embed)
- [ ] 마일스톤 + 마일스톤 상태 자동 감지
- [ ] 간트 / 타임라인 뷰
- [ ] 이메일 알림 (Hangfire)
- [ ] 검색 & 필터 (PostgreSQL FTS)

### Phase 3 — 고도화 (3~4개월)
- [ ] 스프린트 관리 (백로그, 스프린트 보드, 회고)
- [ ] 리포트 & 분석 (번다운, 부하, 사이클 타임)
- [ ] 자동화 규칙 (트리거 → 액션)
- [ ] MS Teams 회의록 자동 연결
- [ ] Slack 연동
- [ ] AI 기능 (태스크 요약, 일정 제안)

---

## 9. 미결 사항 (Open Questions)

1. **과금 모델**: 프리미엄 SaaS vs 셀프호스팅 오픈소스 vs 하이브리드?
2. **멀티 워크스페이스**: 한 계정에 여러 워크스페이스 허용 여부?
3. **스프린트 vs 칸반**: 스크럼 방식 우선 지원 여부?
4. **데이터 지역화**: 특정 국가/리전 데이터 저장 요구사항 있음?
5. **오프라인 지원**: PWA 오프라인 편집 범위는?

---

*이 문서는 초안으로, 팀 검토 후 업데이트될 예정입니다.*
