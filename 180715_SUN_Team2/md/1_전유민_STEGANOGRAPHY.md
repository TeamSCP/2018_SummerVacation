## XCZ.KR Prob 12

https://user-images.githubusercontent.com/40850499/43039906-0e86c66a-8d72-11e8-9a0a-6e275c99e0c1.png

### 문제 개요

-xcz.kr 12번째 문제를 들어가보시면 Title에는 Steganography라고 적혀있고 Description에는 힌트 3개와 Download파일을 받으실 수 있습니다.

### Steganography

Steganography는 stegano(감추어져있다)와 graphos(쓰다)의 합성어로 steganography라는 단어가 만들어졌습니다. 정확한 의미로는 사진, 음악, 동영상 등의 일반적인 파일 안에 정보(파일)를 숨기는 기술입니다. 대표적인 사례로는 9.11 테러 당시 오사마 빈 라덴이 모나리자 그림에 비행기 도면을 숨긴 사례가 있습니다.

### Description

Description에는 힌트 3가지와 다운로드 파일 하나가 주어집니다. 첫 번째 힌트를 해석하면 해독 키 값=쓰는 툴 이름이고, 두 번째 힌트는 툴 이름은 OpenPuff입니다. 스테가노그래피툴은 그렇게 많지가 않아서 O와 P가 들어가는 툴은 OpenPuff밖에 없기 때문입니다. 세 번째 힌트는 오직 OpenPuff만 사용하라는 힌트입니다.

-http://xcz.kr/START/prob/prob_files/WindScene.mp3

-파일을 다운받으면 WindScene이라는 이름으로 된 mp3파일이 하나 주어집니다.

### OpenPuff

https://user-images.githubusercontent.com/40850499/43039908-271d0d92-8d72-11e8-85e9-15abd2821889.png

-OpenPuff를 실행시키면 맨 처음 실행 화면입니다.

-Steganography에서 숨긴 파일을 분석해야 하기 때문에 Unhide를 눌러줍니다.

https://user-images.githubusercontent.com/40850499/43039914-3f9f3b2e-8d72-11e8-88b6-3f618ea812f5.png

-Unhide를 눌러주면 여러가지 복잡해보이는 화면이 보입니다.

-첫 번째 힌트를 보시면 해독 키 값=쓰는 툴 이름이라고 하였으니 Cryptography에 OpenPuff를 입력해줍니다.

-주어진 해독 키 값은 1개이므로 Enable(B)와 (C)에 체크를 풀어줍니다.

-마지막으로 Add Carriers를 눌러 다운로드 받은 파일(WindScene)을 찾아줍니다.

https://user-images.githubusercontent.com/40850499/43039918-52bebdc4-8d72-11e8-88dc-9e99d885f409.png

-위의 사항들을 모두 설정해주시면 위의 화면으로 바뀌게되고 마지막으로 우측 하단에 있는 Unhide!를 눌러줍니다.

https://user-images.githubusercontent.com/40850499/43039922-62351078-8d72-11e8-8375-95d314109531.png

-Unhide를 눌러주시면 숨겨져 있던 파일(이름이 key로 되어있음)이 나오게 됩니다.

https://user-images.githubusercontent.com/40850499/43039929-7a9b616c-8d72-11e8-8d0d-5dd9ae03b80a.png

-이름은 역시 key라고 되어있으며 열기를 눌러주시면 메모장 파일이라는걸 확인하실 수 있습니다.

https://user-images.githubusercontent.com/40850499/43039935-8b896622-8d72-11e8-95f9-8f3e22053551.png

-열어주시면 메모장 파일안에 key값이 나오게 됩니다. (검은색 부분)

