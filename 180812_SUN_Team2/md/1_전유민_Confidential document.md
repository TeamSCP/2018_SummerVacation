## XCZ.KR Prob 27

![image](https://user-images.githubusercontent.com/40850499/44000414-5a559366-9e5a-11e8-9a57-2fb1b7866c21.png) 

### 문제 개요

XCZ.KR 27번 문제를 보시면 Title에 XCZ Company Hacking Incident라고 적혀있고 Description에는 하나의 파일(디스크)을 주고 이 파일 안에서 증거가 될 수 있는 FORENSER가 외부로 유출시켰던 기밀문서를 찾아서 키 값을 구해내는 문제입니다.

### Prob27 File

문제에서 제시한 Prob27.7z파일을 다운받고 압축을 풀어주면 Prob27이라는 파일을 하나 줍니다.

![image](https://user-images.githubusercontent.com/40850499/44000419-784b8204-9e5a-11e8-8427-0134cb37139b.png) 

Prob27파일 안에 XCZ파일이 있고 그 안에는 윈도우 운영체제의 폴더들이었습니다.

이렇게 보면서 분석하기엔 아무것도 나오지 않아 FTK Imager 프로그램을 활용하여 분석해보았습니다.

### FTK Imager

FTK Imager는 디지털 포렌식 분야에서 많이 쓰이는 프로그램으로 파일 분석이나 삭제한 파일을 복구할 때 유용하게 쓰이는 프로그램입니다.

다운로드 : http://accessdata.com/

![image](https://user-images.githubusercontent.com/40850499/44000435-9b8380dc-9e5a-11e8-8b59-ba2f13c7ce88.png) 

FTK Imager로 파일을 열어봤을 때와 그냥 바탕화면에서 파일을 열어볼 때와 매우 다르다는걸 발견하실 수 있습니다. FTK Imager로 열어보면 더 자세하게 어떤 파일이 있는지 분석하실 수 있습니다. 

![image](https://user-images.githubusercontent.com/40850499/44000437-ba01485a-9e5a-11e8-882a-a3d3eb1864ed.png) 

많은 폴더들과 파일들을 계속 분석해보다가 이상한 파일을 하나 발견했습니다. Hex값 부분을 보시면 다른 파일들은 아무 내용도 존재하지 않았지만, 이 파일만 유독 내용이 많이 적혀있었고, 문제에 있었던 FORENSER라는 이름이 보입니다. 그리고 이메일 같이 보이는 FORENSER@xcz.kr과 Boss@xcz.kr이라고 문제에서 보였던 이름으로 된 이메일도 같이 보입니다.

 ![image](https://user-images.githubusercontent.com/40850499/44000442-d1389942-9e5a-11e8-91ca-dd67f7fffb57.png) 

문제의 파일로 의심되어 FTK Imager에서 Export Files를 눌러 이 파일을 추출하였습니다.

### DBX파일이란?

DBX는 Outlook Express에 의해 생성 된 폴더에 주어진 확장입니다. 그것은 특정 사서함에 대한 전자 메일 메시지를 포함하고 문서 및 설정 디렉토리에 저장됩니다. 그것은 백업 전자 메일 메시지에 위해 다른 폴더로 복사 할 수 있습니다.  따라서 이 문제는 메일 메시지와 관련되어 있고 Hex값에서 FORENSER@xcz.kr과 Boss@xcz.kr이 나온것도 메일에 관한 문제라는 걸 알 수 있었습니다.

### Mail Viewer

메일 뷰어 프로그램은 이메일 파일을 열어 볼 수 있는 프로그램입니다. 정확히 말하면 이메일 확장자(IDX,MBX,DBX)파일을 열어 확인할 수 있는 프로그램입니다.

다운로드 : http://www.snapfiles.com/get/dbxviewer.html

![image](https://user-images.githubusercontent.com/40850499/44000447-e685133e-9e5a-11e8-9824-3a4887e9b2c0.png) 

메일 뷰어 프로그램을 실행시키면 위 화면처럼 뜨고 이 문제에서 알아볼 파일은 아까 파일을 분석할 때 Outlook Express 파일안에 있었으므로 Outlook Express message database를 체크하고 불러오기를 통해 파일을 찾아주고 OK를 누릅니다.

![image](https://user-images.githubusercontent.com/40850499/44000454-fbc69420-9e5a-11e8-8767-5f96955346bc.png) 

아까 Hex값 부분에서 봤던 FORENSER@xcz.kr과 BOSS@xcz.kr이 같이 나오고 FORENSER가 BOSS에게 보낸 파일이라는걸 알 수 있습니다. 그리고 Attachments에 confidential이라는 파일이 있는데, 열어보면 어떤 내용인지 확인할 수 있습니다.

### CLEAR

![image](https://user-images.githubusercontent.com/40850499/44000463-12411d60-9e5b-11e8-9760-bbd73a6a759c.png) 

워드 형식으로 파일이 열리고 기밀문서의 내용을 볼 수 있습니다.

![image](https://user-images.githubusercontent.com/40850499/44000467-2509e968-9e5b-11e8-84d9-b3a4c5f91a50.png) 

파일을 다 읽어봐도 뭐가 키 값인지 몰라서 계속 찾아보다가 나중에 드래그한 부분을 색깔을 바꾸면 키 값이 나타나게 되는걸 알게 되었습니다. (드래그하지 않았을 때는 흰색 글씨로 되어 있어서 바탕과 색깔이 같아 찾을 수 없음.)

문제 클리어~
