---
title: 'Supabase 배우기'
date: '2026-01-31'
category: 'Dev'
tags: ['Supabase', 'PostgreSQL', 'Auth']
summary: 'Supabase에 대한 이해를 바탕으로 활용법 배우기'
---

# Supabase 기초

## PostgreSQL
PostgreSQL은 **산업용 RDBMS**이다. Supabase가 해당 RDBMS를 사용하며 다음과 같은 특징을 갖는다.

특징:
- **ACID Transaction에 강력**
- **JSON 지원**(NoSQL 흉내)
- 확장성 좋음
- 오픈소스
- 대기업/핀테크/금융에서 사용

=> <mark>비즈니스 데이터에 가장 안전한 DB</mark>

구조: Table(Row&Column)구조
- Row: Record, Tuple  → 하나의 유일한 정보모음
- Column: Field, Attribute  → 원자값으로 이루어진 도메인(속성)

기본개념:
- Primary Key: 각 Row의 신분증 → 최소성 + 유일성 + NOT NULL
- Foreign Key: 다른 테이블의 PK 참조  → 관계 형성
- Transaction: 여러 쿼리를 한 묶음으로 실행

## Supabase

### What&Features
Supabase란 <mark>PostgreSQL을 중심으로 Auth, API, 보안(RLS)를 한번에 제공하는 BE 플랫폼</mark>

주요 기능표
| 구성 | 설명 |
| --- | --- |
| PostgreSQL | DBMS 
| Auth | 로그인/세션 |
| REST API | API 자동 생성 |
| RLS | DB 레벨 보안 |
| Storage | 파일 저장 |
| RealTime | 실시간 구독 |

---

### 상세 기능

####  REST API
Supabase는 <mark>API 서버 구축을 최소화</mark> 시켜줌

기존:
```txt
Client → route.ts → DB
```
supabase:
```txt
Client → Supabase → DB
```

#### Auth
<mark>사용자 인증·인가 시스템 제공</mark>

기능
- **회원가입**
- **로그인**
- 세션 유지
- **JWT 발급**
- 유저 식별
```ts
const { data } = await supabase.auth.signInWithPassword({
  email,
  password,
});
```
단 명심할 것은<mark> 로그인 했다고 데이터를 모두 보여주는게 아닌 RLS통과 단계도 설정</mark>해놔야 한다.

#### **RLS(Row Level Security)**
<mark>이 Row를 이 유저가 볼 수 있는가를 DB가 판단</mark>하는 시스템

배경: 
기존 프론트에서 id를 체크하고 API응답에 의한 조건분기로 사용자 식별·다이렉션 → 보안 리스크 매우 큼

구조:
```txt
Client
 → Supabase API
   → PostgreSQL
     → RLS 검사
       → 허용 / 차단
```

예시:
```sql
CREATE POLICY "users can read own posts"
ON posts
FOR SELECT
USING (auth.uid() = user_id);
```

실무 RLS 패턴
**자기 데이터만 조회**
```sql
auth.uid() = user_id
```
**로그인한 사람만**
```sql
auth.role() = 'authenticated'
```
**관리자만**
```sql
auth.jwt() ->> 'role' = 'admin'
```
이러한 정책 설정은 **본인이 직접 지정**해줘야 한다.

기본 role값은 **'authenticated'**이며, <mark>수정사항은 반드시 서버컴포넌트에서 진행</mark>해야함을 유의하자.

보통 어드민 체크는다음과 같은 함수로 빼놓는다
```sql
CREATE FUNCTION is_admin()
RETURNS boolean
LANGUAGE sql
STABLE
AS $$
  SELECT (auth.jwt() -> 'app_metadata' ->> 'role') = 'admin';
$$;

이걸 RLS에서 아래와 같이 사용
USING (is_admin());
```

#### **Real-Time**
<mark>DB에서 데이터가 바뀌는 순간을 클라이언트가 즉시 감지하는 기능</mark>

기존:
```txt
Client
 └─ 3초마다 API 요청
     └─ 데이터 바뀌었나?
```
기존 클라이언트의 **Polling** 방식은 **서버 부하**, **네트워크 낭비**, **UX 저하**, **새로고침 필수**라는 문제를 야기한다.

Supabase의 Realtime은 <mark>Client컴포넌트가 DB 변경에 대한 구독을 하면 DB가 이벤트를 변경감지 후 자동 푸싱</mark>해준다.

구조:
```txt
Client
 └─ WebSocket 연결
     └─ Supabase Realtime Server
         └─ PostgreSQL WAL(Log)
```

예시:
```ts
supabase
  .channel("posts")
  .on(
    "postgres_changes",
    { event: "INSERT", schema: "public", table: "posts" },
    payload => {
      console.log("새 글!", payload.new);
    }
  )
  .subscribe();
```

주로 **채팅·알림·좋아요/조회수·관리자 대시보드**에서 사용한다.

=> <mark>Real-Time은 DB 변경 알림 시스템이다.</mark>

## Supabase 실무패턴

### Server&Client Component

#### Server Component
- **DB에서 데이터를 미리 Read하여 완성된 HTML을 내려줌**
- 초기 렌더링이 빠르고 SEO에 유리
- **이벤트가 없어 Write 불가**
#### Client Component
- 버튼 클릭, 폼 제출 등 사용자 이벤트 처리
- **이벤트를 트리거로 Write(INSERT/UPDATE/DELETE) 수행**
- DB 상태 변경은 항상 **사용자 의도**에서 시작됨

Server:
```ts
// page.tsx
const products = await supabase.from("products").select("*");

return <ProductList products={products} />;
```

Client:
```js
"use client";

const likeProduct = async (id) => {
  await supabase.from("likes").insert({ product_id: id });
};
```
<mark>RLS로 권한 설정을 완벽히 해야한다.</mark> 
 추가로 단순한 CRUD(Ex: 닉네임 수정)에 대해서만 사용할 것

=> <mark>Read는 Server, Write는 Client → 성능 · 보안 · 책임 분리에 강한 구조</mark>

### route.ts

<mark>Supabase는 DB담당
Route.ts는 비즈니스/외부 로직 담당</mark>

Route.ts는 주로 아래와 같은 상황에서 활용한다.

1. Service Role Key(어드민 키)가 필요한 작업

2. 입력검증과 비즈니스 로직이 복잡할 때

3. 외부 API 혹은 Secret Key 사용시

4. 웹훅 - 외부에서 우리 서비스로의 알림이 필요할 때

5. 파일 업로드시 검사가 필요할 때

6. Rate Limit/Bot 방지/Log

=> <mark>브라우저에 맡기면 위험한 일은 route.ts에서 한다.</mark>

#### **권장 운영 원칙**
1. Read는 서버 컴포넌트에서 Supabase로 직접 한다.
2. Write는 **Server Actions** or **route.ts**를 통해서 한다.
3. **단순한 Write**에 한해서 클라이언트 컴포넌트에서 Supabase로 직접 조작한다.





