# Bandit level 0→16 풀이
----------------------------------
* 모든 문제를 캡처해서 설명하면 문서가 불필요하게 길어질 것 같아서 표로 정리했습니다. 


## 0→5
| level     | 사용 명령어 및 옵션 | 출제의도 분석 | 비고 (풀이) |
| :-------: | :----------------: | :-----: | :----:|
| 0→1       | ls, cat       | 명령어 사용 | - |
| 1→2       | ls, cat       | 명령어 뒤에 쓰는 '-'문자의 의미(옵션 취급), 시스템의 구조(트리)와 파일 경로에 대한 개념 | cat ./- |
| 2→3       | ls, cat       | 파일 이름에 띄어쓰기가 포함 될 경우 읽는 방법(3가지) | 파일 이름이 'spaces in the filename'일 경우: ①cat spaces* ②cat spaces\ in\ the\ filename ③cat s+(tab키)   |
| 3→4       | ls, cd, ls -al       | hidden file 읽기 | ls -al |
| 4→5       | ls, cd, cat       | 1→2, 3→4 문제 복합형 | ls -al → cat ./-file07 |


#### * 참고 자료
x

----------------------------------
## 5→13

| level     | 사용 명령어 및 옵션 | 출제 의도 분석 | 비고 (풀이) |
| :-------: | :----------------: | :----: | :----: |
| 5→6       | ls, cd, find [위치] -size [크기 ex:1033c] | find 사용 | cd inhere → find . -size 1033c |
| 6→7       | cd, find [위치] -user [이름] -group [이름] -size [크기], 2>/dev/null | find 옵션의 중복 사용, 오류메시지 리다이렉트 | cd / → find . -user bandit7 -group bandit6 -size 33c 2>/dev/null  |
| 7→8       | ls, cat [파일] (파이프) grep [패턴] [파일] | 파이프 사용, grep 사용 | cat data.txt (파이프) grep millionth data.txt |
| 8→9       | ls, sort [파일] (파이프) uniq -u | 파이프의 의의, sort와 uniq 사용 | sort data.txt (파이프) uniq -u, 파이프에 의해 uniq 명령어의 인자값은 sort data.txt값으로 주어진다 |
| 9→10      | ls, xxd [파일] (파이프) grep -A4 '==' | xxd 사용, grep의 옵션 | xxd data.txt (파이프) grep -A4 '==', 'A'옵션은 해당 문자열 뒤로 4줄, 'B'옵션은 해당 문자열 앞으로 4줄, 'C'옵션은 해당 문자열 앞뒤로 4줄을 출력해 준다 |
| 10→11     | ls, cat | base64 | base64 decode |
| 11→12     | ls, cat | 시저암호 | 시저암호 해독기 사용 |
| 12→13     | - | 반복된 압축 풀기 | 문제 훼손으로 풀수 없어서 넘어감 |



#### * 참고 자료

>6→7 오류메시지 리다이렉션  
http://blog.shakii.co.kr/94

-------------------------------------------------
## 13→16

| level     | 사용 명령어 및 옵션 | 출제 의도 분석 | 비고 (풀이) |
| :-------: | :----------------: | :----: | :----: |
| 13→14     | ssh [옵션] 사용자@호스트이름 | SecureSHell명령을 통해 다음 레벨의 권한으로 원격 접속 | ssh -i ./sshkey.private bandit14@localhost → 다음 레벨 권한 획득 → 다음 레벨 pw 획득 |
| 14→15     | nc [타겟 호스트] [포트번호] | NetCat 명령어로 네트워크에 연결해서 데이터를 쓰고 읽기 | nc localhost 30000 → 현재 레벨 pw 제출 → 다음 레벨 pw 읽기 |
| 15→16     | openssl s_client [옵션] | openssl의 heartbleed 취약점(-ign_eof 옵션 사용) | openssl version(취약한 버전인지 확인) → openssl s_client -connect localhost:30001 → 'B'입력(heartbleed취약점 존재 확인) → openssl s_client -connect localhost:30001 -ign_eof → 현재 레벨 pw 보내기 → 다음 레벨 pw 획득 |


#### * 참고자료

>13→14   
1. secure shell 정의, ssh 명령어 예시 http://hack-cracker.tistory.com/131

>14→15   
1. 어느 곳에 접속할 때: nc [-옵션] hostname port,
접속을 기다릴 때: nc -l -p port [-옵션] [타겟 호스트] [포트]
2. netcat의 개념, nc명령어 사용 방법(링크 2개) http://htst.tistory.com/61,  http://blog.naver.com/PostView.nhn?blogId=carmine1025&logNo=220554474008


>15→16   
1. SSL/TSL설명, openssl s_client 명령어 사용방법 http://websecurity.tistory.com/100
2. heartbleed취약점 설명
 https://sangwook.github.io/2014/04/08/openssl-heartbeat-heartbleed.html
3. KISA>보안공지>OpenSSL 취약점(HeartBleed) 대응 방안 권고 (openssl 취약 버전 나와있음)
https://www.krcert.or.kr/data/secNoticeView.do?bulletin_writing_sequence=20884
4. -ign_eof 옵션 설명 https://www.openssl.org/docs/man1.0.2/apps/s_client.html
