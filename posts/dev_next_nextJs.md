---
title: 'Next.js 이해하기'
date: '2026-01-29'
category: 'Dev'
tags: ['Next.js', 'App Router', 'Server component', 'Client component']
summary: 'Next.js를 이해하고 실무에서 적용가능한 개념 학습하기'
---

# Next.js 기본개념 + 실무패턴 배우기

## Next.js

### Next.js란
Next.js: <mark>React를 "브라우저 UI 라이브러리"에서 "서버 중심 웹 어플리케이션 프레임워크"로 확장한 도구</mark>

기존 React에서 추가 된 기능
- **서버 실행 환경**
- **라우팅 규칙**: 기존 React의 라우팅 코드로 선언한 라우팅구조를 폴더로 강제
- **렌더링 전략(SSR/SSG/ISR)**
- **데이터 패칭 규칙**: page.tsx 혹은 server component에서만 데이터를 fetch 한다.
```ts
fetch("/api/products", { cache: "no-store" })
// Next는 fetch()에 캐시·재검증 전략 기능을 부여한다
```
- **캐시/재검증**: 이전 데이터 재사용, 캐시 유효기간 전략에 대한 기능
```ts
// SSG 전략
fetch(url, { cache: "no-store" })
```
```ts
// SSG 전략
fetch(url, { cache: "force-cache" })
```
```ts
// ISR 전략
fetch(url, { next: { revalidate: 60 } })
```
- **API 라우트**:   Next.js의 서버 실행 환경을 HTTP API 엔드포인트로 노출하는 기능으로 클라이언트·서버 어디서든 호출 가능한 REST 스타일 API를 제공한다. route.ts 파일이 담당한다.

다만 초기 Next는 서버/클라이언트 경계가 흐릿하고 데이터 권위에 대한 포지셔닝이 애매하여 다음과 같은 시스템을 도입한다.

---

### Next App Router

목적: <mark>"어플리케이션의 구조를 코드 규칙으로 고정하자"</mark>

규칙1. **폴더 = URL**
```markdown
app/
  page.tsx        → /
  products/
    page.tsx      → /products
```
=> <mark>폴더 구조가 곧 서비스 구조이다.</mark>

규칙2. **파일명 = 역할**
| page.tsx | layout.tsx | loading.tsx | error.tsx| route.ts |
| --- | --- | --- | --- | --- |
| URL 진입점 | 공통 프레임 | 로딩상태 | 에러 바운더리 | API 엔드포인트 |

=> <mark>반드시 이름을 똑같이 작성해야 작동한다.</mark>

규칙3. **기본 실행 위치는 서버**

모든 컴포넌트가 기본적으로 서버 컴포넌트로 선언된다.

=> <mark>page.tsx와 그외 server component로 구분되는데 일반 server component는 URL로 노출되지 않고 page.tsx의 조각으로 포함된다.</mark>

규칙4. **Client Component는 "예외 선언"**
```ts
"use client";
```

Client Component만 가능한 기능
- **useState**
- **useEffect**
- **onClick**
- **브라우저 API**

=> <mark> 상호작용이 필요한 부분만 클라이언트로 격리시킨다.</mark>

결국 이러한 구조를 통해 데이터 fetch와 기준을 **서버의 책임**으로 전환하고 브라우저 **상태변화를 최소화**함으로써 CSR의 데이터 권위를 구조적으로 서버에 고정시킨다.

전체 구조
```markdown
URL 요청
 ↓
App Router가 폴더 기준으로 페이지 결정
 ↓
Server Component 실행
 ↓
Server fetch + 캐시/재검증 판단
 ↓
필요 시 route.ts(API) 호출
 ↓
HTML or React Tree 생성
 ↓
브라우저는 필요한 부분만 hydration
```

=> <mark>브라우저는 일꾼이 아닌 표현자로서 역할을 담당한다.</mark>

---

### 실무 디렉토리 분석

