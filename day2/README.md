# 2일차 데이터 엔지니어링 수집

> 데이터 처리 및 분석의 가장 처음 과정인 데이터 수집 도구를 이용한 데이터 적재를 실습합니다. 관계형 데이터베이스 수집을 위한 Apache Sqoop, 파일 데이터 수집을 위한 TreasureData Fluentd 를 이용해 실습합니다. 이번 장에서 사용하는 외부 오픈 포트는 22, 80, 5601, 8080, 9880, 50070 입니다

- 목차
  * [1. 테이블 수집 기본](#테이블-수집-기본)
    - [1-1. 최신버전 업데이트 테이블](#1-최신버전-업데이트-테이블)
    - [1-2. 테이블 임포트 실습](#2-테이블-임포트-실습)
    - [1-3. 테이블 익스포트 실습](#3-테이블-익스포트-실습)
  * [2. 파일 수집 기본](#파일-수집-기본)
    - [1-1. 최신버전 업데이트](#1-최신버전-업데이트)
    - [1-2. 플루언트디를 웹서버처럼 사용하기](#2-플루언트디를-웹서버처럼-사용하기)
    - [1-3. 수신된 로그를 로컬에 저장하는 예제](#3-수신된-로그를-로컬에-저장하는-예제)
    - [1-4. 서버에 남는 로그를 지속적으로 모니터링하기](#4-서버에-남는-로그를-지속적으로-모니터링하기)
<br>

## 테이블 수집 기본

## 1. 최신버전 업데이트 테이블

> 원격 터미널에 접속하여 관련 코드를 최신 버전으로 내려받고, 과거에 실행된 컨테이너가 없는지 확인하고 종료합니다

### 1-1. 최신 소스를 내려 받습니다
```bash
# terminal
cd /home/ubuntu/work/data-engineer-${course}-training
git pull
```
<br>

### 1-2. 현재 기동되어 있는 도커 컨테이너를 확인하고, 종료합니다

#### 1-2-1. 현재 기동된 컨테이너를 확인합니다
```bash
# terminal
docker ps -a
```
<br>

#### 1-2-2. 기동된 컨테이너가 있다면 강제 종료합니다
```bash
# terminal 
docker rm -f `docker ps -aq`
```
> 다시 `docker ps -a` 명령으로 결과가 없다면 모든 컨테이너가 종료되었다고 보시면 됩니다
<br>


### 1-3. 실습을 위한 이미지를 내려받고 컨테이너를 기동합니다
```bash
# terminal
cd /home/ubuntu/work/data-engineer-${course}-training/day2

docker-compose pull
docker-compose up -d
docker-compose ps
```

[목차로 돌아가기](#2일차-데이터-엔지니어링-수집)

<br>
<br>


## 2. 테이블 임포트 실습

### 2-1. 실습명령어 검증을 위한 ask 를 리뷰하고 실습합니다

> ask 명령어는 아래와 같이 전달받은 명령을 실행하는 스크립트입니다

```bash
#!/bin/bash
while true; do
    echo
    echo "$ $@"
    echo
    read -p "위 명령을 실행 하시겠습니까? [y/n] " yn
    case $yn in
        [Yy]* ) "$@"; break;;
        [Nn]* ) exit;;
        * ) echo "[y/n] 을 입력해 주세요.";;
    esac
done
```
<br>

#### 2-1-1. 스쿱 명령어 실습을 위해 컨테이너에 접속합니다
```bash
# terminal
docker-compose exec sqoop bash
```
<br>

* 간단한 출력 명령을 수행합니다
```bash
# docker
ask echo hello world
```
<details><summary> 정답확인</summary>

> "hello world" 가 출력되면 정상입니다

</details>
<br>


#### 2-1-2. 데이터베이스, 테이블 목록 조회

> 스쿱을 이용하여 별도의 데이터베이스 접속 클라이언트가 없어도 기본적인 명령어(DDL, DML)을 수행할 수 있습니다

* 데이터베이스 목록을 조회합니다
```bash
# docker
sqoop list-databases --connect jdbc:mysql://mysql:3306 --username sqoop --password sqoop
```
<br>

* 테이블 목록을 조회합니다
```bash
# docker
sqoop list-tables --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop
```
<br>

* 테이블 정보 및 조회를 합니다
```bash
# docker
sqoop eval --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop -e "DESCRIBE user_20201025"
ask sqoop eval --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop -e "SELECT * FROM user_20201025"
```

> 위와 같이 특정 데이터베이스에 계속 명령을 날리기에는 불편함이 있으므로 반복되는 명령어를 bash 쉘을 통해 만들어보면 편하게 사용할 수 있습니다

<details><summary>[실습] eval 명령을 쉽게 할 수 있는 간단한 bash 스크립트(예제: foo)를 만들어 보세요 </summary>

* 아래의 명령으로 터미널이 명령을 받을 준비를 하도록 하고
```bash
# docker
cat > foo
```
* 아래의 내용을 복사해서 붙여넣은 다음 <kbd><samp>Ctrl</samp>+<samp>C</samp></kbd> 명령으로 나오면 파일이 생성됩니다
```bash
#!/bin/bash
sqoop eval --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop -e "$@"
```
* 실행 가능한 파일로 만들기 위해서 모드를 변경합니다
  - 실행 시에 반드시 쌍따옴표("")로 묶어야 제대로 수행됩니다
```bash
# docker
chmod +x foo
foo "DESCRIBE user_20201025"
```

</details>
<br>


#### 2-1-3. 테이블 조인, DDL, DML 명령어 수행

> eval 명령어를 이용하면 Join, Create, Insert, Select 등 DDL, DML 명령을 수행할 수 있으며, *실제 테이블 수집 시에도 다수의 테이블 대신 Join 한 결과를 사용하는 경우 효과적인 경우*도 있습니다

* 고객 테이블과 매출 테이블을 조인하는 예제
```sql
# docker
ask sqoop eval --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
    -e "SELECT u.*, p.* FROM user_20201025 u JOIN purchase_20201025 p ON (u.u_id = p.p_uid) LIMIT 10"
```
<br>

* 테이블 생성 예제
```sql
# docker
ask sqoop eval --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
    -e "CREATE TABLE tbl_salary (id INT NOT NULL AUTO_INCREMENT, name VARCHAR(30), salary INT, PRIMARY KEY (id))"
```
<br>

* 데이터 입력 예제
```sql
# docker
ask sqoop eval --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
    -e "INSERT INTO tbl_salary (name, salary) VALUES ('suhyuk', 10000)"
```
<br>

* 데이터 조회 예제
```sql
# docker
ask sqoop eval --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
    -e "SELECT * FROM tbl_salary"
```
<br>


#### 2-1-4. 실습 테이블 생성 및 수집 실습

##### 실습용 테이블 정보를 활용하여 테이블을 생성하세요

* 데이터베이스: testdb
* 테이블 이름: student 

| 컬럼명 | 컬럼유형 | 데이터 예제 |
| - | - | - |
| no | INT | `AUTO_INCREMENT` |
| name | VARCHAR(50) | 박수혁 |
| email | VARCHAR(50) | suhyuk.park@gmail.com |
| age | INT | 30 | 나이 |
| gender | VARCHAR(10) | 남 |

<details><summary>[실습] 위에서 명시한 student 테이블을 sqoop eval 명령어를 통해 testdb 에 생성하세요 </summary>

```bash
# terminal
cd /home/ubuntu/work/data-engineer-basic-training/day2/
docker-compose exec mysql mysql -usqoop -psqoop
```
```sql
# mysql>
USE testdb;

CREATE TABLE student (
  no INT NOT NULL AUTO_INCREMENT
  , name VARCHAR(50)
  , email VARCHAR(50)
  , age INT
  , gender VARCHAR(10)
  , PRIMARY KEY (no)
);
```
```sql
DESC student;
```
* 아래와 같이 나오면 정답입니다
```text
+--------+-------------+------+-----+---------+----------------+
| Field  | Type        | Null | Key | Default | Extra          |
+--------+-------------+------+-----+---------+----------------+
| no     | INT(11)     | NO   | PRI | NULL    | AUTO_INCREMENT |
| name   | VARCHAR(50) | YES  |     | NULL    |                |
| email  | VARCHAR(50) | YES  |     | NULL    |                |
| age    | INT(11)     | YES  |     | NULL    |                |
| gender | VARCHAR(10) | YES  |     | NULL    |                |
+--------+-------------+------+-----+---------+----------------+
```

</details>
<br>


##### 실습용 데이터 정보를 활용하여 데이터를 입력하세요

* 고객 데이터 정보
```text
('권보안','Kwon.Boan@lgde.com',18,'여')
,('민의주','Min.Euiju@lgde.com',20,'여')
,('김혀시','Kim.Hyeosi@lgde.com',20,'남')
,('김유은','Kim.Yueun@lgde.com',38,'여')
,('박윤미','Park.Yoonmi@lgde.com',27,'여')
,('박예하','Park.Yeha@lgde.com',30,'남')
,('이병하','Lee.Byungha@lgde.com',21,'남')
,('김휘비','Kim.Hwibi@lgde.com',38,'남')
,('박재문','Park.Jaemoon@lgde.com',49,'남')
,('우소은','Woo.Soeun@lgde.com',30,'여')
```

<details><summary>[실습] sqoop eval 명령어를 통해 student 테이블에 입력하세요 </summary>

```sql
# mysql>
use testdb;

INSERT INTO student (name, email, age, gender) VALUES 
('권보안','Kwon.Boan@lgde.com',18,'여')
,('민의주','Min.Euiju@lgde.com',20,'여')
,('김혀시','Kim.Hyeosi@lgde.com',20,'남')
,('김유은','Kim.Yueun@lgde.com',38,'여')
,('박윤미','Park.Yoonmi@lgde.com',27,'여')
,('박예하','Park.Yeha@lgde.com',30,'남')
,('이병하','Lee.Byungha@lgde.com',21,'남')
,('김휘비','Kim.Hwibi@lgde.com',38,'남')
,('박재문','Park.Jaemoon@lgde.com',49,'남')
,('우소은','Woo.Soeun@lgde.com',30,'여');
```
```sql
SELECT * FROM student;
```
* 아래와 같이 나오면 정답입니다
```text
+----+-----------+-----------------------+------+--------+
| no | name      | email                 | age  | gender |
+----+-----------+-----------------------+------+--------+
|  1 | 권보안    | Kwon.Boan@lgde.com    |   18 | 여     |
|  2 | 민의주    | Min.Euiju@lgde.com    |   20 | 여     |
|  3 | 김혀시    | Kim.Hyeosi@lgde.com   |   20 | 남     |
|  4 | 김유은    | Kim.Yueun@lgde.com    |   38 | 여     |
|  5 | 박윤미    | Park.Yoonmi@lgde.com  |   27 | 여     |
|  6 | 박예하    | Park.Yeha@lgde.com    |   30 | 남     |
|  7 | 이병하    | Lee.Byungha@lgde.com  |   21 | 남     |
|  8 | 김휘비    | Kim.Hwibi@lgde.com    |   38 | 남     |
|  9 | 박재문    | Park.Jaemoon@lgde.com |   49 | 남     |
| 10 | 우소은    | Woo.Soeun@lgde.com    |   30 | 여     |
+----+-----------+-----------------------+------+--------+
10 rows in set (0.00 sec)
```

</details>
<br>



### 2-2. 로컬 프로세스로, 로컬 저장소에 텍스트 파일로 테이블 수집

> 스쿱은 분산 저장소와 분산 처리를 지원하지만, 로컬 리소스 (프로세스) 그리고 로컬 저장소 (디스크, SATA) 에도 저장하는 기능을 가지고 있습니다

* 컨테이너 로컬 디스크에 예제 테이블(student)을 수집합니다
  - <kbd>-jt local</kbd> : 로컬 프로세스로 (원격 분산처리가 아니라) 테이블을 수집
  - <kbd>-fs local</kbd> : 로컬 디스크에 (원격 부산저장소가 아니라) 테이블을 수집
  - <kbd>-m 1</kbd> : 하나의 프로세스로 실행
  - <kbd>--connect jdbc:mysql://mysql:3306/testdb</kbd> : 대상 서버 Connection String
  - <kbd>--username</kbd> : 이용자
  - <kbd>--password</kbd> : 패스워드
  - <kbd>--table</kbd> : 테이블이름
  - <kbd>--target-dir</kbd> : 저장경로 (컨테이너 내부의 로컬 저장소)
  - <kbd>--as-parquetfile</kbd> : 옵션을 추가하면 파케이 포맷으로 저장됩니다
  - <kbd>--fields-terminated-by '\t'</kbd> : 문자열을 구분자로 텍스트 파일로 저장됩니다 (파케이 옵션과 사용 불가)
  - <kbd>--relaxed-isolation</kbd> : 테이블에 Shared Lock 을 잡지 않고 가져옵니다
  - <kbd>--delete-target-dir</kbd> : 대상 경로가 있다면 삭제 후 수집합니다

* 테이블 수집 합니다
```bash
# docker
ask sqoop import -jt local -fs local -m 1 --connect jdbc:mysql://mysql:3306/testdb \
  --username sqoop --password sqoop --table student --target-dir /home/sqoop/target/student
```
<br>

* 로컬 저장소에 제대로 수집이 되었는지 확인합니다
```bash
# docker
ls /home/sqoop/target/student
ask cat /home/sqoop/target/student/part-m-00000
```
<br>


### 2-3. 저장 파일 포맷을 변경하며 수집

> 스쿱은 텍스트 파일 (TSV, CSV) 뿐만 아니라 파케이 (Parquet 컬럼지향 바이너리 압축 포맷) 포맷으로 저장이 가능합니다

#### 2-3-1. 탭 구분자 포맷으로 저장

<details><summary>[실습] 앞서 생성된 학생(student) 테이블을 탭으로 구분된 포맷으로 로컬 `/home/sqoop/target/student_tab` 경로에 저장하세요 </summary>

* 스쿱 명령어로 테이블을 수집합니다
```bash
# docker
ask sqoop import -jt local -fs local -m 1 --connect jdbc:mysql://mysql:3306/testdb \
  --username sqoop --password sqoop --table student --target-dir /home/sqoop/target/student_tab \
  --fields-terminated-by '\t'
```
* 생성된 파일이 텍스트파일로 저장되었는지 확인합니다
```bash
# docker
head /home/sqoop/target/student_tab/part-m-00000
```

</details>
<br>


#### 2-3-2. 파케이 포맷으로 저장

> 파케이 포맷 저장 <kbd>--as-parquetfile</kbd> 

<details><summary>[실습] 앞서 생성된 학생(student) 테이블을 파케이 포맷으로 로컬 `/home/sqoop/target/student_parquet` 경로에 저장하세요 </summary>

* 스쿱 명령어로 테이블을 수집합니다
```bash
# docker
ask sqoop import -jt local -fs local -m 1 --connect jdbc:mysql://mysql:3306/testdb \
  --username sqoop --password sqoop --table student --target-dir /home/sqoop/target/student_parquet \
  --as-parquetfile
```
* 생성된 파일이 파케이로 저장되었는지 확인후 파일명을 기억해둡니다
```
# docker
ask ls -d1 /home/sqoop/target/student_parquet/*
```

</details>
<br>


#### 2-3-3. 파케이 포맷 읽기

> 파케이 포맷은 바이너리 포맷이라 문서편집기 등을 통해 직접 확인할 수 없기 때문에 별도의 도구를 통해서만 읽어들일 수 있습니다

* 생성된 파일이 파케이로 저장되었는지 확인후 파일명을 기억해둡니다
```bash
# docker
filename=`ls -d1 /home/sqoop/target/student_parquet/*`
```
<br>

* 파케이 포맷으로 저장된 테이블을 출력합니다 
  - 파케이 포맷의 파일은 바이너리 포맷이라 cat 혹은 vi 등으로 내용을 확인할 수 없습니다
  - 서버에 설치된 /jdbc/parquet-tools-1.8.1.jar 어플리케이션을 이용하여 확인이 가능합니다
```bash
# docker
hadoop jar /jdbc/parquet-tools-1.8.1.jar head file://${filename}
```
<br>

* 파케이 포맷 도구를 이용하여 사용가능한 기능
  - <kbd>head -n 5</kbd> : 상위 5개의 문서를 출력합니다 (default: 5)
  - <kbd>cat</kbd> : 문서를 그대로 출력합니다
  - <kbd>schema</kbd> : 테이블 스키마를 출력합니다
  - <kbd>meta</kbd> : 파케이 포맷의 메타데이터를 출력합니다 
  - <kbd>dump</kbd> : 텍스트 포맷으로 출력 합니다

<details><summary>[실습] 같은 방식으로 해당 파케이 파일의 상위(head) 3개문서, 스키마(schema), 메타(meta) 출력을 해보세요 </summary>

* 상위 3개 문서 반환
```bash
# docker
ask hadoop jar /jdbc/parquet-tools-1.8.1.jar head -n 3 file://${filename}
```
* 스키마 출력 
```bash
# docker
ask hadoop jar /jdbc/parquet-tools-1.8.1.jar schema file://${filename}
```
* 메타정보 출력
```bash
# docker
ask hadoop jar /jdbc/parquet-tools-1.8.1.jar meta file://${filename}
```

</details>
<br>


### 2-4. 클러스터 환경에서 하둡 저장소로 예제 테이블 수집

> **"클러스터 모드"** 란? 분산 저장/처리 엔진을 활용하여 원격지 장비의 리소스를 활용하여 원격 디스크에 저장할 수 있는 모드입니다

* <kbd>-fs namenode:port</kbd> : File System 이 분산 파일시스템 의미 (Ex. HDFS)
* <kbd>-jt jobtracker:port</kbd> : Job Tracker 가 분산 처리시스템 의미 (Ex. YARN)
  - 본 예제에서는 관련 설정이 되어 있으므로 -fs, -jt 옵션을 지정하지 않아도 됩니다
  - 저장경로의 경우에도 hdfs:// 는 명시하지 않아도 hdfs 에 저장됩니다

#### 2-4-1. 클러스터 환경에서 에제 테이블 `seoul_popular_trip`을 수집 합니다

* 클러스터 환경의 경우 `-jt`, `-fs` 옵션이 없으며, 저장 경로를 하둡 경로로 인식합니다
  - 명시적으로 hdfs:// 를 넣어도 무관합니다
```bash
# docker
ask sqoop import -m 1 --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
  --table seoul_popular_trip --target-dir /user/sqoop/target/seoul_popular_trip
```
<br>

* 원격 하둡 저장소에 제대로 수집이 되었는지 확인합니다
```bash
# docker
hadoop fs -ls /user/sqoop/target/seoul_popular_trip
ask hadoop fs -cat /user/sqoop/target/seoul_popular_trip/part-m-00000
```

#### 2-4-2. 예제 실습이 종료되었으므로 <kbd><samp>Ctrl</samp>+<samp>D</samp></kbd> 혹은 <kbd>exit</kbd> 명령으로 컨테이너를 종료합니다
<br>

[목차로 돌아가기](#2일차-데이터-엔지니어링-수집)

<br>
<br>


## 3. 테이블 익스포트 실습

> 스쿱은 관계형 데이터를 하둡 분산 저장소에 저장(import)도 하지만, 반대로 관계형 데이터베이스에 다시 적재(export)하는 기능도 있습니다. 특히 **적재 서비스는 파일 스키마와 테이블 스키마가 다른 경우 전체 작업이 실패**하므로 사전에 확인해 둘 필요가 있습니다


### 3-1. 새로운 테이블을 생성하고 적재

#### 3-1-1. 동일한 스키마를 가진 테이블(`seoul_popular_exp`) 생성

```bash
# terminal
docker-compose exec mysql mysql -usqoop -psqoop
```
* 테스트 적재를 위한 테이블을 생성합니다
```sql
# mysql>

use testdb;

CREATE TABLE testdb.seoul_popular_exp (
  category INT NOT NULL
  , id INT NOT NULL
  , name VARCHAR(100)
  , address VARCHAR(100)
  , naddress VARCHAR(100)
  , tel VARCHAR(20)
  , tag VARCHAR(500)
) CHARACTER SET UTF8 COLLATE UTF8_GENERAL_CI;
```
```sql
show tables;
```
<br>


### 3-2. 적재 오류시에 디버깅 하는방법

> 적재 서비스는 스키마 불일치에 따른 실패가 자주 발생하는데 의도적으로 발생시켜 디버깅하는 방법을 익힙니다

#### 3-2-1. 구분자 오류에 따른 실패 확인

* 적재 작업을 수행하면 오류가 발생하고 예외가 발생하고, 정확한 오류를 확인할 수 없습니다
```bash
# docker
ask sqoop export -m 1 --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
  --table seoul_popular_exp --export-dir /user/sqoop/target/seoul_popular_trip
```
<br>


#### 3-2-2. 탭구분자로 테이블 재수집

* 수집한 테이블의 경우 콤마를 기준으로 수집했는데, 필드에 콤마(,)가 들어가 있어서 export 시에 필드의 개수가 맞지 않는다는 오류를 확인합니다
  - 방법#1. 적재 수행 시에 콤마 구분자로 수행하거나 (단, 이번 예제는 내부에 콤마가 들어가 있는 튜플이 있어서 사용할 수 없다)
  - 방법#2. 테이블 수집을 탭 구분자로 변경하거나 (하여 콤마 보다는 탭을 구분자로 선택하는 것이 좀 더 일반적입니다)

```bash
# docker
ask sqoop import -m 1 --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
  --table seoul_popular_trip --target-dir /user/sqoop/target/seoul_popular_exp \
  --fields-terminated-by '\t' --delete-target-dir 
```
<br>

```bash
# docker
ask hadoop fs -cat /user/sqoop/target/seoul_popular_exp/part-m-00000 | more
```

<details><summary>[실습] 출력 결과 확인</summary>

> 출력 결과가 아래와 같다면 성공입니다

```text
0    281    통인시장    110-043 서울 종로구 통인동 10-3     03036 서울 종로구 자하문로15길 18     02-722-0911    엽전도시락,종로통인시장,통인시장닭꼬치,런닝맨,엽전시장,통인시장데이트,효자베이커리,통인시장, 1박2일,기름떡볶이
0    345    타르틴    140-863 서울 용산구 이태원동 119-15     04350 서울 용산구 이태원로23길 4 (이태원동)     02-3785-3400    타르틴,이태원디저트카페,파이,런닝맨,파이맛집,이태원맛집, 유재석,식신로드,타르트맛집
```

</details>
<br>


#### 3-2-3. 탭구분자로 테이블 재적재

* 탭 구분자로 익스포트된 경로의 파일을 이용하여 다시 익스포트를 수행합니다
```bash
ask sqoop export -m 1 --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
  --table seoul_popular_exp --export-dir /user/sqoop/target/seoul_popular_exp \
  --fields-terminated-by '\t' 
```
<details><summary>[실습] 출력 결과 확인</summary>

> 출력 결과가 아래와 같다면 성공입니다

```text
21/07/17 10:09:44 INFO mapreduce.ExportJobBase: Transferred 483.1064 KB in 12.7846 seconds (37.7882 KB/sec)
21/07/17 10:09:44 INFO mapreduce.ExportJobBase: Exported 1956 records.
```

</details>
<br>


### 3-3. 테이블 익스포트 시에 스테이징을 통한 적재

> 적재 시에 도중에 일부 컬럼 때문에 실패하는 경우 대상 테이블에 일부 데이터만 적재되어 사용되는 경우가 있는데 이러한 경우를 회피하기 위해 스테이징 테이블을 사용할 수 있습니다. 즉, 한 번 테스트로 적재를 해보는 개념으로 이해하시면 됩니다

* 스테이징 테이블은 원본 테이블을 그대로 두고 별도의 스테이징 테이블에 적재 후 완전 export 가 성공하면 원본 테이블을 clear 후 적재합니다

#### 3-3-1. 아래와 같이 임시 스테이징 테이블을 동일한 스키마로 생성하고 익스포트를 수행합니다

```sql
# mysql>
CREATE TABLE testdb.seoul_popular_stg (
  category INT NOT NULL
  , id INT NOT NULL
  , name VARCHAR(100)
  , address VARCHAR(100)
  , naddress VARCHAR(100)
  , tel VARCHAR(20)
  , tag VARCHAR(500)
) CHARACTER SET UTF8 COLLATE UTF8_GENERAL_CI;
```
```sql
show tables;
```
<br>

#### 3-3-2. 수행 시에 맵 갯수를 4개로 늘려서 테스트 해보겠습니다

* 이미 적재된 테이블에 다시 적재하는 경우는 중복 데이터가 생성되므로  삭제 혹은 TRUNCATE 는 수작업으로 수행되어야만 합니다

```bash
# docker
sqoop eval --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
  -e "TRUNCATE seoul_popular_exp"

ask sqoop export -m 4 --connect jdbc:mysql://mysql:3306/testdb --username sqoop --password sqoop \
  --table seoul_popular_exp --export-dir /user/sqoop/target/seoul_popular_exp \
  --fields-terminated-by '\t' --staging-table seoul_popular_stg --clear-staging-table
```

#### 3-3-3. 적재 과정에서 stg 및 exp 테이블을 조회해 보면 상태를 확인할 수 있습니다

```sql
# docker
cmd "SELECT COUNT(1) FROM seoul_popular_stg"
cmd "SELECT COUNT(1) FROM seoul_popular_exp"
```

> 적재시에 4개의 맵 작업수로 수행된 사항에 대해서도 디버깅 해보시면 어떻게 동작하는 지 확인할 수 있습니다

<details><summary>[실습] 출력 결과 확인</summary>

> 출력 결과가 아래와 같다면 성공입니다 - 단, 수집 과정에서는 `seoul_popular_stg` 카운트가 나오거나 exp 테이블 레코드가 없을 수 있습니다

```text
# "SELECT COUNT(1) FROM seoul_popular_stg"
-----------------------
| COUNT(1)            |
-----------------------
| 0                   |
-----------------------
# "SELECT COUNT(1) FROM seoul_popular_exp"
-----------------------
| COUNT(1)            |
-----------------------
| 1956                |
-----------------------
```

</details>

#### 3-3-4. 테이블 수집 실습이 종료되었으므로 <kbd><samp>Ctrl</samp>+<samp>D</samp></kbd> 혹은 <kbd>exit</kbd> 명령으로 컨테이너를 종료합니다

* 테이블 수집 관련 컨테이너를 종료합니다

```bash
cd /home/ubuntu/work/data-engineer-${course}-training/day2
docker-compose down
```

[목차로 돌아가기](#2일차-데이터-엔지니어링-수집)

<br>
<br>



---

## 파일 수집 기본

## 1. 최신버전 업데이트
> 원격 터미널에 접속하여 관련 코드를 최신 버전으로 내려받고, 과거에 실행된 컨테이너가 없는지 확인하고 종료합니다

### 1-1. 최신 소스를 내려 받습니다
```bash
# terminal
cd /home/ubuntu/work/data-engineer-${course}-training
git pull
```
<br>

### 1-2. 현재 기동되어 있는 도커 컨테이너를 확인하고, 종료합니다

#### 1-2-1. 현재 기동된 컨테이너를 확인합니다
```bash
# terminal
docker ps -a
```
<br>


#### 1-2-2. 기동된 컨테이너가 있다면 강제 종료합니다
```bash
# terminal 
docker rm -f `docker ps -aq`
```
> 다시 `docker ps -a` 명령으로 결과가 없다면 모든 컨테이너가 종료되었다고 보시면 됩니다
<br>


### 1-3. 이번 실습은 예제 별로 다른 컨테이너를 사용합니다

> `cd /home/ubuntu/work/data-engineer-${course}-training/day2/`<kbd>ex1</kbd> 와 같이 마지막 경로가 다른 점에 유의 하시기 바랍니다

* 1번 실습의 경로는 <kbd>ex1</kbd>이므로 아래와 같습니다
```bash
# terminal
cd /home/ubuntu/work/data-engineer-${course}-training/day2/ex1
```

[목차로 돌아가기](#2일차-데이터-엔지니어링-수집)

<br>
<br>



## 2. 플루언트디를 웹서버처럼 사용하기

> 플루언트디 로그 수집기는 **에이전트 방식의 데몬 서버**이기 때문에 일반적인 웹서버 처럼 *http 프로토콜을 통해서도 로그를 전송받을 수 있습*니다

* 가장 일반적인 데이터 포맷인 *JSON 으로 로그를 수신하고, 표준출력(stdout)으로 출력*하는 예제를 실습해봅니다
* 다른 부서 혹은 팀내의 다른 애플리케이션 간의 **로그 전송 프로토콜이 명확하지 않은 경우 가볍게 연동해볼 만한 모델**입니다

![ex1](images/ex1.png)

### 2-1. 도커 컨테이너 기동
```bash
cd /home/ubuntu/work/data-engineer-${course}-training/day2/ex1
docker-compose pull
docker-compose up -d
```
<br>


### 2-2. 도커 및 플루언트디 설정파일 확인

> 기본 설정은 /etc/fluentd/fluent.conf 파일을 바라보는데 예제 환경에서는 `docker-compose.yml` 설정에서 해당 위치에 ex1/fluent.conf 파일을 마운트해 두었기 때문에 컨테이너 환경 내에서 바로 실행을 해도 잘 동작합니다. 별도로 `fluentd -c /etc/fluentd/fluent.conf` 로 실행해도 동일하게 동작합니다

#### 2-2-1 도커 컴포즈 파일 구성 `docker-compose.yml`

```yml
version: "3"

services:
  fluentd:
    container_name: fluentd
    image: psyoblade/data-engineer-fluentd:2.1
    user: root
    tty: true
    ports:
      - 9880:9880
    volumes:
      - ./fluent.conf:/etc/fluentd/fluent.conf
      - ./send_http.sh:/home/root/send_http.sh
    working_dir: /home/root

networks:
  default:
    name: default_network
```
> 플루언트디가 9880 포트를 통해 http 웹서버를 기동했기 때문에 컴포즈 파일에서 해당 포트를 노출시킨 점도 확인합니다

#### 2-2-2 플루언트디 파일 구성 `fluent.conf`
```xml
<source>
    @type http
    port 9880
    bind 0.0.0.0
</source>

<match test>
    @type stdout
</match>
```
<br>


### 2-3. 플루언트디 기동 및 확인

#### 2-3-1. 에이전트 기동을 위해 컨테이너로 접속 후, 에이전트를 기동합니다
```bash
# terminal
docker-compose exec fluentd bash
```
```bash
# docker
fluentd
```
* `fluentd -c /etc/fluentd/fluent.conf`로 기동해도 동일하게 동작합니다

<details><summary>[실습] 출력 결과 확인</summary>

> 출력 결과가 오류가 발생하지 않고, 아래와 유사하다면 성공입니다

```bash
[Default] Start fluentd agent w/ default configuration
/opt/td-agent/embedded/bin/fluentd -c /etc/fluentd/fluent.conf
2021-07-18 11:03:23 +0000 [info]: parsing config file is succeeded path="/etc/fluentd/fluent.conf"
2021-07-18 11:03:23 +0000 [info]: using configuration file: <ROOT>
  <source>
    @type http
    port 9880
    bind "0.0.0.0"
  </source>
  <match test>
    @type stdout
  </match>
</ROOT>
2021-07-18 11:03:23 +0000 [info]: starting fluentd-1.11.5 pid=22 ruby="2.4.10"
2021-07-18 11:03:23 +0000 [info]: spawn command to main:  cmdline=["/opt/td-agent/embedded/bin/ruby", "-Eascii-8bit:ascii-8bit", "/opt/td-agent/embedded/bin/fluentd", "-c", "/etc/fluentd/fluent.conf", "--under-supervisor"]
2021-07-18 11:03:24 +0000 [info]: adding match pattern="test" type="stdout"
2021-07-18 11:03:24 +0000 [info]: adding source type="http"
2021-07-18 11:03:24 +0000 [info]: #0 starting fluentd worker pid=31 ppid=22 worker=0
2021-07-18 11:03:24 +0000 [info]: #0 fluentd worker is now running worker=0
```

</details>
<br>


#### 2-3-2. 별도의 터미널에서 CURL 명령으로 확인합니다

* 웹 브라우저를 통해 POST 전달이 안되기 때문에 별도 터미널로 접속합니다
  - 클라우드 터미널에 curl 설치가 되어있지 않을 수도 있으므로 도커 컨테이너에 접속합니다
```bash
# terminal
docker-compose exec fluentd bash
```
```bash
# docker
curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:9880/test
# HTTP/1.1 200 OK
# Content-Type: text/plain
# Connection: Keep-Alive
# Content-Length: 0
```
> 사전에 배포된 `send_http.sh` 를 실행해도 동일한 결과를 얻습니다

<details><summary>[실습] 출력 결과 확인</summary>

> 에이전트가 기동된 컨테이너의 화면에는 아래와 같이 수신된 로그를 출력하면 성공입니다

```text
2021-07-18 11:12:47.866146971 +0000 test: {"action":"login","user":2}
2021-07-18 11:13:38.334081170 +0000 test: {"action":"login","user":2}
```

</details>
<br>


#### 2-3-3. 실습이 끝났으므로 플루언트디 에이전트를 컨테이너 화면에서 <kbd><samp>Ctrl</samp>+<samp>C</samp></kbd> 명령으로 종료합니다

![stop](images/stop.png)

#### 2-3-4. 1번 예제 실습이 모두 종료되었으므로 <kbd><samp>Ctrl</samp>+<samp>D</samp></kbd> 혹은 <kbd>exit</kbd> 명령으로 컨테이너를 종료합니다

> 컨테이너가 종료된 터미널의 프롬프트(도커의 경우 root@2a50c30293829 형식)를 통해 확인할 수 있습니다


[목차로 돌아가기](#2일차-데이터-엔지니어링-수집)

<br>
<br>



## 3. 수신된 로그를 로컬에 저장하는 예제

> 로그를 전송해 주는 서비스 혹은 애플리케이션을 별도로 기동하는 것도 귀찮은 일이기 때문에, 플루언트디에서 제공하는 로그를 자동으로 발생시키는 더미 로그 플러그인을 이용하여 로그를 발생시키고, 발생된 로그를 로컬 저장소에 저장하는 예제를 실습합니다

* 플루언트디 더미 에이전트를 이용하여 임의의 로그를 자동으로 발생시킴
* 발생된 로그(이벤트)를 로컬 저장소에 임의의 경로에 저장합니다

![ex2](images/ex2.png)
<br>


### 3-1. 이전 컨테이너 종료 및 컨테이너 기동

> 이전 실습에서 기동된 컨테이너를 종료 후, 기동합니다.

```bash
cd /home/ubuntu/work/data-engineer-${course}-training/day2/ex1
docker-compose down

cd /home/ubuntu/work/data-engineer-${course}-training/day2/ex2
docker-compose pull
docker-compose up -d
```
<br>


#### 3-1-1 도커 컴포즈 파일 구성 `docker-compose.yml`

```yml
version: "3"

services:
  fluentd:
    container_name: fluentd
    image: psyoblade/data-engineer-fluentd:2.1
    user: root
    tty: true
    ports:
      - 9880:9880
    volumes:
      - ./fluent.conf:/etc/fluentd/fluent.conf
      - ./target:/fluentd/target
    working_dir: /home/root

networks:
  default:
    name: default_network
```
> 저장되는 로그를 호스트 장비의 볼륨에 마운트하여 컨테이너가 종료되는 경우에도 유지될 수 있도록 구성합니다
<br>


#### 3-1-2 플루언트디 파일 구성 `fluent.conf`
```xml
<source>
    @type dummy
    tag lgde.info
    size 5
    rate 1
    auto_increment_key seq
    dummy {"info":"hello-world"}
</source>

<source>
    @type dummy
    tag lgde.debug
    size 3
    rate 1
    dummy {"debug":"hello-world"}
</source>

<filter lgde.info>
    @type record_transformer
    <record>
        table_name ${tag_parts[0]}
    </record>
</filter>

<match lgde.info>
    @type file
    path_suffix .log
    path /fluentd/target/${table_name}/%Y%m%d/part-%Y%m%d.%H%M
    <buffer time,table_name>
        timekey 1m
        timekey_wait 10s
        timekey_use_utc false
        timekey_zone +0900
    </buffer>
</match>

<match lgde.debug>
    @type stdout
</match>
```
> 더미 에이전트가 로그를 발생시키고, 발생된 로그를 로컬 저장소에 저장합니다. 

* 더미 에이전트 로그 생성
  - lgde.info 와 lgde.debug 2가지 이벤트가 발생
  - info 이벤트만 filter, match 되도록 구성
  - debug 이벤트는 stdout 출력만 되도록 구성
* 필터 플러그인 구성
  - `record_transformer`를 통해 별도의 필트를 추가합니다
  - `${tag_parts[0]}` : 태그의 0번째 즉 lgde 를 말합니다
  - `record` : 앨리먼트를 통해서 `table_name`:`lgde` 라는 값이 추가됩니다
* 매치 플러그인 구성
  - 지정된 경로에 log 파일로 생성합니다
  - 1분에 한 번 저장하되 10초 까지 지연된 로그를 저장합니다
  - 한국 시간 기준으로 포맷을 생성 유지합니다

<br>


### 3-2. 에이전트 기동 및 확인

#### 3-2-1. 에이전트 기동을 위해 컨테이너로 접속 후, 에이전트를 기동합니다

```bash
# terminal
docker-compose exec fluentd bash
```
```bash
# docker
fluentd
```
* 아래와 같이 출력되고 있다면 정상입니다 (lgde.debug)
```bash
# docker
2021-07-18 11:52:24 +0000 [info]: #0 starting fluentd worker pid=58 ppid=49 worker=0
2021-07-18 11:52:24 +0000 [info]: #0 fluentd worker is now running worker=0
2021-07-18 11:52:25.081705098 +0000 lgde.debug: {"debug":"hello-world"}
2021-07-18 11:52:25.081710640 +0000 lgde.debug: {"debug":"hello-world"}
```
<br>


#### 3-2-2. 별도의 터미널에서 생성되는 로그 파일을 확인합니다

```bash
# terminal
cd /home/ubuntu/work/data-engineer-basic-training/day2/ex2
docker-compose exec fluentd bash
```
```bash
# docker
tree /fluentd/target/
```

<details><summary>[실습] 출력 결과 확인</summary>

> 출력 결과가 오류가 발생하지 않고, 아래와 유사하다면 성공입니다

```text

root@7d33f313cc13:~# 
/fluentd/target/
├── ${table_name}
│   └── %Y%m%d
│       └── part-%Y%m%d.%H%M
│           ├── buffer.b5c7647447f29a8c135e1164e39113f3b.log
│           └── buffer.b5c7647447f29a8c135e1164e39113f3b.log.meta
└── lgde
    └── 20210718
        └── part-20210718.2051_0.log

5 directories, 3 files
```

</details>
<br>

### 3-3. 에이전트 및 컨테이너 종료

#### 3-3-1. 실습이 끝났으므로 플루언트디 에이전트를 컨테이너 화면에서 <kbd><samp>Ctrl</samp>+<samp>C</samp></kbd> 명령으로 종료합니다

#### 3-3-2. 1번 예제 실습이 모두 종료되었으므로 <kbd><samp>Ctrl</samp>+<samp>D</samp></kbd> 혹은 <kbd>exit</kbd> 명령으로 컨테이너를 종료합니다


[목차로 돌아가기](#2일차-데이터-엔지니어링-수집)

<br>
<br>



## 4. 서버에 남는 로그를 지속적으로 모니터링하기

> 앞의 예제에서는 로그가 자동으로 생성된다는 것을 가정했는데, 이번 예제에서는 실제 로그가 생성되는 것을 재현해보고, 임의의 아파치 웹서버로그가 생성되는 것을 잘 모니터링하면서 수집하는 지를 실습합니다

![ex3](images/ex3.png)


### 4-1. 이전 컨테이너 종료 및 컨테이너 기동

> 이전 실습에서 기동된 컨테이너를 종료 후, 기동합니다.

```bash
cd /home/ubuntu/work/data-engineer-${course}-training/day2/ex2
docker-compose down

cd /home/ubuntu/work/data-engineer-${course}-training/day2/ex3
docker-compose pull
docker-compose up -d
```
<br>


#### 4-1-1 도커 컴포즈 파일 구성 `docker-compose.yml`

```yml
version: "3"

services:
  fluentd:
    container_name: fluentd
    image: psyoblade/data-engineer-fluentd:2.1
    user: root
    tty: true
    volumes:
      - ./apache_logs:/home/root/apache_logs
      - ./flush_logs.py:/home/root/flush_logs.py
      - ./fluent.conf:/etc/fluentd/fluent.conf
      - ./source:/fluentd/source
      - ./target:/fluentd/target
    working_dir: /home/root

networks:
  default:
    name: default_network
```
> 아파치 웹서버의 로그가 생성될 source 경로와 수집된 로그가 저장될 target 경로를 호스트 경로에 마운트 되었습니다.
<br>



#### 4-1-2 플루언트디 파일 구성 `fluent.conf`

```xml
<source>
    @type tail
    @log_level info
    path /fluentd/source/accesslogs
    pos_file /fluentd/source/accesslogs.pos
    refresh_interval 5
    multiline_flush_interval 5
    rotate_wait 5
    open_on_every_update true
    emit_unmatched_lines true
    read_from_head false
    tag weblog.info
    <parse>
        @type apache2
    </parse>
</source>

<match weblog.info>
    @type file
    @log_level info
    add_path_suffix true
    path_suffix .log
    path /fluentd/target/${tag}/%Y%m%d/accesslog.%Y%m%d.%H
    <buffer time,tag>
        timekey 1h
        timekey_use_utc false
        timekey_wait 10s
        timekey_zone +0900
        flush_mode immediate
        flush_thread_count 8
    </buffer>
</match>

<match weblog.debug>
    @type stdout
    @log_level debug
</match>
```
> 소스 경로에 저장되는 로그를 모니터링하여 타겟 경로에 저장합니다 

* 소스 플러그인 설정
  - 로그 레벨은 info 로 지정된 경로에서 tail 플러그인으로 모니터링합니다
  - weblog.info 태그를 이용하였으며, apache2 로그를 파싱합니다
  - 로그파일이 새로 작성(log-rotate) 되더라도 최대 5초간 대기합니다
* 매치 플러그인 설정
  - 즉각적인 확인을 위해 최대한 자주 플러시 하도록 설정 했습니다
<br>


#### 4-1-3. 아파치 로그 생성기 (`flush_logs.py`) 코드를 분석합니다

> 아파치 웹서버의 로그를 실제와 유사하게 저장하고, 롤링까지 하게 만들기 위해서 별도의 파이썬 스크립트가 필요합니다.

```python
#!/usr/bin/env python
import sys, time, os, shutil

# 1. read apache_logs flush every 100 lines until 1000 lines
# 2. every 1000 lines file close & rename file with seq
# 3. create new accesslogs and goto 1.

def readlines(fp, num_of_lines):
    lines = ""
    for line in fp:
        lines += line
        num_of_lines = num_of_lines - 1
        if num_of_lines == 0:
            break
    return lines

fr = open("apache_logs", "r")
for x in range(0, 10):
    fw = open("source/accesslogs", "w+")
    for y in range(0, 10):
        lines = readlines(fr, 100)
        fw.write(lines)
        fw.flush()
        time.sleep(0.1)
        sys.stdout.write(".")
        sys.stdout.flush()
    fw.close()
    print("file flushed ... sleep 10 secs")
    time.sleep(10)
    shutil.move("source/accesslogs", "source/accesslogs.%d" % x)
    print("renamed accesslogs.%d" % x)
fr.close()
```
> 원본 로그를 읽어서 source 경로에 저장하는 스크립트

* 한 번에 100 라인씩을 읽어서 source 의 accesslogs 에 flush 합니다
* 매 1000라인 마다 마치 용량이 커서 롤링된 것처럼 파일명을 변경합니다
* 롤링된 이후에는 원본 accesslogs 파일에 계속 append 합니다
<br>


### 4-2. 에이전트 기동 및 확인

#### 4-2-1. 에이전트 기동을 위해 컨테이너로 접속 후, 에이전트를 기동합니다

```bash
# terminal
docker-compose exec fluentd bash
```
```bash
# docker
fluentd
```
> 플루언트디의 경우 기동 시에 오류가 없었다면 정상적으로 기동 되었다고 보시면 됩니다
<br>



#### 4-2-2. 시스템 로그를 임의로 생성

> 웹서버의 로그를 생성하기 어렵기 때문에 기존에 적재된 액세스로그를 읽어서 주기적으로 accesslog 파일에 append 하는 프로그램(`flush_logs.py`)을 통해서 시뮬레이션 합니다

* 로그 생성기를 통해 accesslog 파일을 계속 source 경로에 append 하고, target 경로에서는 수집되는지 확인합니다

```bash
# terminal
cd /home/ubuntu/work/data-engineer-basic-training/day2/ex3
docker-compose exec fluentd bash
```
```bash
python flush_logs.py
```
<br>

```
for x in $(seq 1 100); do tree -L 1 /fluentd/source; tree -L 2 /fluentd/target; sleep 10; done
```
> 위의 명령어로 주기적으로 출력 경로를 확인할 수 있습니다

<details><summary>[실습] 출력 결과 확인</summary>

> 출력 결과가 오류가 발생하지 않고, 아래와 유사하다면 성공입니다

```bash
021-07-18 13:01:20 +0000 [info]: #0 detected rotation of /fluentd/source/accesslogs
2021-07-18 13:01:20 +0000 [info]: #0 following tail of /fluentd/source/accesslogs
2021-07-18 13:01:31 +0000 [info]: #0 detected rotation of /fluentd/source/accesslogs
2021-07-18 13:01:31 +0000 [info]: #0 following tail of /fluentd/source/accesslogs
2021-07-18 13:01:32 +0000 [warn]: #0 pattern not matched: "46.118.127.106 - - [20/May/2015:12:05:17 +0000] \"GET /scripts/grok-py-test/configlib.py HTTP/1.1\" 200 235 \"-\" \"Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html"
2021-07-18 13:01:43 +0000 [info]: #0 detected rotation of /fluentd/source/accesslogs
2021-07-18 13:01:43 +0000 [info]: #0 following tail of /fluentd/source/accesslogs
2021-07-18 13:01:54 +0000 [info]: #0 detected rotation of /fluentd/source/accesslogs; waiting 5.0 seconds
```

```bash
root@2cf7c79e8367:~# for x in $(seq 1 100); do tree -L 1 /fluentd/source; tree -L 2 /fluentd/target; sleep 10; done
/fluentd/source
├── accesslogs.0
├── accesslogs.1
├── accesslogs.2
├── accesslogs.3
├── accesslogs.4
├── accesslogs.5
├── accesslogs.6
├── accesslogs.7
├── accesslogs.8
├── accesslogs.9
└── accesslogs.pos

0 directories, 11 files
/fluentd/target
├── ${tag}
│   └── %Y%m%d
└── weblog.info
    ├── 20150517
    ├── 20150518
    ├── 20150519
    ├── 20150520
    ├── 20150521
    └── 20210718
```

</details>
<br>

### 4-3. 에이전트 및 컨테이너 종료

#### 4-3-1. 실습이 끝났으므로 플루언트디 에이전트를 컨테이너 화면에서 <kbd><samp>Ctrl</samp>+<samp>C</samp></kbd> 명령으로 종료합니다

#### 4-3-2. 1번 예제 실습이 모두 종료되었으므로 <kbd><samp>Ctrl</samp>+<samp>D</samp></kbd> 혹은 <kbd>exit</kbd> 명령으로 컨테이너를 종료합니다


[목차로 돌아가기](#2일차-데이터-엔지니어링-수집)

<br>


### 4-4. 컨테이너 정리
* 테스트 작업이 완료되었으므로 모든 컨테이너를 종료합니다 (한번에 실행중인 모든 컨테이너를 종료합니다)
```bash
cd /home/ubuntu/work/data-engineer-${course}-training/day2/ex3
docker-compose down
```

[목차로 돌아가기](#2일차-데이터-엔지니어링-수집)

