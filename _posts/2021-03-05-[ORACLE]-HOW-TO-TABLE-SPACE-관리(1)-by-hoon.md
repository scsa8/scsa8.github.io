---
layout: post
title: "[Oracle] How_To TBS 관리방법(1)"
description: "Oracle Trouble Shooting How to No.2"
categories: 
    - Oracle Database
author: "Dev. Hoon"
tags: ["Dev. Hoon","Oracle"]
---

DB 운영자의 필수 코스. TBS 관리를 알아보자.

## 1. TABLE SPACE(이하 TBS)란? 

간단하게는 DB 데이터를 저장하는 공간이다.

저장된 DB 데이터는 최소단위인 **BLOCK**으로 나타내지며, 이 BLOCK들이 모인 것이 **EXTENT**, EXTENT가 모인 것이 **SEGMENT**, SEGMENT가 모여 있는 것이 **TABLE SPACE**이다.

    DB의 초기 설정에 따라 이 값은 다를 수 있지만, 필자가 운영하는 시스템은 1 BLOCK은 65536 Byte(2^16)이며, 1 EXTENT는 8 BLOCK으로 사용한다. TOTAL TBS는 4000GB정도이며, 이중 60%정도는 항상 차있다.

이 TBS는 논리적인 공간이다. 실제 서버에서는 TBS에 해당하는 물리적인 데이터들이 데이터 파일에 나뉘어 담긴다. 보통은 1개 TBS에 1개 데이터 파일을 사용하지만, TBS가 커질수록 데이터 파일은 늘어날 수 있다.

![file1_data_structure_of_oracle](/assets/images/hoon/20210306/file1_data_structure_of_oracle.gif "data structure of oracle")
----------------------------
oracle document 참조

## 2. TBS 정리는 왜 필요한가?

우리가 사용하는 PC의 저장공간을 생각해보자. 저장공간이 꽉 차면 새로운 데이터를 저장할 수 없다.

마찬가지로, TBS 할당량이 꽉 찬 경우, 그 TBS에는 새로운 Segment를 생성할 수 없는 것은 물론 해당 TBS를 사용하는 Segment에 속하는 테이블 또한 데이터를 입력/수정할 수 없다.

오라클의 경우 다음 에러가 발생한다. 

```sql
ORA-1653: unable to extend table [TableName] by 4096 in tablespace [TableSpaceName] 
```

위 에러가 발생하는데는 여러 이유가 있을 수 있다.
에러를 그대로 해석하면, TBS에서 Extend가 불가능하다고 했다.

그 말인 즉, Extend할 공간이 없어서 불가한 것일 수 있지만, 자동 Extend 가 불가하거나 Object의 MAX Extend에 도달해서 그런 것일 수 있다.

다만, DBA에 의해 잘 관리되고 있는 DB라면 보통 Object의 Max는 Unlimited로 주는경우가 많고 EXTEND에 대한 옵션이 이미 정리되어 있을 것이기에, 대부분의 경우 실제로 **TBS가 부족한 것**이다.
​

## 3. TBS 관리의 시작. 조회

TBS로 인한 에러가 발생했다면, DB 데이터에 입력이 멎어버리므로 **장애사유**가 된다. 따라서 TBS 상태를 주기적으로 체크해야 하며, 필요하다면 자동화하여 일정 수치 이상 차오를시 감지할 수 있도록 해야된다.

이를 위해 다음 쿼리를 사용할 수 있다.

```sql
select a.tablespace_name as tbs,
        round(sum(a.total1)/1024/1024/1024,1) "TotalGB",
        round(sum(a.total1)/1024/1024/1024,1)-round(sum(a.sum1)/1024/1024/1024,1) "UsedGB",
        round(sum(a.sum1)/1024/1024/1024,1) "FreeGB",
        round((round(sum(a.total1)/1024/1024/1024,1)-round(sum(a.sum1)/1024/1024/1024,1))/round(sum(a.total1)/1024/1024/1024,1)*100,2) "Used%"
from
    (select tablespace_name,0 total1,sum(bytes) sum1,max(bytes) MAXB,count(bytes) cnt
    from dba_free_space
    where tablespace_name like '%' -- 사용하는 TBS 이름을 넣는다.
    group by tablespace_name
    union
    select tablespace_name,sum(bytes) total1,0,0,0
    from dba_data_files
    where tablespace_name like '%' -- 사용하는 TBS 이름을 넣는다.
    group by tablespace_name) a
group by a.tablespace_name
order by tbs;
```

위 쿼리를 사용시, TBS별로 전체 사이즈 (GB 기준), 사용량, 잔여량, 사용 %까지 깔끔하게 확인할 수 있다.

> 주의사항

- DBA 시스템 VIEW를 사용하기에 권한이 없는 경우 조회할 수 없다. 권한이 없다면 USER SYSTEM VIEW를 사용할것.
  
    