실무 디렉터리 분석
```markdown
src/
├─ app/  
│  # ⭐ Next App Router의 중심
│  # - URL 라우팅
│  # - 페이지 진입점
│  # - 서버 렌더링/캐시/ISR 제어
│
│  ├─ layout.tsx
│  │  # (Server)
│  │  # 모든 페이지에 공통으로 적용되는 루트 레이아웃
│  │  # 헤더/푸터/Provider 배치
│
│  ├─ page.tsx
│  │  # (Server)
│  │  # "/" 루트 페이지
│  │  # 페이지 조립자 역할 (데이터 패칭 + 컴포넌트 조합)
│
│  ├─ loading.tsx
│  │  # 페이지 로딩 중 표시할 UI
│
│  ├─ error.tsx
│  │  # 에러 바운더리 (서버/클라 에러 처리)
│
│  ├─ (public)/
│  │  # URL에는 보이지 않는 그룹 라우트
│  │  # 공개 페이지 묶음 (로그인 불필요)
│
│  │  ├─ products/
│  │  │  ├─ page.tsx
│  │  │  │  # (Server) /products
│  │  │  │  # 상품 목록 페이지 (ISR/SSR 가능)
│  │  │
│  │  │  └─ [id]/
│  │  │     └─ page.tsx
│  │  │        # (Server) /products/:id
│  │  │        # 상품 상세 페이지
│  │  │
│  │  └─ about/
│  │     └─ page.tsx
│  │        # (Server) /about
│
│  ├─ (auth)/
│  │  # 인증 관련 페이지 그룹
│
│  │  ├─ login/
│  │  │  └─ page.tsx
│  │  │     # (Server) /login
│  │
│  │  └─ register/
│  │     └─ page.tsx
│  │        # (Server) /register
│
│  ├─ (app)/
│  │  # 로그인 이후 사용자 영역
│
│  │  ├─ dashboard/
│  │  │  └─ page.tsx
│  │  │     # (Server) /dashboard
│  │
│  │  └─ orders/
│  │     └─ page.tsx
│  │        # (Server) /orders
│
│  └─ api/
│     # ⭐ API Routes (백엔드 엔드포인트)
│     # - HTTP 요청 처리
│     # - 입력 검증
│     # - 서비스 호출
│
│     ├─ products/
│     │  └─ route.ts
│     │     # (Server)
│     │     # /api/products
│     │     # GET: 상품 조회
│     │     # POST: 상품 생성
│     │
│     ├─ orders/
│     │  └─ route.ts
│     │     # /api/orders
│     │
│     └─ auth/
│        └─ route.ts
│           # /api/auth
│
├─ features/
│  # ⭐ 도메인(업무) 중심 코드
│  # - 페이지와 무관
│  # - 진짜 비즈니스 로직의 집합
│
│  ├─ products/
│  │
│  │  ├─ components/
│  │  │  # UI 컴포넌트들 (서버/클라 구분)
│  │
│  │  │  ├─ ProductList.server.tsx
│  │  │  │  # (Server Component)
│  │  │  │  # 상품 리스트 UI (데이터 기반, 상호작용 없음)
│  │  │
│  │  │  ├─ ProductCard.server.tsx
│  │  │
│  │  │  ├─ FilterBar.client.tsx
│  │  │  │  # (Client Component)
│  │  │  │  # 필터/정렬/검색 (useState, onClick)
│  │  │
│  │  │  └─ AddToCart.client.tsx
│  │  │     # (Client Component)
│  │  │     # 장바구니 추가 (상호작용)
│  │
│  │  ├─ services/
│  │  │  # ⭐ 비즈니스 유스케이스
│  │  │
│  │  │  ├─ server/
│  │  │  │  # 서버 전용 서비스
│  │  │  │  # - DB 접근
│  │  │  │  # - 비즈니스 규칙
│  │  │
│  │  │  │  ├─ getProducts.ts
│  │  │  │  │  # 상품 목록 조회
│  │  │  │
│  │  │  │  ├─ getProductById.ts
│  │  │  │
│  │  │  │  └─ createProduct.ts
│  │  │  │     # 상품 생성 로직
│  │  │
│  │  │  └─ client/
│  │  │     # 브라우저 전용 서비스
│  │  │     # - /api 호출 래퍼
│  │  │
│  │  │     ├─ products.api.ts
│  │  │     │  # fetch('/api/products')
│  │  │
│  │  │     └─ cart.api.ts
│  │
│  │  ├─ validators/
│  │  │  # ⭐ 런타임 입력 검증
│  │  │
│  │  │  ├─ product.schema.ts
│  │  │  │  # Zod 스키마
│  │  │  │  # 서버 입력 방어선
│  │  │
│  │  │  └─ index.ts
│  │
│  │  ├─ types.ts
│  │  │  # (옵션)
│  │  │  # 도메인 타입
│  │  │  # 보통 schema에서 z.infer로 생성
│  │
│  │  └─ constants.ts
│
│  └─ orders/
│     # 상품과 동일한 구조
│
├─ lib/
│  # ⭐ 인프라 / 연결 레이어
│  # - 도메인 지식 없음
│  # - 기술 교체에만 반응
│
│  ├─ supabase/
│  │  ├─ server.ts
│  │  │  # 서버용 Supabase 클라이언트
│  │  │  # (Service Role Key 사용 가능)
│  │
│  │  ├─ client.ts
│  │  │  # 브라우저용 Supabase 클라이언트
│  │  │  # (Public Anon Key)
│  │
│  │  └─ middleware.ts
│
│  ├─ auth/
│  │  ├─ getSession.ts
│  │  │  # 서버 세션 조회
│  │  └─ requireUser.ts
│  │     # 권한 가드
│
│  ├─ http/
│  │  ├─ serverFetch.ts
│  │  │  # 서버 fetch 래퍼 (캐시/재검증 옵션)
│  │  └─ clientFetch.ts
│  │     # 클라이언트 fetch 래퍼
│
│  └─ env.ts
│     # 환경변수 검증/로딩
│
├─ components/
│  # ⭐ 완전 범용 UI 컴포넌트
│  # 도메인 지식 ❌
│
│  └─ ui/
│     ├─ Button.tsx
│     ├─ Modal.tsx
│     └─ Input.tsx
│
├─ styles/
├─ utils/
└─ types/
```
#### app/
이 폴더는 **서비스의 진입점**이다.

