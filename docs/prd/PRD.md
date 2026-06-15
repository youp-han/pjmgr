# Product Requirements Document — Project Management Web App

**작성일:** 2026-06-15  
**버전:** 0.1 (초안)  
**상태:** 검토 중

---

## 1. 제품 개요

팀과 개인이 프로젝트, 태스크, 일정, 리소스를 한 곳에서 관리할 수 있는 웹 기반 프로젝트 관리 도구.

### 핵심 목표
- 프로젝트 진행 상황의 가시성 확보
- 팀원 간 협업 및 커뮤니케이션 중앙화
- 데드라인 추적 및 리소스 배분 최적화

---

## 2. 사용자 유형 (Personas)

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
- 이메일 + 비밀번호 로그인
- OAuth 2.0: Google, GitHub
- 2FA (TOTP)
- 세션 관리 (디바이스 목록, 강제 로그아웃)
- 감사 로그 (Admin용)

---

## 4. 추가 기능 (Nice-to-Have)

| 기능 | 우선순위 | 설명 |
|------|----------|------|
| 스프린트 관리 | 높음 | 스프린트 생성, 백로그 관리, 스프린트 리뷰 |
| 타임 트래킹 | 중간 | 태스크별 실제 소요 시간 기록 |
| 자동화 (Automation) | 중간 | 규칙 기반 트리거 (예: 상태 변경 시 담당자 알림) |
| 템플릿 | 중간 | 프로젝트·태스크 템플릿 라이브러리 |
| 외부 연동 (Integration) | 낮음 | Slack, GitHub, Jira, Google Drive |
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

## 6. 기술 스택 후보

### 프론트엔드
- **Framework:** Next.js 14+ (App Router)
- **UI Library:** shadcn/ui + Tailwind CSS
- **State:** Zustand / TanStack Query
- **실시간:** WebSocket (Socket.io) 또는 SSE
- **드래그앤드롭:** @dnd-kit

### 백엔드
- **Runtime:** Node.js (NestJS) 또는 Python (FastAPI)
- **DB:** PostgreSQL (메인) + Redis (캐시·세션)
- **파일 스토리지:** S3-compatible (AWS S3 / MinIO)
- **검색:** PostgreSQL Full-Text Search 또는 Meilisearch
- **Queue:** BullMQ (알림·이메일 비동기 처리)

### 인프라
- **배포:** Docker + Docker Compose (초기) → Kubernetes (확장)
- **CI/CD:** GitHub Actions
- **모니터링:** Sentry (에러), Prometheus + Grafana (메트릭)

---

## 7. 화면 목록 (Screen Inventory)

```
/ (랜딩 페이지)
/login
/signup
/forgot-password

/dashboard                        ← 홈 대시보드
/projects                         ← 프로젝트 목록
/projects/new
/projects/[id]                    ← 프로젝트 상세
/projects/[id]/board              ← 칸반 보드
/projects/[id]/list               ← 리스트 뷰
/projects/[id]/timeline           ← 간트 차트
/projects/[id]/calendar           ← 캘린더
/projects/[id]/reports            ← 리포트
/projects/[id]/settings           ← 프로젝트 설정

/tasks/[id]                       ← 태스크 상세 (모달 or 페이지)

/workspace/settings               ← 워크스페이스 설정
/workspace/members                ← 멤버 관리
/workspace/billing                ← 플랜 & 결제

/profile                          ← 내 프로필
/notifications                    ← 알림 센터
```

---

## 8. 개발 우선순위 (Phase)

### Phase 1 — MVP (1~2개월)
- [ ] 인증 (이메일, Google OAuth)
- [ ] 워크스페이스 & 프로젝트 CRUD
- [ ] 태스크 CRUD (담당자, 우선순위, 상태, 마감일)
- [ ] 칸반 보드 뷰
- [ ] 리스트 뷰
- [ ] 기본 알림 (인앱)

### Phase 2 — 협업 강화 (2~3개월)
- [ ] 댓글 & 멘션
- [ ] 파일 첨부
- [ ] 간트 / 타임라인 뷰
- [ ] 마일스톤
- [ ] 이메일 알림
- [ ] 검색 & 필터

### Phase 3 — 고도화 (3~4개월)
- [ ] 스프린트 관리
- [ ] 리포트 & 분석
- [ ] 자동화 규칙
- [ ] 외부 연동 (Slack, GitHub)
- [ ] AI 기능

---

## 9. 미결 사항 (Open Questions)

1. **과금 모델**: 프리미엄 SaaS vs 셀프호스팅 오픈소스 vs 하이브리드?
2. **멀티 워크스페이스**: 한 계정에 여러 워크스페이스 허용 여부?
3. **스프린트 vs 칸반**: 스크럼 방식 우선 지원 여부?
4. **데이터 지역화**: 특정 국가/리전 데이터 저장 요구사항 있음?
5. **오프라인 지원**: PWA 오프라인 편집 범위는?

---

*이 문서는 초안으로, 팀 검토 후 업데이트될 예정입니다.*
