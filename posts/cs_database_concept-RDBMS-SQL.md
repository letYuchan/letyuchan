---
title: '데이터베이스 기초'
date: '2026-01-30'
category: 'CS'
tags: ['DB', 'RDBMS', 'NoSQL', 'SQL']
summary: '데이터베이스 기본 개념 이해가기'
---

# 데이터베이스 기본 개념 이해
## 데이터베이스 필수개념
### 용어 정리

Database: 
**구조**와 **규칙**그리고 **영속성**을 가진 데이터의 집합

DBMS(DB-Management System):
DB를 **생성·관리·보안·조회**하게 해주는 소프트웨어

RDBMS:
관계형 DB를 관리하는 DBMS

SQL(Structured Query Language): 
DB에 명령하는 언어

---

### 관계형 vs 비관계형

#### 관계형DB
-  정의: 
데이터를 <mark>표(테이블)로 저장하고 테이블 간 관계를 명시적으로 정의</mark>하는 DB
```txt
users
--------------------------------
id | email | created_at
```
- 종류: PostgreSQL, MySQL, Oracle, SQLite
- 특징: 고정된 Schema / **Column 타입 강제** / **PK, FK 존재** / **SQL(Query Lang)**
- 장점: **데이터무결성** / **트랜잭션 강력**  / 비즈니스 로직에 강함 / JOIN으로 결합

#### 비관계형DB(NoSQL)
- 정의:
<mark>관계를 강제하지 않고 확장성·유연성을 우선</mark>한 DB
- 종류:
```txt
// Key - Value
"user:1" → { name: "kim" }
// Redis - 빠르다 → 캐시, 세션
```
```json
// Document
{
  "userId": 1,
  "posts": [...]
}
// MongoDB
```
Cassandra - Column, 대규모 분산
Neo4j - Graph, 관계 중심
- 장점: Schema 유연 / 수평 확장 쉬움 / **읽기 빠름**
- 단점: JOIN 없음 / **무결성을 직접 관리** / **트랜잭션 약함**

#### 표 정리
| 항목 | 관계형 DB (RDBMS) | 비관계형 DB (NoSQL) |
| --- | --- | --- |
| 데이터 구조 | 테이블 (Row / Column) | 문서 / 컬렉션 / Key-Value |
| 스키마 | 고정 스키마 (사전 정의) | 유연한 스키마 (없거나 느슨) |
| 관계 표현 | PK / FK 로 명시적 관계 | 관계 없음 (중첩 or 코드 처리) |
| 데이터 무결성 | DB가 제약조건으로 보장 | 애플리케이션이 직접 책임 |
| 트랜잭션 | ACID 보장 (강력) | 제한적 또는 없음 |
| 조회 방식 | SQL + JOIN | 단일 문서 / 키 기반 조회 |
| 확장 방식 | 수직 확장 중심 | 수평 확장에 유리 |
| 주 사용 사례 | 핵심 비즈니스 데이터 | 캐시 / 로그 / 세션 |

=> <mark>관계형은 규칙과 신뢰를, 비관계형은 확장과 유연함을</mark>

## 관계형 딥다이브

### 개념 및 용어 정리
<img src="https://blog.kakaocdn.net/dna/b5v1nS/btrqFpl1ckk/AAAAAAAAAAAAAAAAAAAAAJ3nEYsziOsouzMNbAfEgyWs5dosB1xpeIKuIDUD2clF/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2FYSGklBJpfuI2zVerEK04LScWY0%3D" alt="image" height="600px" width="400px" />

- Table: 관계형 DB의 구조(Relation이라고도 부름)
- row: 하나의 데이터(Record, Tuple이라고도 부름)
- column: 속성(Field, Attribute 라고도 부름)
- schema: column의 구조 정의
- Degree: Column의 개수
- Cardinality: Row의 개수
- Domain: 하나의 컬럼이 취할 수 있는 원자 값들의 집합 Ex) 성별: 남/여

---

### Key
#### **Primary Key (PK)**
- 정의: <mark>각 행을 유일하게 식별하는 Column</mark>
- 규칙: **Unique(**도메인 값 중복 X)와 **NOT NULL**, **최소성**(후보키 중 어느 하나라도 제거하면 유일성을 잃는 성질) 보장
- 특징: 관계연결과 수정/삭제의 기준점
- 예시: id, UUID, auto increment number

