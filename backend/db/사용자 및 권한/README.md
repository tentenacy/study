# 사용자 및 권한

## 사용자 식별

MySQL의 사용자는 계정뿐 아니라 사용자의 접속 지점도 계정의 일부가 된다.

따라서, MySQL에서 계정을 언급할 때는 항상 아이디와 호스트를 함께 명시해야 한다.

만약 사용자 계정에 다음과 같은 계정만 등록돼 있다면 다른 컴퓨터에서는 svc\_id라는 아이디로 접속할 수 없다.

```
'svc_id'@'127.0.0.1' # 127.0.0.1에서 svc_id 아이디로 접속하는 계정
'svc_id'@'%' # 모든 외부 컴퓨터에서 svc_id 아이디로 접속하는 계정
```

  

사용자 계정 식별에서 위 두 계정이 있을 때 MySQL은 범위가 작은 것을 항상 먼저 선택하기 때문에,

첫 번째 계정을 정보를 이용해 인증을 실행한다.

  

* * *

  

## 시스템 계정과 일반 계정

MySQL 8.0부터 계정은 `SYSTEM_USER` 권한을 가지고 있느냐에 따라 **시스템 계정**과 **일반 계정**으로 구분된다.

> 💡 시스템 계정
> 
> : 서버 관리자를 위한 계정
> 
> 💡 일반 계정
> 
> : 응용 프로그램이나 개발자를 위한 계정

  

**시스템 계정에 의한 데이터베이스 서버 관리와 관련된 중요 작업**

- 계정 관리(계정 생성 및 삭제, 그리고 계정의 권한 부여 및 제거)
- 다른 세션(Connection) 또는 그 세션에서 실행 중인 쿼리를 강제 종료
- 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정

  

**MySQL 서버 내장 계정(삭제되지 않도록 주의)**

- ‘mysql.sys’@’localhost’ : MySQL 8.0부터 기본으로 내장된 sys 스키마의 객체(뷰나, 함수, 그리고 프로시저)들의 DEFINER로 사용되는 계정
- ‘mysql.session’@’localhost’ : MySQL 플러그인이 서버로 접근할 때 사용되는 계정
- ‘mysql.infoschema’@’localhost’ : information\_schema에 정의된 뷰의 DEFINER로 사용되는 계정

```
# 잠겨있는 상태의 내장 계정 확인
SELECT user, host, account_locked FROM mysql.user WHERE user LKE 'mysql.%';
```

  

* * *

  

## 계정 생성

MySQL 5.7 버전 까지는 `GRANT` 명령으로 권한 부여와 계정 생성이 동시에 가능했지만,

8.0 버전부터는 계정의 생성은 `CREATE USER` 명령으로, 권한 부여는 `GRANT` 명령으로 구분해서 실행하도록 바뀌었다.

  

**계정 생성 옵션**

- 계정의 인증 방식과 비밀번호
- 비밀번호 관련 옵션(비밀번호 유효 기간, 비밀번호 이력 개수, 비밀번호 재사용 불가 기간)
- 기본 역할(Role)
- SSL 옵션
- 계정 잠금 여부

  

```
# 일반적으로 많이 사용되는 옵션을 가진 CREATE USER 명령
CREATE USER 'user'@'%'
	IDENTIFIED WITH 'mysql_native_password' BY 'password'
	REQUIRED NONE
	PASSWORD EXPIRE INTERVAL 30 DAY
	ACCOUNT UNLOCK
	PASSWORD HISTORY DEFAULT
	PASSWORD REUSE INTERVAL DEFAULT
	PASSWORD REQUIRE CURRENT DEFAULT;
```

1. **IDENTIFIED WITH**

`IDENTIFIED WITH` 뒤에는 반드시 인증 방식을 명시해야 한다.

기본 인증 방식을 사용하고자 한다면 `IDENTIFIED WITH ‘password’` 형식으로 명시하면 된다.

  

**다양한 인증 방식을 제공하는 플러그인**

- Native Pluggable Authentication
    - 암호화 해시값 생성을 위해 SHA-1 알고리즘 사용
    - 비밀번호에 대한 해시 값을 저장해두고, 클라이언트에서 보낸 값과 일치하는지 비교
