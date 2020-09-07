---
bg: "Projects.jpg"
layout: post
title:  "Atmega128를 활용한 간단한 게임만들기 프로젝트"
crawlertitle: "Atmega 128 프로젝트"
summary: "Atmega 128 프로젝트"
date: 2020-04-16 16:30:00 +0900
category: Projects
author: Err0rCode7
---

이번 포스팅은 작년 MicroProcessor 수업에서 Atmega128을 사용하여 만들었던 미니 게임에 대해서 리뷰해보겠습니다.<br>

---
### Atmega128을 이용한 미니게임 Project
---

우선 Atmega128은 ATMEL사가 개발한 8비트 AVR마이크로 컨트롤러의 megaAVR 패밀리 계열중 하나의 프로세서입니다. 이 프로세서는 64핀으로 구성되었으며, TQFP형 패키지의 저 전력 8비트 CMOS 마이크로 컨트롤러입니다. C언어와 라이브러리를 이용해서 명령어를 쉽게 이용할 수 있으며 원하는 포트의 핀에 LCD, 스위치, 7-segments, MOTOR 그리고 LED를 연결하고 코드상의 레지스터를 이용하여 사용할 수 있습니다. <br>

이러한 Atmega128을 이용해서 미니 게임을 만들어 봤었습니다. RISC 구조로 명령어도 간단하고 사용하는데 많은 어려움이 없었던 것으로 기억이 납니다. <br>


전원을 Atmega128와 연결하고 Atmega128과 컴퓨터의 포트를 연결해서 컴퓨터에서 작성한 C언어를 이용하여 원하는 명령어를 사용하여 실습과 프로젝트를 진행했었습니다.<br>

수업 실습시간을 이용해서 Atmega128 키트에 있던 모든 기능들을 다 사용해보았었습니다.<br>

그리고 마지막 term project로 각 조마다 아이디어 구상하여 창의적인 아이템을 만들어보는 프로젝트 기회를 가졌었습니다. <br>

이 프로젝트에서 저희 조는 미니게임을 만들었었는데요, LCD 화면에 캐릭터와 장애물을 표현하고 계속해서 우측에서 좌측으로 장애물이 움직이면서 스위치를 이용하여 캐릭터를 점프시켜 장애물을 피하는 게임이었습니다.

