# Groupeat — 단체주문 플랫폼

> 단톡방 메뉴 취합과 개별 입금 대조로 흩어져 있던 단체 주문을, 링크 하나로 끝내는 서비스입니다.
> CEOS 23기 프로젝트 (2026.05 ~ 2026.08)

<p>
  <img src="https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/React_Query-FF4154?style=flat-square&logo=reactquery&logoColor=white" />
  <img src="https://img.shields.io/badge/Zustand-4B3B2A?style=flat-square" />
  <img src="https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white" />
  <img src="https://img.shields.io/badge/pnpm-F69220?style=flat-square&logo=pnpm&logoColor=white" />
</p>

**배포** [<!-- 배포 URL을 여기에 넣어 주세요 -->](https://www.groupeat.co.kr/)

<br>

## 문제 정의

단체 주문은 오랫동안 이런 방식으로 처리되어 왔습니다.

```
단톡방에 메뉴판 사진 업로드
  → 각자 카톡으로 메뉴 답장
  → 총무가 수기로 취합
  → 계좌번호 공유 후 입금 대조
  → 누락 · 중복 · 변경 발생
```

주문 인원이 늘어날수록 부담이 **총무 한 사람에게** 몰리는 구조였습니다.
Groupeat은 이 과정을 링크 하나로 대체합니다. 원하는 주문 조건을 선택하고, 픽업 일자와 메뉴 및 수량을 선택만 한다면 바로 픽업까지 이어질 수 있습니다.

<br>

## 스크린샷

<!--
  스크린샷을 이 위치에 넣어 주세요. 아래 형식을 권장합니다.

  <p>
    <img width="240" alt="주문 생성" src="..." />
    <img width="240" alt="메뉴 선택" src="..." />
    <img width="240" alt="주문 현황" src="..." />
  </p>
-->

<br>

## 기술 스택

| 영역 | 스택 |
|---|---|
| Framework | Next.js (App Router) |
| Language | TypeScript |
| 서버 상태 | React Query |
| 클라이언트 상태 | Zustand |
| 스타일 | Tailwind CSS |
| 타입 생성 | openapi-typescript |
| 패키지 관리 | pnpm |
| CI / 코드 리뷰 | GitHub Actions, CodeRabbit |

<br>

## 개발 방식

### 명세를 단일 기준으로 삼는 구조

화면은 매번 다르지만 **API 연동은 같은 형태가 반복**됩니다.
이 프로젝트에서는 화면 구현보다 명세 확인과 연동 배선의 반복이 실제 병목이라고 판단하고, 그 부분을 구조로 해결했습니다.

착수 전에 **API 명세와 페이지 플로우를 먼저 정의**하고, 디렉토리 구조 · 네이밍 · 상태 관리 방식을 팀 공통 컨벤션으로 고정했습니다.
생성 품질의 편차는 모델이 아니라 **입력의 편차**에서 발생한다고 보았기 때문입니다.

정해진 컨벤션 위에서 작업하면 누가 맡든 결과물의 형태가 같아지고, 리뷰에서 확인할 것이 로직 자체로 좁혀집니다.

### 생성 결과를 신뢰하지 않는 검증 단계

명세 기반으로 만든 코드를 사람이 전부 검토하는 것은 현실적이지 않습니다.
그래서 **자동으로 걸러낼 수 있는 오류**와 **사람이 반드시 확인해야 할 판단**을 분리했습니다.

**1단계 — 타입으로 걸러내기**
`openapi-typescript`로 API 명세에서 타입을 자동 생성해, 명세와 코드의 불일치를 컴파일 단계에서 검출합니다.
사람이 검토하지 않아도 형태 오류는 빌드가 막아 줍니다.

**2단계 — 리뷰 규칙을 명시하기**
타입 검증은 형태만 보장할 뿐 **런타임 값과 비즈니스 로직은 잡지 못합니다.**
그 한계를 전제로, 반복해서 어긋나기 쉬운 지점을 `.coderabbit.yaml`에 프로젝트 규칙으로 직접 작성했습니다. 

| 검증 항목 | 대상 |
|---|---|
| React Hooks | 의존성 배열 누락 |
| App Router | Server / Client 컴포넌트 분리, 불필요한 `'use client'` |
| TypeScript | `any` 사용 금지, 명시적 타입 정의 |
| Zustand | 불필요한 리렌더를 유발하는 상태 관리 패턴 |
| 유지보수성 | 과도하게 길거나 복잡한 함수 |
| 예외 처리 | 비정상 입력 · 엣지 케이스에 대한 try-catch, 폼 검증, 폴백 UI |

**3단계 — CI에서 강제하기**
ESLint · Prettier · GitHub Actions로 스타일과 정적 분석을 파이프라인에서 고정했습니다.

<br>

### 인증 / 인가

주문과 마이페이지는 로그인이 필요한 영역입니다.
클라이언트에서 화면을 가리는 방식 대신, **요청 단계에서 차단**하도록 `proxy.ts`에 접근 제어를 두었습니다.

- 쿠키의 `ACCESS_TOKEN` 유무로 보호 라우트 접근을 판단
- 미인증 상태로 `/mypage`, `/order` 진입 시 리다이렉트
- 인증 상태로 로그인 페이지 진입 시 리다이렉트

<br>

## 프로젝트 구조

```
app/          라우트 및 페이지 (Next.js App Router)
components/   공용 UI 컴포넌트
hooks/        커스텀 훅
store/        Zustand 전역 상태
types/        API 명세 기반 생성 타입
lib/          유틸리티
src/          <!-- 실제 용도에 맞게 채워 주세요 -->
proxy.ts      라우트 접근 제어
```

<br>

## 규모

| 항목 | 수치 |
|---|---|
| 연동 API 엔드포인트 | 77개 |
| 구현 페이지 | 37개 |
| 커밋 | 1,300+ |
| 프론트엔드 개발 인원 | 2명 |
| 개발 기간 | 3개월 |
