## XCZ.KR Prob 36

![image](https://user-images.githubusercontent.com/40850499/43685782-57499b24-98f4-11e8-842a-b841e3430795.png) 

### 문제 개요

XCZ.KR 36번 문제를 보시면 Title에는 File Deleted(파일이 지워졌다)라고 적혀있고 Description에는 피시방에서 아동 청소년 보호법에 위배되는 파일을 소지한 기록이 발견되어 형식에 맞춰서 증거를 수집하라는 문제입니다. 

### Prob File

밑에 있는 Prob File 문제파일을 다운받고 압축을 풀어주면 file_deleted라는 파일을 하나 줍니다.

![image](https://user-images.githubusercontent.com/40850499/43685794-a46cc3a4-98f4-11e8-9474-8907cbf02b7b.png) 

파일을 열고 unl 디렉터리를 보면 여러 하위 디렉터리들이 보입니다. 자세하게 살펴보기 위하여 FTK Imager 프로그램으로 파일 소지 기록의 흔적을 찾아 보았습니다.

### FTK Imager

FTK Imager는 디지털 포렌식 분야에서 많이 쓰이는 툴로 휴지통에서 삭제한 파일도 복원할 수 있는 프로그램 입니다.

다운로드 : http://accessdata.com/

![image](https://user-images.githubusercontent.com/40850499/43685797-b4ff2770-98f4-11e8-86c8-56570a8228c7.png) 

FTK Imager로 Prob File을 열고 계속 찾아본 결과, Recent라는 파일안에 s3c3r7.avi.lnk라는 수상한 이름으로된 파일이 하나 있습니다. 이 파일에 무언가 있을 것 같아 FTK에서 추출한 다음, 파일 분석 도구인 010Editor로 열어보았습니다.

### 010 Editor

수 많은 HEX값들을 알아볼 수 있는 프로그램 중 하나인 010 Editor는 각종 파일들의 포맷 구조 템플릿을 지원해주고 파일별로 헤더를 분석해주는 툴입니다.

다운로드 : http://www.sweetscape.com/010editor/

![image](https://user-images.githubusercontent.com/40850499/43685800-c97b7956-98f4-11e8-832c-7c78d01cc4b4.png) 

문제를 풀기 위하여 알아야하는 정보는 원본 경로, 만들어진 시간, 마지막 실행 된 시간, 쓰인 시간, 볼륨 시리얼 이렇게 5가지를 알아야 문제를 풀 수 있습니다. 계속 분석하다보면 5가지를 모두 찾을 수 있게 됩니다.

원본 경로  

![image](https://user-images.githubusercontent.com/40850499/43685804-d9409e84-98f4-11e8-88a1-be6fee6b0d31.png) 

만들어진 시간, 마지막 실행 된 시간, 쓰인 시간

![image](https://user-images.githubusercontent.com/40850499/43685809-e555fad4-98f4-11e8-9ceb-7807301155cf.png) 

볼륨 시리얼

![image](https://user-images.githubusercontent.com/40850499/43685811-f06db7fe-98f4-11e8-9343-169a76a0c449.png) 

![image](https://user-images.githubusercontent.com/40850499/43685814-fce92f36-98f4-11e8-8445-edb35a142988.png) 

정리해서 써보면 원본경로(String LocalBasePath[20]) = H:\study\s3c3r7.avi

만들어진 시간(FILETIME Creation Time) = 10/16/2013 04:52:10

마지막 실행 된 시간(FILETIME Access Time) = 10/16/2013 14:24:50

쓰인 시간(FILETIME Write Time) = 10/16/2013 10:37:47

볼륨 시리얼(DWORD DriveSerialNumber) = A9 02 A8 D0

### CLEAR

아까 문제에서 GMT +9라 했으니 만들어진 시간, 마지막 실행 된 시간, 쓰인 시간에서 시간을 모두 9시간을 더해줍니다. 그리고 시리얼 넘버는 리틀엔디안 방식으로 되어 있어서 뒤에서부터 읽어야된다고 합니다. 따라서 D0A8-02A9가 됩니다.

아까 준 예시처럼 정리해보면 H:\study\s3c3r7.avi_20131016135210_20131016232450_20131016193747_D0A8-02A9

이렇게 최종적으로 됩니다.

이 값을 아까 문제에 있었던 md5로 인코딩해주면 플래그값이 나오게 됩니다.

![image](https://user-images.githubusercontent.com/40850499/43685816-08eab6b0-98f5-11e8-9e81-fd93026fc260.png) 

인코딩하면 플래그값이 모두 lowercase(소문자)로 나와서 바꿀 필요가 없고 바로 플래그값을 입력해주면 된다.

문제 해결~
