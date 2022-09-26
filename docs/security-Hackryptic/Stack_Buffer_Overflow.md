
# Table of Contents

1.  [Stack-Based Buffer Overflow](#org5f5f579)
2.  [Memory Sections](#orgdda3ebb)
3.  [Prevention](#org0a8ba61)
4.  [ROP(Return Oriented Programming)](#org573469f)
5.  [CPU General Purpose Register](#org6212f4c)
6.  [Call 명령어](#orgd88602e)
7.  [Endianness](#org5ac5669)
8.  [Shellcode](#orgd388966)

\#-**- mode: org -**-

본 포스트는 HackTheBox Academy의 "STACK-BASED BUFFER OVERFLOWS ON LINUX X86" 모듈을 개인적으로 공부하며 정리한 내용이다.


<a id="org5f5f579"></a>

# Stack-Based Buffer Overflow

-   버퍼로 입력되는 데이터의 크기가 버퍼에 할당된 크기 이상을 침범할 때 발생한다.
-   공격자가 임의의 코드를 작성하거나 return address를 조작하는 등의 방식으로 악용할 수 있다.


<a id="orgdda3ebb"></a>

# Memory Sections

-   .text: 실제 명령어 코드들을 포함하는 영역이다. 통상적인 경우, 보안상의 이유로 쓰기 권한은 비활성화한다.
-   .data: global 및 static 변수들이 저장된 영역이다.
-   .bss: 선언은 되었으나 값이 지정되지 않은 static 변수들이 저장된 영역이다. (.data 섹션의 일부로 포함되기도 한다.)
-   Heap: (malloc 등)동적으로 메모리에 할당되고 해제되는 메모리 공간. (.bss 섹션의 끝으로부터 시작하여 증가하는 방향으로 할당된다)
-   Stack: LIFO 구조로 구성되며 return address, 지역 변수, 파라미터 등이 스택 프레임 구조로 할당되는 공간이다. (경우에 따라서는 스택 포인터 값 역시 저장된다.)


<a id="org0a8ba61"></a>

# Prevention

-   DEP(Data Execution Prevention): 메모리 공간에 read-only 권한만을 지정함으로써, Stack 공간에서의 코드 실행을 방지하는 방법이다.
-   ASLR(Address Space Layout Randomization): 메모리에 할당되는 바이너라와 동적모듈(dll등)의 base address를 실행시마다 randomization 한다.
-   Canaries: 버퍼와 return address 주소 사이에 일정한 값을 두고, 해당 값이 변경되었는지 체크하는 방법이다.


<a id="org573469f"></a>

# ROP(Return Oriented Programming)

-   프로그램에(혹은 라이브러리에) 포함된 기능들을 이용하기 위한 목적으로 return address를 조작함으로써 프로그램의 흐름을 바꾸는 행위이다.


<a id="org6212f4c"></a>

# CPU General Purpose Register

-   64비트 레지스터로써 다룰시에는 R이 붙는다 (RAX, RBX, RCX &#x2026;)

-   이하의 4가지 레지스터들은 Data regsister라 불리며 아래에서 설명하는 용도뿐 아니라 계산값들을 일시적으로 저장하는 등의 용도로 범용적으로 사용된다.  
    -   EAX: 산술 연산의 accumulator로써 사용된다.. (Function의 return value를 저장하는 용도로도 종종 사용된다.)
    -   EBX: memory addressing을 위한 base address를 저장하는 데 사용된다.
    -   ECX: Loop에서 conter 역할로 주로 사용된다.
    -   EDX: Multiply나 Division시에 큰 값을 다루기 위한 용도로 간혹 사용되며, I/O 과정에서 입출력장치와 데이터를 주고 받는데도 사용된다고 한다.

-   다음 3가지 레지스터들은 Pointer Register로서 대부분의 경우 아래에서 설명하는 용도로만 사용된다.
    -   EIP: 다음에 실행되는 명령어의 주소를 가지고 있다.
    -   ESP: 스택의 가장 위쪽 (마지막으로 데이터를 push한 address)를 가리키는 주소를 담고 있다.
    -   EBP: 현재 스택 프레임의 기준이 되는 base 부분을 가리키는 주소를 담고 있다.

-   다음 2가지 index register들은 주로 string 관련 작업(혹은 1바이트씩 처리하는 기타 작업들)을 처리하는 데 사용된다.
    -   ESI: source index 주소를 가리키는 데 사용된다.
    -   EDI: destination index 주소를 가리키는 데 사용된다.

-   Stack Frame: Base Pointer와 Stack Pointer로 정의되는 공간으로써 스택 프레임에 대응되는 함수의 데이터(지역변수 등)들이 저장된다.

-   대부분의 함수의 시작과 끝부분에는 새로운 스택 프레임을 만들고 정리하는 부분이 있다.
    -   prologue:
        
            push ebp
            mov ebp, esp
    
    -   epliogue:
        
            mov esp, ebp
            pop ebp ; 이 부분까지는 leave로 대체할 수 있음
            ret


<a id="orgd88602e"></a>

# Call 명령어

-   call 명령어가 수행될 시에는
    1.  우선 현재 EIP값(원래 다음 명령어를 가리키던 값)을 스택에 저장하고
    2.  EIP에 Function이 시작하는 주소를 새로 대입한다.


<a id="org5ac5669"></a>

# Endianness

-   데이터가 메모리 공간에 저장될 때 각 바이트가 어떠한 순서로 저장되는 지 정의한다.
-   Big-endian: 데이터를 원래 순서대로 저장 (0x12345678: 0x12, 0x34, 0x56, 0x78)
-   Little-endian: 데이터를 역순의 바이트 순서로 저장 (0x12345678: 0x78, 0x56, 0x34, 0x12)


<a id="orgd388966"></a>

# Shellcode

-   공격대상에서 쉘을 실행하기 위해 사용하는 코드.
-   쉘 코드를 직접 작성할 수도 있지만 적절한 툴들을 사용하면(msfvenom이나 파이썬의 pwntools 모듈 등) 간편하게 생성할 수 있다.
    -   다만 쉘 코드가 내부적으로 어떻게 동작하는 지에 대한 개념은 알고 있는 것이 좋다.

-쉘 코드 작성시 CPU 아키텍처와 대상 플랫폼 그리고 쉘코드에서 제외되어야 하는 문자들(null 등)에 유의하여 작성해야 한다.

