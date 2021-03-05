---
layout: post
title: "[Oracle] How_To INVALID Object 처리방법"
description: "Oracle Trouble Shooting How to No.1"
categories: Oracle Database
author: "Dev. Hoon"
---

# DB 개발 및 운영을 하며 때때로 마주치는 OBJECT INVALID 상태 처리 방법

*****

## 1. INVALID는 왜 발생하는가? 

DB OBJECT들은 서로가 서로에 대한 참조 *DEPENDENCY*를 가지고 있다.

이때, 참조 중인 OBJECT의 *스키마가 변경*되거나, *OBJECT가 드랍*되는 등 더 이상 참조를 할 수 없거나 컴파일 오류를 발생할 경우 OBJECT는 **INVALID** 상태로 변경된다.


## 2. INVALID 조회하기

DB는 자신의 상태를 표현할 수 있는 VIEW들을 가지고 있으며, 이를 *SYSTEM VIEW*라 한다. 바로 이 시스템 뷰를 조회하면 조회 대상의 상태를 확인할 수 있다. SYSTEM VIEW는 ALL, DBA, USER 접두사가 붙으며, 각각은 사용 권한에 따라 조회 가능 여부와 데이터 범위가 다르다.

INVALID는 OBJECT 별로 발생하는 상태이다. 따라서, USER_OBJECTS나 DBA_OBJECTS, ALL_OBJECTS와 같은 SYSTEM VIEW를 확인하면 STATUS 칼럼으로 확인할 수 있다.
​
## 3. INVALID 처리하기

ORACLE의 경우, 자동 컴파일을 지원한다. 컴파일 에러가 나지 않는 경우 (간단히 스키마만 바뀌었거나, TABLE을 REBUILD 하는 등 관련 object의 참조를 해치지 않는 경우)에는 해당 object를 조회/실행할 경우 알아서 컴파일해 준다. 따라서, 간단하게는 **그냥 해당 object를 조회하거나 실행하면 된다.** (특히 VIEW의 경우 조회 한 번만 수행하면 바로 컴파일된다)

그러나 어떤 운영 환경에서는 일정 시간 이후에는 컴파일이 금지되는 환경들이 있다. **자동 컴파일만 믿었다가 컴파일이 금지되어버리면 바로 장애로 이어진다.** INVALID된 OBJECT들은 어쨌든 사용할 수 없는 상태이니, 작업이 끝난 후에는 반드시 확인해서 문제 된 object를 컴파일 해줄 필요가 있다.
​
대부분의 DB TOOL에서는 OBJECT의 상태 조회 시 *COMPILE* 버튼을 제공한다. 정확히 어떤 OBJECT가 INVALID 되었는지 알고 있으며 몇 건 되지 않는다면, 해당 OBJECT 정보를 열고 개별로 클릭해 줘도 무방하다.

그러나 대부분은 어떤 OBJECT가 INVALID 된 것인지 알기 어렵다. Dependency는 경우에 따라 수십개 OBJECT에 걸쳐있기 때문이다. 따라서 SYSTEM VIEW를 이용해서 대상을 찾아야 한다.

더 나아가, SYSTEM VIEW는 OBJECT의 정보를 가지고 있으니, 이를 이용해서 COMPILE 구문을 쉽게 생성할 수 있다. INVALID된 OBJECT가 많을수록 다 클릭하고 있을 수는 없으니, **다이나믹하게 쿼리를 생성하여 수행하는 것이 훨씬 간편하다.**

​다음은 다양한 OBJECT들에 대해 한 번에 조회하고 컴파일 할 수 있는 쿼리를 생성해 주는 쿼리이다.

```SQL
SELECT
'alter '||DECODE(object_type,'PACKAGE BODY', 'PACKAGE',object_type)||' '
||owner||'.'||object_name||'compile'||DECODE(object_type,'PACKAGE BODY',' body;',';')
FROM DBA_OBJECTS
WHERE object_type in ('VIEW', 'FUNCTION', 'PACKAGE', 'PACKAGE BODY', 'PROCEDURE', 'TRIGGER', 'SYNONYM')
AND status = 'INVALID';
```
* ＊주의사항
    - DBA_OBJECTS라는 SYSTEM VIEW를 사용하므로, DBA_OBJECTS를 조회할 수 없는 경우엔 USER_OBJECTS를 사용하면 된다.

    - 단 USER_OBJECTS는 현재 로그인한 USER의 OBJECT만 보여준다. DEPENDENCY가 타 계정까지 연결되어 INVALID된 경우, 다른 계정에서 다시 찾아야 한다.

​