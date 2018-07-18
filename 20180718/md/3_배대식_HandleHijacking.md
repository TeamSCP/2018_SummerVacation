# Handle Hijacking

> 해당 기술은 UC 포럼의 `harakrinox`의 글을 참조 하였습니다.

## 읽기 전..
  
  - 프로세스간 통신(IPC)중 Named Pipe를 사용합니다.
  - `공격 대상 프로세스가 핸들을 구해 올 수 없다`의 가정 하에 진행 됩니다.
  - `개념증명(PoC)`에 사용 되는 코드는 대상이 둘 다 실행 파일이지만, 파이프 서버쪽을 `하이재킹 대상` 혹은 `dll injection`의 대상이라고 생각 해주세요.
  - 유저 모드에서 사용 되는 우회 기법입니다.

## What is Hijacking

- 하이재킹에 대한 간단한 설명

## remind handle

- 전 문서에서도 핸들에 대해 언급했지만 다시 한 번더.

## 전체적인 이미지 개요도(이미지 삽입)

## 왜 다른 프로세스의 핸들을 사용하는게 AntiCheat에 유효?
- 오래 전(중학교~대1)에는 공격 대상이 타겟 프로세스

## IPC에 대한 설명
- File
- Anonymous Pipe
- Named Pipe (*)
- Socket
- Shared Memory
- Memory Mapped File
- Windows Message (*)

## IPC중 Named Pipe 설명

- 소켓 프로그래밍 해봤다면.. 나름 이해 가는듯?

## 1. 테스트 프로그램 작성(타겟 프로그램)
  - 간단히 Int, String 타입의 변수의 상태를 확인 할 수 있는 프로그램
## 2. 테스트 프로그램 작성(파이프 서버(하이재킹 대상))
  - 하이재킹 대상 ( CreateNamedPipe, ConnectNamedPipe, OpenProcess로 타겟 프로그램에 대한 핸들을 받아옴. 여기서는 lsass의 역할을 대신함 )
## 3. 테스트 프로그램 작성(파이프 클라이언트(명령))
  - 명령 ( 명령코드작성(서버에게 보냄), CreateFile로 열음 )
