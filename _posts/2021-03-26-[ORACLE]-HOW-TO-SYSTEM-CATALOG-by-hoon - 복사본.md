---
layout: post
title: "[Oracle] How_To SYSTEM CATALOG"
description: "Oracle Trouble Shooting How to No.4"
categories: 
    - Oracle Database
author: "Dev. Hoon"
tags: ["Dev. Hoon","Oracle"]
---

오라클에는 유용한 OBJECTS들이 있다. 알면 피가되고 살이되는 OBJECT들이다. 
이전 포스팅에서도 이 OBJECTS들을 이용해 여러 유용한 정보들을 찾아낸 바 있다.
오늘은 유용한 OBJECTS들, 특히 SYSTEM CATALOG에 대해 알아보자.

# ORACLE SYSTEM CATALOGS

<http://www.gurubee.net/lecture/1520>
* DB개발자의 영원한 친구 GURUBEE 사이트의 도움을 받아 작성했습니다. 

## 1. SYSTEM CATALOG 란?

간단하게는 ORACLE의 상태를 나타내는 메타데이터를 의미한다. **SYSTEM CATALOG**나 **DICTIONARY**라고도 부른다. 여기에 있는 대부분의 데이터들이 VIEW이기 때문에, **SYSTEM VIEW**라고도 부른다.
이 CATALOG들은 사용자가 관리하는 데이터가 아니다. SYS 계정에 소속되며, DB의 변경이 있을때 ORACLE이 알아서 데이터를 정리한다. 

## 2. SYSTEM CATALOG의 조회

이 SYSTEM VIEW들은 권한에 따라 조회할 수 있는 것과 조회할 수 없는 것이 있다.
예를들어, DBA% 로 시작하는 VIEW는 모든 CATALOG를 조회할 수 있는 권한이 필요하다.
그 권한은 이렇게 부여한다.
```
grant select_catalog_role to [UserName];
```
> revoke select_catalog_role from [UserName]; 으로 권한을 회수할수 있다.    

권한 부여를 위해 적절한 ADMIN 계정으로 접속해야한다. 정확히는, 유저에게 롤을 부여할 수 있는 권한을 가진 계정이 부여하면 된다. 그런데 보통 ADMIN이 이 권한을 가지고 있을테고 굳이 이 권한을 나눠줄 이유가 없으니, 보통은 ADMIN 계정 접속이 필요하다.

권한이 정상 부여되었다면, 이제 검색하려는 CATALOG의 이름만 알면 조회할 수 있다.

CATALOG는 굉장히 많기 때문에, 이를 다 외울 필요는 없다. 다음 구문을 통해 모든 카탈로그 테이블의 이름을 조회할수 있다.
```sql
select *
from dictionary;
```

현재 필자가 사용하는 환경 (ORACLE CLOUD DB)에서 조회시 3205개의 TABLE_NAME이 나온다.

이걸 다 쓰진 않고, 이중 10~20개 정도면 보통의 운영환경에선 충분하다.

## 3. CATALOG의 분류

크게 정적 VIEW와 동적 VIEW로 나눈다.
정적 VIEW는 DDL등의 작업으로 변동이 일어난 상태를 저장한다.

예를들어, 당신이 테이블을 새로 생성했다면 해당 OBJECT가 카탈로그에 등록될 것이고, 이 정보는 다른 DDL이 일어날떄까지 변하지 않을것이다.

동적 VIEW는 실시간으로 DB의 상태를 보여준다. 가장 대표적인 것이 SESSION 정보이다. SESSION은 사용자의 DML문에도 발생하고, 그 DML문이 실행됨에 따라 정보가 변한다.

- 정적 VIEW
  - DBA% VIEW : DB에 존재하는 모든 정보를 보여준다. 
  - ALL% VIEW : 내가 권한을 가진 모든 정보를 보여준다.
  - USER% VIEW : 내가 접속한 유저의 정보만 보여준다.

예를들어보자.

