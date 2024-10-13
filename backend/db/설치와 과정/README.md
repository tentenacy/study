# 설치와 과정

## 상용화 방식

MySQL은 **오픈 코어 모델**이라는 상용화 방식을 채택한다.

> 💡 오픈 코어 모델
> 
> : 상용과 오픈소스의 핵심 내용은 동일하면서, 특정 부가 기능들만 상용 버전에 포함하는 방식

  

## 엔터프라이즈 에디션에만 지원하는 부가적인 기능

- Thread Pool
- Enterprise Audit
- Enterprise TDE(Msater Key 관리)
- Enterprise Authentication
- Enterprise Firewall
- Enterprise Monitor
- Enterprise Backup
- MySQL 기술 지원

  

하지만, Percona Server 백업 모니터링 도구 또는 플러그인을 활용하면 이 부족한 부분을 메꿀 수 있다.

  

* * *

  

## MySQL 서버 디렉터리 구조

- bin : MySQL 서버와 클라이언트 프로그램, 그리고 유틸리티를 위한 디렉터리
- include : C/C++ 헤더 파일들이 저장된 디렉터리
- lib : 라이브러리 파일들이 저장된 디렉터
- share : 다양한 지원 파일들이 저장돼 있으며, 에러 메시지나 샘플 설정 파일(my.ini)이 있는 디렉터리

  

* * *

  

## 서버를 안전하게 시작하거나 종료하기

시스템 설정을 적용하여 서버를 시작하고 종료하고 싶다면, mysqld\_safe 스크립트를 사용한다.

이는 my.cnf의 \[mysqld\_safe\] 섹션의 설정들을 참조해서 서버를 시작하거나 종료한다.

  

* * *

  

## 클린 셧다운

MySQL 서버에서는 실제 트랜잭션이 정상적으로 커밋돼도 데이터 파일에 변경된 내용이 기록되지 않고 로그 파일에만 기록돼 있을 수 있다.

사용량이 많은 MySQL 서버에서는 이런 현상이 더 일반적인데, 이는 결코 비정상적인 상황이 아니다.

서버가 종료되고 다시 시작된 이후에도 이런 현상이 지속될 수 있는데, 이 때 **클린 셧다운**을 통해 서버 종료 시 모든 커밋된 내용을 데이터 파일에 기록하고 종료할 수 있다.

> 💡 클린 셧다운   
> : 모든 커밋된 데이터를 데이터 파일에 적용하고 종료하는 것  

  

```
SET GLOBAL innodb_fast_shutdown=0;
systemctl stop mysqld.service
```

클린 셧다운으로 종료되면 다시 MySQL 서버가 기동할 때 별도의 트랜잭션 복구 과정을 진행하지 않아 빠르게 시작할 수 있다.

  

* * *

  

## 서버 접속 시 [localhost](http://localhost) vs 127.0.0.1

`--host=localhost` 옵션을 사용하면 MySQL 클라이언트 프로그램은 항상 소켓 파일을 통해 MySQL 서버에 접속하게 되는데,

이는 ‘Unix domain socket’을 이용하는 방식으로 TCP/IP을 통한 통신이 아닌 유닉스의 프로세스 간 통신(IPC: Inter Process Communication)의 일종이다.

반면, **127.0.0.1**을 사용하는 경우에는 자기 서버를 가리키는 루프백 IP이기는 하지만 TCP/IP 통신 방식을 사용하는 것이다.

```
mysql --host=localhost -uroot -p
mysql -h127.0.0.1 -uroot -p
```

  

* * *

  

## 시스템 변수

> 💡 시스템 변수  
> : MySQL 서버 기동 시 메모리나 작동 방식을 초기화하고, 접속된 사용자를 제어하기 위해 저장된 값  

  

```
SHOW GLOBAL VARIABLES;
SHOW VARIABLES;
```

시스템 변수는 적용 범위에 따라 **글로벌 변수**와 **세션 변수**로 구분된다.

> 💡 글로벌 변수
> 
> : 하나의 MySQL 서버 인스턴스에서 전체적으로 영향을 미치는 시스템 변수, 주로 MySQL 서버 자체에 관련된 설정일 때가 많음
> 
> 💡 세션 변수
> 
> : MySQL 클라이언트가 MySQL 서버에 접속할 때 기본으로 부여하는 옵션의 기본값을 제어하는 시스템 변수

또한, 시스템 변수는 MySQL 서버가 기동 중인 상태에서 변경 가능한지에 따라 **동적 변수**와 **정적 변수**로 구분된다.

```
SHOW GLOBAL VARIABLES LIKE '%max_connections%';
SET GLOBAL max_connections=500;
SHOW GLOBAL VARIABLES LIKE 'max_connections';
```

SET 명령을 통해 변경되는 시스템 변숫값이 MySQL 설정 파일에 반영되는 것은 아니기 때문에,

서버가 재시작해도 설정을 영구히 적용하려면 my.cnf 파일도 변경해줘야 한다.

MySQL 8.0 버전부터는 `SET PERSIST` 명령을 이용하면 이 두 가지 작업을 동시에 처리할 수 있다.

변경된 값은 mysqld-auto.cnf라는 별도의 설정 파일에 기록된다.

그리고 MySQL 서버가 다시 시작될 때 기본 설정파일과 자동 생성된 mysqld-auto.cnf 파일을 같이 참조해서 시스템 변수를 적용한다.

단, `SET PERSIST` 명령은 세션 변수에는 적용되지 않는다.

```
SET PERSIST max_connections=5000;
SHOW GLOBAL VARIABLES LIKE 'max_connections';
```

  

```
SET PERSIST_ONLY max_connections=5000;
SHOW GLOBAL VARIABLES LIKE 'max_connections';
```

`RESET PERSIST` 명령으로 모든 시스템 변수를 삭제할 수도 있다. 특정 시스템 변수만 삭제하려면 다음과 같이 할 수 있다.

```
RESET PERSIST max_connections;
RESET PERSIST IF EXISTS max_connections;
```

정리하면, `SHOW`나 `SET` 명령에서 `GLOBAL` 키워드를 사용하면 글로벌 시스템 변수를 읽고 변경할 수 있으며,

`GLOBAL` 키워드를 빼면 자동으로 세션 변수를 읽고 변경한다.

그리고 `SET`을 통한 변경은 동적 변수에만 해당한다.