api/의 route.ts는 클라이언트의 **request 입구**까지만 담당한다.
- HTTP 메서드 정의
- 요청 파싱
- 입력값 검증
- 인증/권한 체크
- 서비스 함수 호출
- HTTP Response 생성

()로 디렉터리 이름을 묶은 이유는 그룹화를 시키고싶지만 실제 URL에서는 노출시키고 싶지 않을때 사용한다.

paget.tsx는 주로 초기데이터를 패칭하는 로직을 호출해주면 되겠다.

api/에서는 우리제품에 대한 api뿐만 아니라 외부 api 엔드포인트도 정의한다.

#### feature/
실제 **업무 로직**을 **도메인 중심**으로 분리시킨 폴더이다.

Server / client 컴포넌트 모두 가능하고 page.tsx는 오케스트레이터로서 이 안에 코드를 조립만 한다.

특히 services 폴더는 server와 client로 쪼개어 <mark>server 내부에서 필요한 서비스 로직은 server에, 사용자의 직접적인 상호작용과 관계되는 부분은 client폴더에 적용</mark>시켜주면 되겠다. clinet폴더는 결국 <mark>기존에 내가 사용하던 api호출 및 query작성 기능만을 담당하는것이 아키텍쳐의 원칙에 부합</mark>할 것이다.

validator폴더는 **스키마를 정의**하고 **런타임 검증**을 하는 것이다.

```ts
import { z } from "zod";

export const createProductSchema = z.object({
  name: z.string(),
  price: z.number().min(0),
});

export type CreateProductInput = z.infer<typeof createProductSchema>;
```
zod, yup같은 라이브러리를 통해 실제 런타임에서 타입검증이 실행된다.

#### lib/ 
**인프라**와 **서버권한**의 영역이다.

DB, 인증, 외부서비스 접근을 담당한다. 조작이 아닌 **연결**이 핵심이다.

---

### 실무중심의 개념이해

#### 데이터 패칭의 위치
기본규칙: 서버/클라이언트 컴포넌트 모두에서 일어날 수 있다.
실무: <mark>서버우선, 클라이언트 최소화</mark>
역할
- 서버: 초기데이터, SEO 데이터, 보안데이터 → page.tsx, layout.tsx
- 클라이언트: 필터, 무한스크롤, 좋아요 등 사용자 상호작용 →  app/api/에서 정의한 엔드포인트로 호출

#### 서버컴포넌트와 클라이언트 컴포넌트 분리 전략
- 서버: 데이터 패칭, DB접근, 캐시/ISR 제어
- 클라이언트: 상태관리, 클릭 이벤트, 즉각적인 UI 반응

#### route.ts 역할
**서버로직을 HTTP 엔드포인트로 노출**하는 요청 핸들러이다. 실제 비즈니스 로직은 services로 위임한다.

#### lib vs services
- lib: DB, Supabase, 인증 같은 **기술을 연결하는 인프라 레이어**로 도메인 의미 X
- services: **use-case layer**로 실제 업무의 의미를 갖는다.

#### validator 실무적 역할
주로 route.ts같은 외부 경계 레이어에 사용되며 schema는 **단일 진실 원천**으로 두고 **z.infer**로 타입을 생성하는 패턴을 사용한다.

#### page.tsx의 실무적 역할
페이지의 **설계도**이자 **조립자**로서 어떤 데이터가 필요하고, 어디서 실행되며, 어떤 컴포넌트가 상호작용을 담당하는지를 결정한다

## Next 실무패턴 딥다이브

### Middleware
정의:  <mark>페이지 코드가 실행되기 전에 서버가 먼저 한번 보는 검사대</mark>

목적: 
**통과**/**반려**/**리다이렉션**

위치:
// 항상 src바로 아래
```markdown
src/
  middleware.ts  
  app/
    page.tsx
```
실무패턴:
- **인증 체크**
```markdown
/admin → 로그인 안 됨 → /login
```
- **리다이렉트**
```markdown
/ → /products
```
---
### Server Actions
정의: <mark>Client에서 서버함수를 직접 호출하게 해주는 기능</mark>

위치:
```markdown
src/
  features/
    products/
      actions/
        createProduct.action.ts
        updateProduct.action.ts
      services/
        server/
          createProduct.ts
      components/
        ProductForm.client.tsx
```

기능: <mark>기존 fetch + /api/route.ts를 건너 뛰게 해준다.</mark>

목적: 기능에 따라 **너무 과한 서비스 레이어의 동작을 최소화**하기 위함

예시
- form submit
- 버튼 클릭
- 상태 변경
```tsx
// actions.ts
"use server";

export async function createProduct(formData: FormData) {
  // DB 로직
}

// button.tsx
<form action={createProduct}>
  <button>추가</button>
</form>
```

=> <mark>페이지 전환 없는 상태 변경이나 단순 POST/PUT에서 쓴다.</mark>