#### **ForeignKey(FK)**
- 정의: <mark>다른 테이블의 PK를 참조하는 키(관계 형성)</mark>
- 규칙: **NULL** or **PK의 도메인**
- 특징: **참조무결성** 보장 → 참조키값이orders.user_id=123 이면, users.id=123 이 반드시 존재해야 함
- 예시: orders.user_id → users.id 를 가리킴

#### Candidate Key
- 정의: 기본키로 사용 가능한 Column

#### Alternate Key: 
- 정의: 후보키가 둘 이상일때 PK를 제외한 나머지 키

#### Super Key
- 정의: 한 속성들의 집합으로 구성된 키
- 특징: 유일성(Unique)는 만족하지만 최소성은 불만족

---

### 무결성(Integrity)
목적: <mark>데이터가 망가지지 않게 하는 안전벨트</mark>

#### 무결성 제약조건
1. **개체 무결성**: <mark>각 Record는 고유해야한다</mark> → PK로 보장
2. **도메인 무결성**: <mark>각 Field는 Domain에 지정된 값만 가짐</mark>
3. **참조 무결성**: <mark>FK가 참조하는 대상은 반드시 존재한다</mark> → NOT NULL or PK의 값

=> DB에서 보장함으로써 **최후의 방어선**으로서 역할

---

### 트랜잭션(Transaction)
![트랜잭션](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRWr7781K_uVGJqravrtHozf3o_oyJg9b1XaA&s)

트랜잭션은 <mark>여러 쿼리를 하나의 작업처럼 묶어 "전부 성공" or "전부 취소"를 보장하는 작업단위</mark>

ACID를 보장
 - **Atomicity**: <mark>연산이 정상적으로 수행(Commit) or 어떠한 연산도 수행 X(Rollback)</mark>
- **Consistency**: <mark>시스템 고정요소가 트랜잭션 수행 전/후 동일</mark>
- **Isolation**: <mark>개별 트랜잭션은 다른 트랜잭션을의 간섭 및 영향X</mark>
- **Durability**: <mark>Commit된 트랜잭션 결과는 영구적으로 기록됨</mark>

---

### 정규화(Normalization) 

목적: <mark>중복을 줄이고, 수정/삭제/추가할 때 데이터가 망가지지 않게 테이블을 쪼개는 설계 규칙</mark>

문제상황
- 갱신 이상: 어떤곳은 바뀌고 어떤 곳은 그대로
- 삽입 이상: 필요없는 정보도 포함해야 삽입 가능
- 삭제 이상: 하나 삭제했는데 다른것도 삭제

목표
- <mark>한 테이블은 하나의 주제만</mark>
- 반복/리스트 구조는 분리
- <mark>하나의 PK에 대해 나머지 모든 컬럼 값이 직접 종속 되도록 설계(제 2 정규형)</mark>

=> **중복을 제거**하는게 핵심

정규화 과정
1. **제 1 정규형**: <mark>모든 Domain은 원자 값으로만 정의</mark>
2. **제 2 정규형**:<mark> 간접 함수 종속이 아닌 완전 함수적 종속을 만족</mark>
3. **제 3 정규형**: <mark>기본키가 아닌 키들이 이행적 함수 종속 관계(X → Y, Y → Z면 X → Z) 만족 X</mark>

제 2 정규형 심화
```txt
orders
--------------------
id (PK)
user_id        ← PK에 직접 종속 (OK)
user_email     ← user_id에 종속 (❌ 간접 종속)

올바른 구조는 다음과 같다.
users
--------------------
id (PK)
email

orders
--------------------
id (PK)
user_id (FK)
```

제 3 정규형 심화
```txt
orders
--------------------
order_id (PK)
user_id
user_email ❌

종속 관계는 다음과 같다.
order_id → user_id
user_id  → user_email
```

이걸 못지키면
- 유저 이메일 변경하면 모든 주문 레코드 수정 → 갱신 이상
- 주문 없이 이메일 저장 불가 → 삽입 이상
- 마지막 주문 삭제시 이메일 정보 유실 → 삭제 이상

