---
bg: "BACKEND.png"
layout: post
title:  "Mysql 5.7 이상 root 비밀번호 잃어버렸을 때"
crawlertitle: "Mysql 5.7 이상 root 비밀번호 잃어버렸을 때"
summary: "Mysql 5.7 이상 root 비밀번호 잃어버렸을 때"
date: 2021-06-21 17:00:00 +0900
category: Backend
author: Err0rCode7
---

#### Mysql 5.7 이상 root 비밀번호 찾기
##### root 비밀번호 찾는 삽질 기록 합니다.

0. 비밀번호 스킵으로 접속

   ```sql
   sudo mysqld --skip-grant
   ```

1. 비밀번호 제거

```sql
update user set authentication_string=null where user='root';
flush privileges;
exit

mysql -uroot
```

2. 비밀번호 설정

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'root';

flush privileges;
exit;
```

---

**추가로 새로운 계정생성**

3. 계정생성

   ```sql
   create user 'test'@'%' identified by 'test';
   ```

4. 모든 권한 부여

   ```sql
   grant all privileges on *.* to test@localhost identified by '비밀번호' with grant option;
   ```