1. SCOTT 이라는 USER는 EMP라는 테이블을 가지고 있다.
2. SCOTT2 라는 USER는 EMP2라는 테이블을 가지고 있다.
3. ADMIN 이라는 USER는 AD_EMP라는 테이블을 가지고 있다.
4. 이때 SCOTT에게 SCOTT2의 EMP2 테이블을 조회할 수 있는 권한이 부여되었다고 가정하자. 즉,
```sql
-- SCOTT2로 로그인한 상태
grant select on scott2.emp2 to scott;
```
5. 이때 SCOTT 계정에서 DBA_TABLES를 조회한다면 EMP, EMP2, AD_EMP 테이블을 볼 수 있다.
6. SCOTT 계정에서 ALL_TABLES를 조회한다면 EMP, EMP2를 볼 수 있다.
7. SCOTT 계정에서 USER_TABLES를 조회한다면 EMP만 볼 수 있다.

-------------

- 동적 VIEW
  - v$ VIEW : 일반적으로 보게되는 동적 VIEW이다. 현재 INSTANCE에서 볼 수 있는 정보를 보여준다.
  - GV$ VIEW : V$VIEW와 거의 동일하나, INSTANCE를 가리지 않고 보여준다. 
  - x$ VIEW : V\$VIEW, GV_\$VIEW의 원본이라고 한다. 여기까지 볼일은 거의 없다.

> 참고로 대부분 SYNONYM이다. 원본은 이름이 조금씩 다르다.
  
-------------
> 위의 VIEW들을 보면, 비슷한 내용도 조건에 따라 다르게 보임을 알 수 있다. <br/>
> VIEW 라는 것을 생각해보면, VIEW 생성 조건에 이런 조건이 들어갔음을 짐작할 수 있다.<br/>
> 정확히는, USERENV라는 내부 함수를 이용해 현재 접속하는 INSTANCE나 USER 이름을 알아내어 조건으로 사용한다. USERENV('schemaid')를 치면 user 번호인 80을 뱉어주고 이를 조건으로 사용하는 방식.

## 4. 주로 사용하는 CATLAOG VIEW 모음

DBA 테이블이 정보를 다 보여주니 ALL%과 USER%는 생략하겠다. 권한이 없을경우, ALL과 USER로 바꿔서 수행하면 된다.


1. DBA_OBJECTS : 모든 OBJECT들을 보여준다.  
    - status : OBJECT가 사용가능하면 VALID, 아닐경우 INVALID로 표현된다.
    - last_ddl_time : 해당 OBJECT가 마지막으로 DDL된 시간을 알려준다.
2. DBA_TABLES : 모든 TABLE을 보여준다.
    - NUM_ROWS, BLOCKS, LAST_ANALYZED : DBA_OBJECTS에선 볼 수 있는, 이 테이블을 사용하는 이유인 중요한 이유중 하나다. 이 컬럼들을 보면 테이블에 통계정보가 있는지, 언제 생성된건지를 가늠할 수 있다.
    - DML_TIMESTAMP : 가장 최근에 DML이 일어난 시간을 알려준다.
3. DBA_TAB_PARTITIONS, DBA_TAB_SUBPARTITIONS : PARTITION된 테이블이라면 테이블 자체는 사실 깡통이고 그 partition들의 상태를 보아야한다. 그때 사용하는 VIEW. DBA_TABLES VIEW와 요령은 같다.
4. DBA_INDEXES, DBA_IND_PARTITIONS, DBA_IND_SUBPARTITIONS : 위의 TABLE과 같은 맥락으로, INDEX정보를 저장한다.
    - STATUS : 위의 테이블들은 STATUS가 {VALID, INVALID}로 발생하는데, INDEX는 {USABLE, UNUSABLE, N/A}로 발생한다.  
    N/A는 INDEX를 PARTITIONS 했을때나 PARTITONI된 INDEX를 다시 SUBPARTITON했을때 등 내부적으로 다시 쪼개질때 발생하는 값이다.
    - STATUS컬럼 다시 한번 더 강조한다. 보통 INVALID에 대한 체크는 DBA_OBJECTS 테이블로 체크하는데, PARTITION된 INDEX들은 거기서 안잡힌다. 매우중요.