따라서 정답 구조는 다음과 같다.
```txt
users
--------------------
id (PK)
email

orders
--------------------
id (PK)
user_id (FK)
```
---

### Index
목적: **검색을 빠르게** 하기 위한 정렬된 구조
```sql
CREATE INDEX ON users(email);
```
- 장점: 조회 빠름
- 단점: **쓰기 느려짐**, **저장공간 사용**

## SQL

### 분류
| 분류| 이름 | 역할 |
| --- | --- | --- |
| DDL | Data Definition Language | 구조 정의 |
| DML | Data Manipulation Language | 데이터 조작 |
| DCL | Data Control Language | 권한/보안 |

---

### DDL
목적: 테이블 구조 만들기

명령어:
```sql
CREATE TABLE
ALTER TABLE
DROP TABLE
```

예제:
```sql
CREATE TABLE users (
  id uuid PRIMARY KEY,
  email text UNIQUE NOT NULL,
  created_at timestamptz DEFAULT now()
);
```
위 예제는 무결성 제약조건을 SQL로 선언

---

### DML
목적: 데이터 다루기

명령어:
삽입 - 컬럼 명시하기
```sql
INSERT INTO users (id, email)
VALUES ('uuid-1', 'test@example.com');
```
조회
```sql
SELECT *
FROM users
WHERE email = 'test@example.com';
```
갱신 - WHERE을 꼭 사용하기
```sql
UPDATE users
SET email = 'new@example.com'
WHERE id = 'uuid-1';
```
삭제
```sql
DELETE FROM users
WHERE id = 'uuid-1';
```

---

### DCL
목적: 누가 무엇을 할 수 있는지 정의
명령어:
```sql
GRANT    -- 권한 부여
REVOKE   -- 권한 회수
CREATE ROLE -- 역할 생성
```
예시:
```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON orders
TO app_user;
```
CRUD 권한 부여 패턴

---

### JOIN
목적: 정규화로 쪼개진 테이블 합쳐서 조회할때 사용

예시:
LEFT JOIN
```sql
SELECT
  u.id,
  u.name,
  o.id AS order_id
FROM users u
LEFT JOIN orders o
  ON o.user_id = u.id;
```
실무에서 주로 사용하는 LEFT JOIN  → 유저는 모두, 주문 없는 유저는 order_id를 NULL로 표현(빼지는 않음) 

INNER JOIN
```sql
SELECT
  o.id,
  o.total_price,
  u.name
FROM orders o
INNER JOIN users u
  ON o.user_id = u.id;
```
매칭이 안되면 결과에서 빠짐

---

### WHERE

목적: 조건 설정
조건 종류
- 비교: =, !=, <, >
- 집합: IN (...)
```sql
SELECT *
FROM orders
WHERE status IN ('PAID', 'SHIPPED', 'DELIVERED');
```
- 범위: BETWEEN
- NULL 체크: IS NULL, IS NOT NULL
- and, or: AND, OR
- 패턴: LIKE
```sql
-- 이름에 '노트' 포함
SELECT *
FROM products
WHERE name LIKE '%노트%';
```
- 존재여부: EXISTS, NOT EXISTS
```sql
-- 주문을 한 번이라도 한 유저
SELECT *
FROM users u
WHERE EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.user_id = u.id
);
```

---

### ORDER BY
목적: 조회한 결과를 어떤 방향으로 정렬할지 정의
명령어:
- ASC: 오름차순(DEFAULT)
- DESC: 내림차순
예시:
```js
SELECT *
FROM posts
ORDER BY created_at DESC;
```

---

### TCL
목적: 여러 DML을 하나의 작업으로 묶기
명령어:
- BEGIN: 시작
- COMMIT: 변경사항 확정
- ROLLBACK: 전체취소
- SAVEPOINT: 체크포인트 생성

예시:
```sql
BEGIN;

INSERT INTO orders (...);
SAVEPOINT after_order;

UPDATE products SET stock = stock - 1 WHERE id = 10;

-- 실패하면
ROLLBACK TO after_order;

COMMIT;
```