![atmega128-1](https://user-images.githubusercontent.com/48249549/79439126-6b393f00-800f-11ea-8389-66fb91c23dcd.png)

![atmega128-2](https://user-images.githubusercontent.com/48249549/79439197-7ab88800-800f-11ea-9ed5-750404534a9f.png)

위에 사진은 실제 프로젝트를 진행하고 찍은 사진입니다. 포트를 많이 사용하는 것에 추가 점수가 있어서 최대한 많은 포트를 사용하다보니
굉장히 지저분 했었네요.. <br>

두번째 사진은 실제 미니게임 프로젝트가 어떤식으로 표현됐었는지 보이네요.

전체적인 게임의 시스템을 설명해드리면, 캐릭터와 장애물이 LCD에 표시가 되고 스위치를 이용하여 캐릭터는 점프를 할 수 있고 장애물을 피할 수 있습니다. 7-segments를 이용하여 사용자는 자신이 얻은 점수를 볼 수 있습니다. 장애물을 피하면 7-segments에 +점수가, 못피하면 -점수가 덧셈이 됩니다. 그리고 Motor와 led를 이용하여 장애물을 피할때와 못피할때에 이펙트를 주었습니다.

[My Atmega128 Project Code](https://github.com/Err0rCode7/Mircoprocessor/blob/master/Project.c)
<br>

위에 저의 깃허브 링크인 이 링크에 들어가시면 당시에 실행했었던 코드를 자세히 확인 하실 수 있습니다.<br>
<br>

---
### Project Code Review
---

```c
#include <mega128.h>
#include <delay.h>
#include <stdio.h>
#include <stdlib.h>
#include <lcd.h>

#define mapsize 16
#define Q0 PORTC.4
#define Q1 PORTC.5
#define Q2 PORTC.6
#define Q3 PORTC.7
#define FND PORTF
#define DCM_IN1 PORTC.2
#define DCM_IN2 PORTC.3
#define STEP_MOTOR PORTE
#define STEP_LED PORTD

#asm
    .equ __lcd_port=0x1B
#endasm

```

처음부터 살펴보면 `<mega128.h>` 와 `<lcd.h>` 라이브러리를 통하여 atmega128 명령어를 include하여 명령어를 사용했었습니다.<br>

그 이후에 필요한 기능의 포트들을 쉬운 변수로 정의해서 사용했었네요.<br>
또한 LCD 포트는 따로 어셈블리어를 적어서 사용한것을 볼 수 있습니다.

```c
int i=0, Score=0;
char obs; //장애물
int fnd1000,fnd100,fnd10,fnd1;
int remainder;
unsigned fnd[17]={0xc0,0xf9,0xa4,0xb0,0x99,0x92,0x82,0xd8,0x80,0x98,0x88,0x83,0xc6,0xa1,0x86,0x8e,0x7f,};

void init(void)
{
    // PORTA LCD
    PORTA=0x00;
    DDRA=0xFF;
    // PORTB LED
    PORTB=0x00;
    DDRB=0xff;

    // PORTC 0,1 : 스위치 2, 3 : DC모터 4,5,6,7 : 7-segment Q
    PORTC=0x00;
    DDRC=0xF0;
    PINC=0x00;
    // PORTD STEP_MOTOR LED
    PORTD=0x00;
    DDRD=0xFF;
    // PORTF STEP_MOTOR Data
    PORTF=0X00;
    DDRF=0XFF;
    // PORTE 7-segment Data
    PORTE=0x00;
    DDRE=0xFF;

    FND = fnd[0];
    Q0 = 1;
    Q1 = 1;
    Q2 = 1;
    Q3 = 1;
}
```

계속해서 필요한 전역변수를 선언해주었고 여기서 `usigned fnd[]` 는 7-segments를 쉽게 사용할 수 있도록 선언한 것을 볼 수 있습니다.<br>

그리고 최대한 기능을 많은 기능(LCD, Motor 등..)을 넣기위해서 포트를 하나하나 핀별로 사용했었습니다. 그래서 초기화 함수를 따로 만들어서 포트의 초기값과 변수의 초기값을 각각 선언한것을 볼 수 있습니다.<br>

이제 메인함수를 들어가기전에 메인 뒤에 선언해놓은 함수들을 먼저 리뷰해보겠습니다.

```c
void display(int); // 7-segment
void DC_M_CW();  //  DC MOTOR 작동
void DC_M_STOP(); //  DC STOP
void Addmap(char [] , char); // 맵 업데이트
void STEP_M_CW();   // STEP MOTOR
void STEP_M_STOP();   // STEP STOP
void STEP_M_CCW();

void Addmap(char cur[], char randC)
{
    for (i = 0; i<mapsize - 2; i++)
    {
        cur[i] = cur[i + 1];

    }
    cur[mapsize-2] = randC;
}
void display(int Scor){
    Score += 50;
    fnd1000 = Scor/1000;
    remainder = Scor%1000;
    fnd100 = remainder/100;
    remainder = Scor%100;
    fnd10 = remainder/10;
    remainder = Scor%10;
    fnd1 = remainder;
    for (i=0;i<50;i++)
    {
        if (Scor >= 1000)
        {
            FND = fnd[fnd1000];
            Q3 = 0;
            Q2 = 1;
            Q1 = 1;
            Q0 = 1;
            delay_ms(10);
        }
        if (Scor >= 100)
        {
            FND = fnd[fnd100];
            Q3 = 1;
            Q2 = 0;
            Q1 = 1;
            Q0 = 1;
            delay_ms(10);
        }
        FND = fnd[fnd10];
        Q3 = 1;
        Q2 = 1;
        Q1 = 0;
        Q0 = 1;
        delay_ms(10);
        FND = fnd[fnd1];
        Q3 = 1;
        Q2 = 1;
        Q1 = 1;
        Q0 = 0;
        delay_ms(10);
    }
    Q0 = 1;
    Q1 = 1;
    Q2 = 1;
    Q3 = 1;
}
void DC_M_CW()
{
    DCM_IN1 = 1;
    DCM_IN2 = 0;
    delay_ms(60);
}

void DC_M_STOP()
{
    DCM_IN1 = 0;
    DCM_IN2 = 0;
}

void STEP_M_CW()
{

    STEP_MOTOR = 0x01;
    STEP_LED = 0x01;
    delay_ms(100);

    STEP_MOTOR = 0x02;
    STEP_LED = 0x02;
    delay_ms(100);

    STEP_MOTOR = 0x04;
    STEP_LED = 0x04;
    delay_ms(100);

    STEP_MOTOR = 0x08;
    STEP_LED = 0x08;
    delay_ms(100);

}
void STEP_M_CCW()
{

    STEP_MOTOR = 0x08;
    STEP_LED = 0x08;
    delay_ms(100);

    STEP_MOTOR = 0x04;
    STEP_LED = 0x04;
    delay_ms(100);

    STEP_MOTOR = 0x02;
    STEP_LED = 0x02;
    delay_ms(100);

    STEP_MOTOR = 0x01;
    STEP_LED = 0x01;
    delay_ms(100);

}
void STEP_M_STOP()
{
    STEP_MOTOR = 0x00;
    STEP_LED = 0x00;
}
```

```c
void display(int)
```
`display()` 함수는 미니게임에서 7-segments를 이용하여 사용자의 점수를 나타내는 기능입니다. (점프를 하면 점수를 잃고 장애물에 닿으면 점수를 잃는 기능을 넣었었습니다.)<br>

```c
void DC_M_CW();  //  DC MOTOR 작동
void DC_M_STOP(); //  DC STOP
```
DC Motor를 사용할 때는 적당한 delay를 주어서 속도를 조절해서 사용해야했었던 것이 기억이 납니다. 그래서 `DC_M_CW()` 에 딜레이를 넣어주어 Motor를 작동하였었고 그리고 `DC_M_STOP`을 이용해서 Motor를 멈추게 하였습니다. ( 점수를 얻으면 이펙트로 Motor를 작동하도록 하였습니다. )

```c
void Addmap(char [] , char); // 맵 업데이트
void STEP_M_CW();   // STEP MOTOR
void STEP_M_STOP();   // STEP STOP
void STEP_M_CCW();
```

나머지 함수를 살펴봅시다. `AddMap()` 함수는 함수명 그대로 맵을 만드는 명령어입니다. <br>

`STEP_`이 앞에 붙은 함수들은 STEP Motor를 작동하거나 정지시키는 함수들 입니다. STEP 모토를 이용하여 성공도를 표현했었습니다.<br>
`STEP_M_CW()`는 시계방향, `STEP_M_CW()`는 반시계방향으로 회전하며 성공도를 표현합니다. ( 여기서 성공도란 자신이 얼마나 장애물을 잘피하고 못피하고를 판단해주는 것을 얘기합니다. )<br>
`STEP_M_STOP()`은 DC Motor와 마찬가지로 정지기능입니다.<br>
( 일정 각도와 시간만큼 회전하고 멈추어야하기 때문에 Motor들은 모두 정지 기능이 필요합니다. )<br>

```c
void main(){

    unsigned char pName[] = {0B11101111}; // 캐릭터
    /* 기본 맵 작성 */
    char map[100] ="__________________________";
    map[7]=0b11111111;
    map[11]=0b11111111;
    map[3]=0b11111111;

    init();

    lcd_init(16);
    lcd_gotoxy(0,1);
    lcd_puts(pName);

    lcd_gotoxy(1,1);
    lcd_puts(map);
    delay_ms(1000);

    while(1)
    {   i = rand()%10;    // 랜덤으로 장애물 생성
        if ( (i%2)==0)
        {
            obs = 0b11111111;
        }
        else if( (i%2)!=0 )
        {
            obs = '_';
        }
        // LCD 핸들링
        PORTB = 0b11111111;
        lcd_gotoxy(1,0);
        lcd_puts(" ");
        lcd_gotoxy(0,1);
        lcd_puts(pName);

        lcd_gotoxy(1,1);
        Addmap(map,obs);

        lcd_puts(map);
        delay_ms(1000);

        if (PINC.0 == 0)
        {
            lcd_gotoxy(0,1);
            lcd_puts(" ");
            lcd_gotoxy(1,0);
            lcd_puts(pName);
            if (map[0]==0b11111111)
            {
                PORTB = 0b00001111;
                display(Score);
                STEP_M_CW();
            }
            delay_ms(1000);
        }
        else
        {
            if (map[0]==0b11111111)
            {

                PORTB = 0b11110000;
                DC_M_CW();
                DC_M_STOP();
                STEP_M_CCW();
                 if(Score >=100)
                 Score -= 100;
            }
        }
    }
}
```

이제 마지막으로 Main 함수의 코드를 살펴봅시다.<br>

처음에 캐릭터와 맵을 미리 선언해주었습니다. 그 다음으로 전체적인 장애물을 포함한 맵이 왼쪽으로 한칸 움직입니다. <br>

그리고 이제 While문을 살펴보면 난수를 생성하고 난수가 짝수이면 장애물을 생성하여 맵에 넣는 방식으로 구현되어있는 것을 볼 수 있습니다. <br>
(사실 이 난수는 계속 일정한 값을 생성합니다. 현재 시간을 이용하여 난수를 생성하여야하는데 그것이 불가능해서 그렇습니다. 그렇기 때문에 게임을 다시 시작하면 계속해서 같은 맵을 체험할 수 밖에 없는 구조입니다.)<br>

LCD 명령어를 활용하여 한칸씩 계속 움직이는 것을 표현하는 하였고 맵에 계속해서 추가해서 맵을 저장하였습니다. <br>



그 다음으로 if 문을 보면 PORTC를 이용하여 스위치의 값 ( 캐릭터가 점프를 뛰었는 가?)과 맵을 비교하여 장애물과 캐릭터가 부딪히게 되는 지 판단하여 점수를 올리거나 내렸고 DC Motor와 STEP Motor를 핸들링 했습니다 . 그리고 계속해서 while문을 통해서 반복되고 게임은 계속 진행됩니다. <br>

이렇게 Atmega128과 함께한 장애물을 피하는 미니게임 Review를 해봤습니다.

다시 살펴보니 허접하지만 나름 재미있게 구현했던것 같습니다.<br>
이 프로젝트를 하고 있었을 당시에 traceroute와 thread에 대해서 배우고 있었던 것이 생각나네요. thread를 못사용하여 진행하는 모습이 뚝뚝 끊기거나 연속적이지 못한게 아쉬웠었습니다. <br>

atmega128를 사용하려는 사람들에게 제 프로젝트가 조금은 도움이 되었었으면 좋겠습니다.<br>

이상으로 Atmega128 Review는 여기까지 하겠습니다.