- **Chaching SHA-2 Pluggable Authentication**
    - 암호화 해시값 생성을 위해 SHA-2 알고리즘 사용
    - Native Pluggable Authentication은 입력이 동일 해시값을 출력하는 데 반해, Caching SHA-2 Authentication은 내부적으로 Salt 키를 활용하여 수천 번의 해시 계산을 수행하여 결과를 만들기 때문에, 동일한 키 값에 대해서도 결과가 다름
    - 시간 단축을 위해 MySQL 서버는 해시 결괏값을 메모리에 캐시해서 사용, 이에 따라 인증 방식의 이름에 ‘Caching’이 붙음
    - SSL/TLS 또는 RSA 키페어를 반드시 사용해야 하기 때문에, 클라이언트 접속 시 SSL 옵션을 활성화해야 함
    - SCRAM(Salted Challenge Response Authentication Mechanism) 인증 방식을 사용
- PAM Pluggable Authentication
    - 유닉스나 리눅스 패스워드 또는 LDAP 같은 외부 인증을 사용할 수 있게 해주는 인증 방식
    - MySQL 엔터프라이즈 에디션에서만 사용 가능
- LDAP Pluggable Authentication
    - LDAP을 이용한 외부 인증을 사용할 수 있게 해주는 인증 방식
    - MySQL 엔터프라이즈 에디션에서만 사용 가능

  

2. **REQUIRE**

MySQL 서버 접속 시 암호화된 SSL/TLS 채널을 사용할 지 여부를 설정한다. 별도 설정하지 않으면 비암호화 채널로 연결한다.

Chaching SHA-2 Authentication 인증 방식을 사용하면 암호화된 채널만으로 MySQL 서버에 접속할 수 있다.

  

3. **PASSWORD EXPIRE**

비밀번호의 유효 기간을 설정하는 옵션이다.

별도 설정하지 않으면 `default_password_lifetime` 시스템 변수에 저장된 기간으로 유효 기간이 설정된다.

  

4. **PASSWORD HISTORY**

한 번 사용했던 비밀번호를 재사용하지 못하게 설정하는 옵션이다.

  

**설정 가능한 옵션**

- `PASSWORD HISTORY DEFAULT`
    - `password_history` 시스템 변수에 저장된 개수만큼 비밀번호의 이력을 저장
    - 저장된 이력에 남아있는 비밀번호는 재사용 할 수 없음
- `PASSWORD HISTORY n`
    - 비밀번호의 이력을 최근 n개 까지만 저장
    - 저장된 이력에 남아있는 비밀번호는 재사용 할 수 없음

  

이전에 사용했던 비밀번호는 `mysql` DB의 `password_history` 테이블에 저장된다.

  

5. **PASSWORD REUSE INTERVAL**

한 번 사용했던 비밀번호의 재사용 금지 기간을 설정한다.

별도 설정하지 않으면 `password_reuse_interval` 시스템 변수에 저장된 기간으로 설정된다.

  

6. **PASSWORD REQUIRE**

비밀번호가 만료되어 새로운 비밀번호로 변경할 때 현재 비밀번호를 필요로 할지 말지를 결정한다.

별도로 설정하지 않으면 `password_require_current` 시스템 변수의 값이 설정된다.

  

7. **ACCOUNT LOCK / UNLOCK**

계정 생성 시 또는 ALTER USER 명령을 사용해 계정 정보를 변경할 때 계정을 사용하지 못하게 잠글지 여부를 결정한다.

  

* * *

  

## 고수준 비밀번호

고수준 비밀번호 관리를 위해 validate\_component를 사용할 수 있다.

validate\_password는 MySQL 5.7 버전까지는 플러그인 형태로 제공됐지만, MySQL 8.0 버전부터는 컴포넌트 형태로 제공된다.

비밀번호 유효기간이나 이력 관리를 통한 재사용 금지 기능뿐만 아니라,

비밀번호를 쉽게 유추할 수 있는 단어들이 사용되지 않게 글자의 조합을 강제하거나 금칙어를 설정하는 기능도 수행한다.

> 💡 고수준 비밀번호  
> : 비밀번호를 쉽게 유추할 수 있는 단어들이 사용되지 않게 글자의 조합을 강제하거나 금칙어 등을 포함  

  

```
# validate_password 컴포넌트 설치 및 확인
INSTALL COMPONENT 'file://component_validate_password';

SELECT * FROM mysql.component;

SHOW GLOBAL VARIABLES LIKE 'validate_password%';
```

  

```
# validate_password 컴포넌트에서 제공하는 시스템 변수
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
```

  

* * *

  

## 이중 비밀번호

