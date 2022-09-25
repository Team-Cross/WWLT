
# Table of Contents

1.  [Stack-Based Buffer Overflow](#orga09dc41)
2.  [Memory Sections](#orgbf22b60)
3.  [Prevention](#orge9962eb)
4.  [ROP(Return Oriented Programming)](#orgdf579ee)

\#-**- mode: org -**-

본 포스트는 HackTheBox Academy의 "STACK-BASED BUFFER OVERFLOWS ON LINUX X86" 모듈을 개인적으로 공부하며 정리한 내용이다.


<a id="orga09dc41"></a>

# Stack-Based Buffer Overflow

-   버퍼로 입력되는 데이터의 크기가 버퍼에 할당된 크기 이상을 침범할 때 발생한다.
-   공격자가 임의의 코드를 작성하거나 return address를 조작하는 등의 방식으로 악용할 수 있다.


<a id="orgbf22b60"></a>

# Memory Sections

-   .text: 실제 명령어 코드들을 포함하는 영역이다. 통상적인 경우, 보안상의 이유로 쓰기 권한은 비활성화한다.
-   .data: global 및 static 변수들이 저장된 영역이다.
-   .bss: 선언은 되었으나 값이 지정되지 않은 static 변수들이 저장된 영역이다. (.data 섹션의 일부로 포함되기도 한다.)
-   Heap: (malloc 등)동적으로 메모리에 할당되고 해제되는 메모리 공간. (.bss 섹션의 끝으로부터 시작하여 증가하는 방향으로 할당된다)
-   Stack: LIFO 구조로 구성되며 return address, 지역 변수, 파라미터 등이 스택 프레임 구조로 할당되는 공간이다. (경우에 따라서는 스택 포인터 값 역시 저장된다.)


<a id="orge9962eb"></a>

# Prevention

-   DEP(Data Execution Prevention): 메모리 공간에 read-only 권한만을 지정함으로써, Stack 공간에서의 코드 실행을 방지하는 방법이다.
-   ASLR(Address Space Layout Randomization): 메모리에 할당되는 바이너라와 동적모듈(dll등)의 base address를 실행시마다 randomization 한다.


<a id="orgdf579ee"></a>

# ROP(Return Oriented Programming)

-   프로그램에(혹은 라이브러리에) 포함된 기능들을 이용하기 위한 목적으로 return address를 조작함으로써 프로그램의 흐름을 바꾸는 행위이다.