5. DBA_TAB_COLS : 모든 테이블의 컬럼을 보여준다.
6. DBA_SEGMENTS : SEGMENT는 OBJECT를 구성하는 최소단위라고 보면 되겠다.
    - BYTES : SEGMENTS의 BYTES,말그대로 용량을 알려준다. TBS를 정리할때 타겟을 쉽게 찾을 수 있다.
7. DBA_DEPENDENCIES : Object들 간의 DEPENDENCY를 보여준다. 영향도 파악을 위해 필수적인 VIEW.
    - REFERENCED_NAME : 참조 되는 OBJECT들이다.
    - NAME : 참조 하는 OBJECT이다.
8. DBA_VIEWS : 모든 VIEW를 보여준다. 여기서 중요한점은, SYSTEM VIEW도 VIEW라는 것이다. SYS 계정의 VIEW를 찾으면 VIEW의 이름과 생성 쿼리를 얻을 수 있다.
9. V\$SQL, V\$SQLAREA : 최근에 parsing된 SQL들이 저장되는 공간이다.
    - sql_id : sql의 id값이다. 파싱을 하며 생성된 hash값으로, SQL이 정확히 같으면 같은 sql_id를 가진다.(공백 하나조차 같아야함.)
    - plan_hash_value : 파싱을 하며 생성한 plan의 hashvalue이다. plan은 같은 SQLID라도 통계정보의 변동이나 DDL로 변경될 수 있다. 정확한 plan을 알기위해서는 해당 값을 알아야 한다.
    - buffer_gets, executions, cpu_time, elapsed_time : 가장 기본적인 SQL 성능지표다. buffer는 읽은 block수, excutions는 해당 sql의 수행횟수, cpu_time은 연산시간이고, elapsed_time은 해당 SQL을 수행하는데 소요된 총 시간이다. 이 값들을 보면 어떤 SQL이 문제가 있는지 짐작할 수 있다.
10. V$SESSION : 현재 INSTANCE에 연결된 SESSION을 볼 수 있다. 단, SESSION은 끊어졌다고 바로 삭제되는 것이 아니라 INACTIVE로 남아있으니 유의.
    - STATUS : ACTIVE와 INACTIVE로 나뉘며, 현재 SESSION이 살아있는지를 보여준다.
    - SID, SERIAL# : 개별 SESSION을 구분해주는 KEY라고 볼 수 있다.
    - SQL_ID : SESSION이 잡고있는 SQL_ID를 볼 수 있다. locking SESSION이 있을때 어떤 SQL이 문제인지 확인하려면 이 값을 조건으로 V$SQL 테이블을 조회하면 된다.
    - EVENT : SESSION이 어떤 EVENT를 발생시키는지를 볼 수 있다.
11. V$LOCKED_OBJECTS : 현재 LOCK이된 객체들을 볼 수 있다. 
    - LOCKED_MODE : ORACLE은 4개단계의 LOCK을 제공한다. 어떤 LOCK이냐에 따라 문제가 생길수도 안생길수도 있으니 확인이 필요하다.
12. V$PARAMETER : DB의 PARAMETER들을 볼 수 있다.
    - 성격상, COLUMN값보단 NAME을 주의깊게 봐야한다. 특히, nls% 관련은 자주 사용하게 되며 cpu_count나 thread 등의 대한 정보는 parallel 처리시 필수적으로 확인해야 한다.

# 마치며

꽤 적지 않은 SYSTEM CATALOG에 대해 설명을 마쳤다. 여기서는 주요한 컬럼, 자주 사용하는 컬럼만 몇개 나열하였지만, System Catalog의 컬럼값들을 알면 알수록 운영상 많은 도움을 받을 것이다.

여기 적지 않은 VIEW들도 많으니, 호기심이 생긴다면 찾아보는 것도 좋겠다.