다수의 응용 프로그램에서 공용으로 데이터베이스 서버를 사용하기 때문에

데이터베이스 서버의 계정 정보는 쉽게 변경하기 어렵다.

그 중에서도 비밀번호는 서비스 실행 중인 상태에서 변경이 불가능했다.

이러한 문제점을 해결하기 위해 MySQL 8.0 버전부터는 **이중 비밀번호** 기능을 추가했다.

> 💡 이중 비밀번호(Dual Password)
> 
> : 계정의 비밀번호로 2개의 값을 동시에 사용할 수 있는 기능. 이중 비밀번호의 ‘Dual’은 둘 중 하나만 일치하면 로그인이 되는 것을 의미한다.

  

이중 비밀번호에서 Primary는 최근 비밀번호를, Secondary는 이전 비밀번호를 의미한다.

ALTER USER를 통해 계정 정보를 변경할 때 반드시 RETAIN CURRENT PASSWORD 옵션을 추가해야 이중 비밀번호가 활성화된다.

```
# 이전 비밀번호를 Secondary로 설정
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password' RETAIN CURRENT PASSWORD;
```

  

MySQL 서버에 접속하는 모든 응용 프로그램의 재시작이 완료되면 Secondary 비밀번호는 삭제하는 게 좋다.

이중 비밀번호는 어디까지나 서비스 실행 중인 상태에서 계정 정보를 변경하기 위해 사용되는 것이므로

보안적으로 안전하다고 볼 수 없기 때문이다.

```
# Secondary 비밀번호 삭제
ALTER USER 'root'@'localhost' DISCARD OLD PASSWORD;
```

  

* * *

  

## 권한

MySQL 5.7 버전까지 권한은 **글로벌 권한**과 **객체 단위 권한**으로 구분됐다. 이들을 정적 권한이라 한다.

> 💡 글로벌 권한
> 
> : 데이터베이스나 테이블 이외의 객체에 적용되는 권한, GRANT 명령으로 특정 객체를 명시하지 않음
> 
> 💡 객체 권한
> 
> : 데이터베이스나 테이블을 제어하는 데 필요한 권한, GRANT 명령으로 특정 객체를 명시해야 함

  

예외적으로 `ALL`은 글로벌과 객체 권한 두 가지 용도로 사용될 수 있는데,

특정 객체에 `ALL` 권한이 부여되면 해당 객체에 적용될 수 있는 모든 객체 권한을 부여하고,

글로벌로 `ALL` 권한이 부여되면 글로벌 수준에서 가능한 모든 권한을 부여하게 된다.

MySQL 8.0 버전부터는 **동적 권한**이 추가됐다.

예를 들어, MySQL 서버의 컴포넌트나 플러그인이 설치되면 그 때 등록되는 권한을 동적 권한이라 한다.

> 💡 동적 권한  
> : MySQL 서버가 시작되면서 동적으로 생성하는 권한  

  

MySQL 5.7 버전까지는 `SUPER`라는 권한이 데이터베이스 관리를 위해 꼭 필요한 권한이었지만,

MySQL 8.0부터는 `SUPER` 권한은 잘게 쪼개어져 동적 권한으로 분산됐다.

이렇게 권한이 분산되면, 백업 관리자, 복제 관리자와 같이 담당자 개별적으로 꼭 필요한 권한만 부여할 수 있다.

```
# 사용자 권한 부여 포맷
GRANT privilege_list ON db.table TO 'user'@'host';
```

  

privilege\_list에는 구분자 ‘,’를 써서 여러 권한을 명시할 수 있다.

`TO` 키워드 뒤에는 권한을 부여할 대상 사용자를 명시하고,

`ON` 키워드 뒤에는 어떤 DB의 어떤 오브젝트에 권한을 부여할 지 결정할 수 있다.

글로벌 권한은 특정 DB나 테이블에 부여될 수 없기 때문에 `ON` 절에는 항상 `*.*`를 사용하게 된다.

```
# 글로벌 권한 부여
GRANT SUPER ON *.* TO 'user'@'localhost';
```

  

```
# 테이블 권한 부여
GRANT SELECT,INSERT,UPDATE,DELETE ON *.* TO 'user'@'loaclhost';
GRANT SELECT,INSERT,UPDATE,DELETE ON employees.* TO 'user'@'loaclhost';
GRANT SELECT,INSERT,UPDATE,DELETE ON employees.department TO 'user'@'loaclhost';
```

  

