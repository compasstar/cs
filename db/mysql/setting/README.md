
# 🛠️ 초기세팅

## 1. MySQL 설치

## 2. CMD 창에서 MySQL 접속
```
mysqlsh -u root -p
```
## 3. SQL 모드로 전환 (JS, PY 모드로 적용되어 있을시)
```
\sql
```

## 4. 데이터베이스(스키마) 생성
```
-- 데이터베이스 생성 (권장 이름: sandbox_db)
CREATE DATABASE sandbox_db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

-- 확인: 데이터베이스 목록 확인
SHOW DATABASES;
```

## 5. 계정생성
```
-- 1. 새 사용자 생성 및 비밀번호 설정 
-- 'localhost'는 로컬 컴퓨터에서만 접속을 허용합니다.
CREATE USER 'haeri'@'localhost' IDENTIFIED BY '당신의_강력한_비밀번호_입력';

-- 2. 생성한 사용자에게 sandbox_db에 대한 모든 권한 부여
-- 'sandbox_db.*'는 해당 DB의 모든 테이블을 의미합니다.
GRANT ALL PRIVILEGES ON sandbox_db.* TO 'sandbox_user'@'localhost';

-- 3. 변경 사항 적용 (필수!)
FLUSH PRIVILEGES;

-- 4. 확인: 방금 만든 사용자가 목록에 있는지 확인
SELECT user, host FROM mysql.user WHERE user = 'haeri';
```

## 6. IntelliJ IDEA 에서 MySQL 연결
Dadabase 탭에서 Data Souce > MySQL 선택

|설정 항목 |입력 값|
| --- | --- | 
|Name|sandbox_mysql|
|Host|localhost|
|Port|3306|기본 포트|
|User|haeri |
|Password|설정한 비밀번호|
|Database|sandbox_db|

### 테스트 테이블 만들기

```
show databases;

use sandbox_db;

create table test_user (
    id int not null auto_increment,
    username varchar(50) not null unique,
    created_at DATETIME default current_timestamp,
    primary key (id)
);

select * from test_user;

drop table test_user;
```


## 7. 스프링 부트 application.yaml 파일 적용

```
spring:
  datasource:
    # URL에 Host, Port, Database 이름이 정확한지 확인
    url: jdbc:mysql://localhost:3306/sandbox_db?serverTimezone=UTC&characterEncoding=UTF-8
    username: haeri
    password: 비밀번호_입력
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```
