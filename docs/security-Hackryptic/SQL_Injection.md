
# Table of Contents

1.  [Definition](#org764226b)
2.  [Union based injection](#org9e89097)
3.  [FingerPrinting](#org5294547)
4.  [INFORMATION\_SCHEMA](#org15a6d2a)
5.  [Users and Previlages](#org38545b2)
6.  [sql injection 대응 방법](#org1d9c247)

\#-**- mode: org -**-

본 포스트는 hackthebox에서 제공하는 sql injection 모듈을 기반으로 공부하며 노트한 내용입니다.
예전에 sql injection 관련하여 배웠던 내용들을 복습하고 또한 새로운 지식들을 습득하는 과정에서 기록한 내용을 글로 적었습니다.


<a id="org764226b"></a>

# Definition

-사용자의 입력을 통해 (개발자가 의도하지 않은) 임의의 SQL 구문을 실행시킬 수 있다.
-예) (php의 경우)
   "select \* from user\underlist where username='$username';"
   &#x2013;> select \* from user\underlist where username='' or 1=1;#'"


<a id="org9e89097"></a>

# Union based injection

-   Union을 사용한 injection으로 특정 테이블의 컬럼들을 확인할 수 있다.
-   select &#x2026;. union select &#x2026;.
-   테이블 Column 수가 같을 때만 사용이 가능하다.
-   Column 개수가 다를 경우에는 junk 데이터로 개수를 맞춰준다.
-   order by 를 사용하여 (error 출력될 때까지 반복) column의 개수를 확인 가능하다.
    -   예) order by 1;


<a id="org5294547"></a>

# FingerPrinting

-   DB의 정보에 대한 조사 방법 (DBMS의 종류 등)
-   예) select @@version &#x2013;> mysql/mariaDB의 경우에만 출력


<a id="org15a6d2a"></a>

# INFORMATION\_SCHEMA

-   데이터베이스 및 테이블에 대한 메타데이터들을 포함
-   예) select \* from databaseA.tableA
-   INFORMATION_SCHEMA.SCHEMATA: DBMS의 모든 데이터베이스에 대한 정보를 가진 테이블
-   INFORMATION_SCHEMA.TABLES: DMBS의 모든 테이블에 대한 정보를 가진 테이블
-   INFORMATION_SCHEMA.COLUMNS: DMBS의 모든 컬럼에 대한 정보르 가진 테이블


<a id="org38545b2"></a>

# Users and Previlages

-   현재 DBMS에 로그인 된 유저 확인: SELECT USER()
-   유저 super_privilage 권한 확인: SELECT superunderpriv FROM mysql.user
-   infomationunderschema를 이용하여 유저의 (각 명령의 대한) 권한 확인: SELECT grantee, privilegeundertype FROM information_schema.user_privileges
-   파일 읽기: LOAD_FILE();
-   secure_fileunderpriv 변수: 어느 디렉터리에서 read/write를 할 수 있는지에 대한 범위를 지정
    -   empty value일 경우에는 모든 디렉터리에서 가능, NULL일 경우 어디에서도 불가능함을 의미
    -   MariaDB의 경우 empty, mysql의 경우는 /var/lib/mysql-files 값이 default
    -   previlage 확인:
        SELECT variable_name, variable_value FROM information_schema.global_variables where  variable_name="secure_file_priv

-   파일 쓰기: SELECT 'kkkkk' INTO OUTFILE '/home/user/kkkkk.txt';


<a id="org1d9c247"></a>

# sql injection 대응 방법

-   Input Sanitization: mysqli_real_escape_string 함수를 사용하여 특정 문자((ASCII 0), \n, \r, \\, ', ", and Control-Z.)를 escape한다.
    -   <https://www.php.net/manual/en/mysqli.real-escape-string.php> 참고

-   Input validation: 정규 표현식을 매칭시키는 preg_match 등의 함수를 사용하여

사용 가능한 문자를 특정 범위로 제한. (알파벳, 숫자 등)

-   User previlage 조정: 유저가 서비스를 사용하여 데이터에 접근하는데 사용하는 계정은 최소한의 권한만 부여한다. (특정 sql 명령으로 특정 테이블에만 접근 가능하도록)

-   웹 방화벽 단에서 특정 접근들을 차단한다.

-   파라미터화 된 쿼리를 사용한다.
    -   주로 사용하는 함수들은 다음과 같다.
    -   mysqli_prepare 함수: Prepares an SQL statement for execution
    -   mysqli_stmt_bind_param 함수: Binds variables to a prepared statement as parameters
    -   mysqli_stmt_execute 함수: Executes a prepared statement
    -   mysqli_stmt_get_result 함수: Gets a result set from a prepared statement as a mysqli_result object
    -   mysqli_fetch_array 함수: Fetch a result row as an associative array, a numeric array, or both
    -   mysqli_stmt_close 함수: Closes a prepared statement