```
# 모든 칼럼 혹은 특정 칼럼 권한 부여
GRANT SELECT,INSERT,UPDATE(dept_name) ON employees.department TO 'user'@'localhost';
```

  

칼럼 단위 권한이 하나라도 설정되면 나머지 모든 케이블의 모든 칼럼에 대해서도 권한 체크를 하기 때문에,

칼럼 하나에 대해서만 권한을 설정하더라도 전체적인 성능에 영향을 미칠 수 있다.

칼럼 단위의 접근 권한이 꼭 필요하다면 `GRANT` 명령으로 해결하기보다는,

테이블에서 권한을 허용하고자 하는 칼럼만으로 별도의 뷰(`VIEW`)를 만들어 사용하는 방법도 생각해볼 수 있다.

뷰도 하나의 테이블로 인식되기 때문에 뷰의 칼럼에 대해 권한을 체크하지 않고 뷰 자체에 대한 권한만 체크하게 된다.

  

* * *

  

## 역할(Role)

MySQL 8.0 버전부터 권한을 묶어서 역할을 사용할 수 있게 됐다.

실제 MySQL 서버 내부적으로 역할은 계정과 똑같은 모습을 하고 있다.

```
# 빈 역할 정의
CREATE ROLE
		role_emp_read,
		role_emp_write;
```

  

```
# 각 역할에 대해 실질적 권한 부여
GRANT SELECT ON employees.* TO role_emp_read;
GRANT INSERT,UPDATE,DELETE ON employees.* TO role_emp_write;
```

  

역할은 그 자체로 사용될 수 없고, 계정에 부여해야 한다.

```
# 계정 생성 및 역할 부여
CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'qwerty';
CREATE USER writer@'127.0.0.1' IDENTIFIED BY 'qwerty';

GRANT role_emp_read TO reader@'127.0.0.1';
GRANT role_emp_read, role_emp_write TO writer@'127.0.0.1';
```

  

reader 계정이 `role_emp_read` 역할을 사용할 수 있게 하려면 `SET ROLE` 명령으로 해당 역할을 활성화해야 한다.

역할이 활성화되면 그 역할이 가진 권한은 사용할 수 있는 상태가 되지만,

계정이 로그아웃됐다가 다시 로그인하면 역할이 활성화되지 않은 상태로 초기화돼 버린다.

```
# 역할 활성화
SET ROLE 'role_emp_read';
```

  

MySQL 서버의 역할 자동 활성화 여부를 `activate_all_roles_on_login` 시스템 변수로 설정할 수 있다.

```
# 역할 자동 활성화
SET GLOBAL activate_all_roles_on_login=ON;
```

  

MySQL 서버의 역할은 서버 내부적으로 계정과 동일한 객체로 취급된다.

mysql DB의 user 테이블을 보면 실제 권한과 사용자 계정이 구분 없이 저장된 것을 확인할 수 있다.

```
# 권한과 사용자 계정이 구분 없이 저장
mysql> SELECT user, host, account_locked FROM user;
+------------------+-----------+----------------+
| user             | host      | account_locked |
+------------------+-----------+----------------+
| role_emp_read    | %         | Y              |
| role_emp_write   | %         | Y              |
| reader           | 127.0.0.1 | N              |
| writer           | 127.0.0.1 | N              |
| mysql.infoschema | localhost | Y              |
| mysql.session    | localhost | Y              |
| mysql.sys        | localhost | Y              |
| root             | localhost | N              |
+------------------+-----------+----------------+
```

  

MySQL 서버는 계정과 권한을 어떻게 구분할까? 구분하지 않는다.

하나의 계정에 다른 계정의 권한을 병합하기만 하면 되므로 MySQL 서버는 역할과 계정을 구분할 필요가 없다.

역할의 호스트는 사용자에 권한을 부여했을 때는 의미가 없고, 계정으로서 로그인할 때 의미가 있다.

또한, 역할과 계정은 동일한 객체이지만, `CREATE ROLE`과 `CREATE USER`로 구분해서 지원하는 이유는

데이터베이스 관리의 직무를 분리하여 보안을 강화하는 용도로 사용될 수 있게 하기 위해서다.

`CREATE ROLE` 명령만 실행 가능한 사용자는 역할을 생성할 수 있고,

생성된 역할은 `account_locked` 칼럼의 값이 ‘Y’로 설정돼 있어서 로그인 용도로 사용할 수 없다.