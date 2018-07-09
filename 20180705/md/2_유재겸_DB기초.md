데이터베이스
===============
모든 내용은 [생활코딩](https://opentutorials.org/course/1)을 참조하였음을 미리 알려드립니다.

#### 목차<br>
1. 데이터 베이스란?
2. MYSQL 사용하기<br>
2-1. 데이터베이스<br>
2-2. 테이블<br>
2-3. 데이터
<br>
3. 실습
----
#### 1. 데이터 베이스란??
우선 데이터베이스라는 것이 뭔지 봅시다.<br>
데이터베이스(DB)의 사전적 의미는 체계화된 데이터의 모임입니다.<br>


![](https://user-images.githubusercontent.com/37801624/42411525-3987ca34-8238-11e8-8b7d-6ff15baca47e.PNG)
<br> (그림 1)(구조화 되어있지 않은 데이터)

그림1과 같이 구조화 되어있지 않은 상태로 정보를 전달하게 되면,
정보가 한눈에 들어오지 않고, 원하는 정보를 찾아내기 어렵습니다.
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/42411526-3c8b42b0-8238-11e8-8e5b-2ee60faf3b12.PNG)
<br>(그림 2)(구조화 되어있는 데이터)
<br><br>그림2는 그림1을 구조화한 것입니다.
<br>정렬되어있지 않은 정보들을 구조화시키면, 원하는 정보를 쉽게 찾을 수 있습니다.
<br>


#### 2. MYSQL 사용하기
![](https://s3.ap-northeast-2.amazonaws.com/opentutorials-user-file/module/98/320.png)
<br>(그림 3)(이미지 출처 [생활코딩](https://opentutorials.org/course/1))
<br><br>
데이터베이스 서버중에 MYSQL이라는 것을 사용해보도록 하겠습니다.
그림2 와 같이 정보를 구조화한 표로 나타낸 것을 테이블이라고 합니다.<br>
테이블 여러개가 모여 한개의 데이터 베이스를 이루고,<br>
데이터베이스 여러개가 모여 한개의 데이터베이스 서버를 이룹니다.
MYSQL 설치방법은 [생활코딩](https://opentutorials.org/course/195/1464)을 참조하세요.
<hr>



#### 2-1. 데이터 베이스
데이터베이스 생성

      CREATE DATABASE `데이터베이스명` CHRACTER SET utf8 COLLATE utf8_general_ci;

데이터베이스 삭제

      DROP DATABASES `데이터베이스명`;
데이터베이스 목록보기

      SHOW DATABASES;
데이터베이스 사용하기

      USE `데이터베이스명`;



<br>
<hr>
### 2-2. 테이블

<br>

테이블 생성    

      CREATE TABLE 테이블명 (
      `칼럼명1` data-type NOT NULL,
      `칼럼명2` data-type NOT NULL,


테이블 삭제

      DROP TABLE 테이블명;

테이블 구조 출력

      DESC 테이블명;
<br>

---
### 2-3. 데이터

테이블 삽입

      INSERT INTO `테이블명` VALUES('value1','value2','value3',...);


      INSERT INTO `table_name` (`칼럼1`, `칼럼2`, `칼럼3`,...) VALUES ('value1', 'value2', 'value3',...);

데이터 수정

      UPDATE `데이터베이스 명` SET 칼럼1='value1', 칼럼2='value2' WHERE 칼럼3=value3;
      //(칼럼3의 값이 value3인곳에, 칼럼1과 칼럼2의 값을 각각 value1과 value2로 바꾸어라 라는 뜻입니다.)

데이터 삭제

      DELETE FROM `테이블명` [WHERE 삭제하려는 칼럼=value];

테이블 데이터 초기화

      TRUNCATE 테이블명;

데이터 검색

      SELECT 찾는 내용 FROM `테이블 명` WHERE `컬럼`='컬럼1의 값' AND `컬럼2`='컬럼2의 값';

정렬

      SELECT * FROM `테이블명` ORDER BY `칼럼1` [DESC(내림차순) | ASC(오름차순)];

      SELECT * FROM `테이블명` ORDER BY `칼럼1` [DESC(내림차순) | ASC(오름차순)] 칼럼2 [DESC(내림차순) | ASC(오름차순)];
      (칼럼1의 데이터를 기준으로 정렬하고, 동일한 데이터가 있다면 그 데이터들을
      칼럼2의 데이터를 기준으로 정렬)


---

### 3. 실습
![](https://user-images.githubusercontent.com/37801624/42417986-61311678-82d1-11e8-9cad-81665a928680.png)<br>
(mysql -u root -p 명령어를 통해서 설치된 mysql에 접속하고 비밀번호 입력(초기비밀번호 apmsetup)
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/42417987-63af6350-82d1-11e8-9f70-58d5be262337.png)<br>
('practice'라는 이름의 데이터베이스를 생성해준다.)
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/42417988-65ace614-82d1-11e8-9088-145217a3bd85.png)<br>
(show databases 를 통해서 practice라는 데이터베이스가 생긴 것을 확인)
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/42417989-6725f616-82d1-11e8-9e0e-d34724296fd4.png)<br>
(DROP DATABASE 'practice'로 데이터베이스를 삭제하고 확인했을 때
practice가 사라진 것을 볼 수 있습니다.)
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/42417990-6ae4af90-82d1-11e8-97f1-7968df45a68f.png)<br>
(다시 'practice'라는 데이터베이스를 만들어 줍시다.)
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/42417991-6dd90a20-82d1-11e8-8af3-401dbc3bb9f5.png)<br>
(테이블을 만들고 칼럼값, 변수형,공백유무,공백일경우에 들어가는 값을 넣어줍니다.)
![](https://user-images.githubusercontent.com/37801624/42417992-700b6fae-82d1-11e8-9b12-998e978ac365.png)<br>
(desc TABLE 테이블이름 으로 테이블 구조를 확인할 수 있습니다.)
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/42417993-728af588-82d1-11e8-8b73-8024ac796dfc.png)<br>
(만들어진 테이블에 데이터를 넣어봅시다. VALUES()안에 들어가있는 값들은 순서대로 들어가게됩니다.)
<br>
<br>
![](https://user-images.githubusercontent.com/37801624/42417994-74d98c00-82d1-11e8-9db9-919c740ffc3c.png)<br>
(select * from singsong 을 통해서 전체 테이블 구조를 확인할 수 있습니다.(원래 검색기능이기는 한데 자세한건 나중에 하도록 합시다.) 입력한 데이터 값이 들어간 것을 볼 수 있죠.)
<br>
<br>![](https://user-images.githubusercontent.com/37801624/42417995-77494782-82d1-11e8-93af-8cbd7721561e.png)<br>
(다시한번 데이터 값들을 넣어주고)
<br>
<br>![](https://user-images.githubusercontent.com/37801624/42417996-7936b958-82d1-11e8-97ad-899fb3696fec.png)<br>
(이건 다른 방식의 데이터 삽입입니다. 이전에 데이터 삽입 방식이 VALUES()안의 값이 순서대로 들어가는 방식이였다면, 이건  (컬럼1,컬럼2,...) VALUES (데이터1,데이터2,...) 를 통해서 데이터1이 컬럼1에 들어가고 데이터2가 컬럼2에 들어가는 방식입니다.)
<br>
<br>![](https://user-images.githubusercontent.com/37801624/42417998-7e1403f4-82d1-11e8-8142-2aeaad43a77d.png)<br>
(다음해볼것은 데이터 수정입니다. 그림과 같이 잘못된 데이터가 들어갔을때는 어떻게 해야될까요?)
<br>
<br>![](https://user-images.githubusercontent.com/37801624/42418001-810f5fa4-82d1-11e8-8948-de4d239998f5.png)<br>
(UPDATE문을 이용해서 수정하면 됩니다.UPDATE 테이블명 SET `칼럼1`='원하는 데이터 값',`칼럼2` = '원하는 데이터 값2' WHERE `칼럼3`='데이터3' (원하는 테이블의 칼럼3의 값이 데이터3으로 설정되어 있으면, 그 행의 칼럼1과 칼럼2의 값을 바꿉니다. ))
<br>
<br>![](https://user-images.githubusercontent.com/37801624/42418353-759e6554-82d9-11e8-871c-e97167525579.png)<br>
(마지막으로 정렬입니다. 위의 사진은 select * from 테이블명 order by date desc를 통해서
  singsong테이블 전체를 date컬럼을 기준으로 내림차순으로 정렬한 것과
  asc를 통해서 오름차순으로 정렬한 것으로 나뉩니다.)
<br>
<br>
+(comment)(내용은 얼마 안되는데 실습한 내용을 모두 보여주려고 하다보니 사진이 많아졌네요!
  다음엔 좀더 간결하고 많은 정보를 담을 수 있도록 고쳐보겠습니다.)
