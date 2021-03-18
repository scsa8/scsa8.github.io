---
layout: post
title: "[Oracle] How_To TBS 관리방법(2)"
description: "Oracle Trouble Shooting How to No.3"
categories: 
    - Oracle Database
author: "Dev. Hoon"
tags: ["Dev. Hoon","Oracle"]
---

    

지난 POST에서는 TABLE SPACE가 무엇인지 간단히 보았고, 조회방법을 소개했다. 이제는 문제가된 TABLE SPACE를 처리할 차례이다.

-------------------


# TBS 정리방법
주로 쓰는 방법은 크게 3가지이다.
1. 공간을 추가 할당하거나
2. 안쓰는 객체를 지워 공간을 반환하거나
3. 객체들을 REBUILD 하는 것이다.

--------------------
 ​

## 1. 공간을 추가로 할당하는 방법
> PC를 사용하다 용량이 부족하면 HDD/SSD를 업그레이드 하거나 증설한다. DB도 마찬가지이다.

TBS는 결국 **DATAFILE**이라는 물리적 공간에 기록되는 데이터의 크기를 **논리적으로 제한**하는 것이다.
이를 처리하는 방법으로 크게 3가지 방법이 있다.  
<br>

 ### 0. 현재 사용하는 DATA FILE을 확인하자.


```sql
select * from dba_data_files;
```
위 쿼리를 수행하면 현재 사용하는 DATA FILE들의 정보를 얻을 수 있는데, 그중 일부를 보면 다음과 같다.

FILE_NAME|TABLESPACE_NAME|BYTES|BLOCKS|STATUS
---|---|---|---|---
+DATA/FEWN1POD/B1911729F0C7A5DEE0530918000AC5BD/DATAFILE/system.840.1053708115|SYSTEM|745537536|91008|AVAILABLE
+DATA/FEWN1POD/B1911729F0C7A5DEE0530918000AC5BD/DATAFILE/sysaux.768.1053708115|SYSAUX|4965269504|606112|AVAILABLE

FILE_NAME은 결국 이 FILE이 어디에 있는 어떤 파일인지를 알려준다. TABLE SPACE NAME은 이 파일이 어떤 TBS에 속해있는지를 보여주며, 이 파일의 사이즈도 친절하게 BYTES와 BLOCKS로 알려준다.


### 1. DATA FILE의 사이즈 변경

```sql
ALTER DATABASE DATAFILE '[DataFile Path & Name]' RESIZE [Size];
```
사용중인 DATAFILE의 사이즈를 변경해준다.   
DataFile Path & Name은 0번에서 수행한 쿼리에 나오는 FileName을 넣어주면 된다. 
위 FILENAME은 오라클 CLOUD이기 때문에 생소한 주소이지만, 일반적으로는 **'C:\ORACLE\ORADATA\ORACLE\SYSTEM01.DBF'** 이런식이다.  
사이즈는 기본 BYTE 단위이며, M, K 등을 붙이면 MB, KB로 인식한다. 500M이라 적으면 500MB로 변경될 것이다.

### 2. DATA FILE의 최대치 늘리기 & 자동으로 늘어나게 하기

```sql
ALTER DATABASE DATAFILE '[DataFile Path & Name]'  AUTOEXTEND ON MAXSIZE UNLIMITED;
```
앞서 Table Space는 논리적인 공간이라고 말했다. 이렇게 할경우, 논리적인 제한이 없어진다. 이제 DATA FILE은 DATAFILE이 있는 운영체제와 DATAFILE이 허용하는 한계까지 쭉 자동으로 늘어난다. 야호!  
다만...진짜 자동으로 끝까지 늘어나버렸다가는 곤란할 수 있다. 서버의 스토리지에는 DATAFILE 뿐만 아니라 rollback이나 undo 영역, temp TBS도 같이 저장되어 있기 때문에...서버 스토리지를 데이터 파일이 다 먹으면 문제가 생길 수 있다. 그래서 보통은 이렇게 하면 안된다. 
> 참고로, 0번의 쿼리를 수행하면 해당 DataFile이 AutoExtend가 가능한지, 최대치는 얼마인지, 한번에 얼마씩 늘리는지 등을 다 볼 수 있다. 그대로 써져있으니 어렵지 않게 볼수 있다.  


### 3. DATA FILE 추가 

```sql
ALTER TABLESPACE [TableSpaceName] ADD DATAFILE '[DataFile Path & Name]' SIZE [Size];
```
넣을 PATH는 기존 DATAFILE과 동일한 PATH를 써주면 되고, NAME은 새로 적당히 하나 만들어주면 된다. ORACLE이 새로운 DATA FILE을 생성하고, TBS에 할당한다.
여유 리소스가 있을경우, 데이터파일을 해당 TBS에 할당해주면 되겠다.

물론, 시스템 운영과 인프라 운영이 분리되어있다면, 바로 가능한 방법은 아니다. 무수한 결재와 작업 대기가 필요할테니 사전에 미리미리 준비가 필요하다. 그리고 운영 환경에서 **리소스가 여유로운건 드문일**이다. 부족할때마다 늘리려고 한다면 이미 인프라는 포화상태일 것이다. 보통은 아래 방법을 많이 쓰게 될 것이다.


--------------------

## 2) 사용하지 않는 테이블 TRUNCATE, DROP

공간을 할당하는 것보다는 현실적이다.  
의외로, TBS를 차지하는 object들을 보다보면, **'왜 이런게 있지?'** 라는 의문이 드는 객체들이 있다. 모종의 이유로 생성되었다가 특정한 이유가 있어서 삭제하지 않았거나 관리되지 않는 객체들이다. 이들을 발견한다면 TRUNCATE와 DROP으로 할당된 TBS를 회수해주자.

특히 작업중 **데이터 백업을 위해 백업 테이블을 생성하는 방식**으로 작업해왔다면, 이런 객체들이 먹는 TBS가 생각보다 크다.

```sql
TRUNCATE TABLE [TableName]; -- TRUNCATE할경우, 연결된 INDEX도 같이 TRUNCATE된다.
DROP TABLE [TableName]; -- TABLE의 경우
DROP INDEX [IndexName]; -- INDEX의 경우
```
가장 깔끔하고 간단한 방법이지만,  **DEPENDENCY**를 생각하면 그렇지 않다.

분명 안쓰는 테이블이었는데 DEPENDENCY를 조회했을때 어딘가와 연결되어있다면? 별게 아니라고 판단하고 날려버렸는데 문제가 생긴다면?  
줄줄이 **INVALID**가 날것이고 DB OBJECT들은 하나씩 뻗어버릴 것이며, 당신은 사유서를 써야할 것이다...
> 사실, 오라클은 delete와 drop은 복구가 가능하다. 그러나 truncate는 무슨수를 써도 복구가 안된다. backup server의 backup된 이미지를 내려받는 것 외엔 불가능하다...  
> backup 이미지를 요청하는 순간 이미 장애가 난것이나 다름없으니 truncate 사용은 항상 신중의 신중을 기하도록 하자.

--------------------


## 3) 객체를 REORG 하는 것 
실제로 DB 운영자가 TBS 부족시 가장 많이 하는 작업이다.  
객체의 REORG는 결국 **만들어진 객체들의 공간을 처음부터 다시 계산해서 만드는 것**이다. DROP후 재생성하거나, TRUNCATE 한 후에 데이터를 다시 넣는 등 object를 아예 다시 만드는 것과 동일하지만, 당연히 훨씬 안전하다.

객체의 REBUILD는 다음 코드를 사용한다. (TBS를 차지하는 문제에 대해 얘기중이니, TABLE과 INDEX만 보자)
```sql
ALTER TABLE [TableName] MOVE TABLESPACE [TableSpaceName]; -- Table의 경우
ALTER INDEX [IndexName] REBUILD TABLESPACE [TableSpaceName]; -- Index의 경우
```
[TableSpaceName]에 옮기고자 하는 TBS 이름을 넣어주면 재생성 하며 그 TBS로 옮길 수도 있다.

​

그런데, 단순히 이렇게는 작업하면 <text style='color:red; font-size:20px'>절대 안된다.</text>  
 구문이 작동 안할수도 있고 심각한 성능장애가 발생할 수 있다.

​

- 위 코드에서 **PARTITION과 SUBPARTITION은 고려되지 않았다.**

만약 partition (또는 subpartition)된 TABLE/INDEX일 경우, 나누어진 partition들을 일일이 옮겨주어야 한다. 

따라서,  PART_A, PART_B 라는 두개 PARTITION으로 구성되었다고 한다면 아래와 같이 적어야 한다.
```sql
ALTER TABLE [TableName] MOVE PARTITION PART_A TABLESPACE [TableSpaceName]; 
ALTER TABLE [TableName] MOVE PARTITION PART_B TABLESPACE [TableSpaceName]; 
ALTER INDEX [IndexName] REBUILD PARTITION PART_A TABLESPACE [TableSpaceName]; 
ALTER INDEX [IndexName] REBUILD PARTITION PART_B TABLESPACE [TableSpaceName]; 
```

PARTITION된 TABLE에 대해 PARTITION keyword 없이 TABLE의 TBS 변경은 에러를 발생시킨다. TABLE은 실제로는 TBS를 점유하고 있지 않기 때문이다. 실제 **공간을 점유하고 있는 것은 PARTITION**들이다. TABLE에 DEFAULT TBS는 등록되어있으나, PARTITION이 빈틈없이 되어있다면 실 데이터는 그 PARTITION별로 할당된 TBS에 저장된다. PARTITION된 TABLE의 TABLE TBS는 실 데이터의 TBS가 아닌 PARTITION에 대한 ATTRIBUTE일 뿐이다. 이 ATTRIBUTE를 바꾸고 싶다면 다음과 같이 적는다.

```sql
ALTER TABLE [TableName] MODIFY DEFAULT ATTRIBUTES TABLESPACE [TableSpaceName];
ALTER INDEX [IndexName] MODIFY DEFAULT ATTRIBUTES TABLESPACE [TableSpaceName];
```
어쨋든, PARTITION이나 SUBPARTITION 된 테이블을 REORG하고 싶다면, 개별 PARTITION들에 대한 REBUILD를해주어야한다.

- **LONG과 LOB 컬럼이 포함된 객체는 MOVE/REBUILD 되지 않는다.**

LONG 컬럼과 LOB 컬럼이 있으면 move, rebuild 되지 않는다. 

불행인지 다행인지 필자가 맡고 있는 시스템은 LONG과 LOB이 컬럼이 있는 테이블을 다룰 일이 없었다. 정확히 모르기도하고, 추가로 더 설명하자니 너무 복잡해 잘 설명하신분의 LINK를 덧붙이고 넘어가겠다.

https://m.blog.naver.com/itisksc/30046151502


 

- <text style='color:red; font-size:20px;'> STAISTICS 정보가 어그러져 PLAN이 바뀐다.</text>

막 작업하면 안되는 가장 중요한 이유이다. **OBJECT를 REBUILD하면 통계정보가 싹 사라진다.**

문제는 통계정보와 PLAN은 아주 밀접하다는 것이다. REORG 한번 했다가 1분 걸리던 프로그램이 2시간이 걸리는 수가 있다. 통계정보라는 특성상, 과거의 통계정보와 동일한 정보는 영영 구할수 없을 것이다. 이런 상황에서 REORG한 OBJECT의 DEPENDENCY가 여러군데로 뻗쳐있어 동시다발적으로 성능 문제가 발생한다면? 

이런 문제에 대해 우리가 사용할 수 있는 방법은 두가지이다.

 - 애시당초 본래 통계정보를 백업하고 REORG 후 다시 집어넣는다.

 - 통계정보를 새로 만든다.

통계정보를 새로 만들면 보통은 정상적으로 동작하지만, **기존에 동작하던 방식대로 수행될 것을 보장할 수는 없다.** REORG를 통해 실제로 데이터의 분포가 바뀌거나 하는 케이스(안쓰는 데이터를 실제로 TRUNCATE 한다던가)가 아니라면, **가능하면 기존 통계정보를 백업 하고 REORG 후 그대로 재적재 하는 것**이 장애 가능성을 없애는 올바른 방법이다.

​

결론을 정리하면, REORG 작업을 진행하려면 

1. 객체의 **partition 여부를 고려**하여 쿼리를 작성한다

2. reorg시 추가 작업이 필요한 컬럼(**LOB, LONG**)을 가지고 있는지 확인한다

3. **통계정보에 대한 백업/재적재 시나리오**를 생각한다.

의 단계를 거쳐주는 것이 좋겠다.

추가로, **REORG 작업중에는 해당 테이블을 사용할 수 없다**. BATCH가 도는 시스템이라면 BATCH시간을 반드시 피할것을 추천한다.

​

-----------------------------------------------------------------------------------------------------------------

​

필자의 시스템은 테이블을 날짜 단위로 partition처리하고, 24시간 실시간으로 운영하며, 한개 테이블에 수십개의 dependency가 걸리는 시스템이다.

TBS는 넉넉치 않아서 적어도 2달에 한번씩은 수동으로 REORG를 해줘야하는데, 한 테이블당 수십줄의 partition reorg 쿼리를 치고 통계처리를 하는것이 힘들어 아래 쿼리를 작성했다.

​

OWNER, TABLE_NAME, YYYYMMDD, PARALLEL,PRESERVE_STATS_YN

위 4개 값만 넣으면 다이나믹하게 REORG 쿼리를 만들어준다.
OWNER와 TABLE NAME은 작업할 유저와 테이블 이름을 넣으면 되며, yyyymmdd는 backup 테이블을 만들때 넣기위한 용도니 작업 날짜를 쓰면 되겠다.  
parallel은 빠른 수행을 위해 병렬로 코어를 사용하는 것이며, preserve에 Y를 넣으면 통계정보 백업 및 재적재 쿼리를, N을 넣으면 재생성 쿼리를 뱉어준다.  

System View에서 정보를 가져오고 DBMS_STATS 패키지를 활용하는 것이니, catalogue select 권한은 필요할 수도 있다.

```sql
select table_name as name, partname, ordervalue, '1_TABLE SCRIPTS' as DESC_COL, query
from (
    select table_name, subpartition_name as partname, 1 as ordervalue, 'alter table '||table_name||' move subpartition '||subpartition_name||' tablespace '|| tablespace_name||' parallel '||:parallel_number||';' query
    from DBA_TAB_SUBPARTITIONS
    where table_owner = :owner
    and table_name = :table_name
    union all
    select table_name, partition_name as partname, 2 as ordervalue, 'alter table '||table_name||' move partition '||partition_name||' tablespace '|| tablespace_name||' parallel '||:parallel_number||';'
    from DBA_TAB_PARTITIONS
    where table_owner = :owner
    and table_name = :table_name
    and subpartition_count = 0
    union all
    select table_name, '-' as partname, 3 as ordervalue, 'alter table '||table_name||' move tablespace '|| tablespace_name||' parallel '||:parallel_number||';'
    from dba_tables
    where owner = :owner
    and table_name = :table_name
    and partitioned = 'NO'
)
union all
-- INDEX REORG SCRIPTS
select index_name as name, partname, ordervalue, '2_INDEX SCRIPTS' as DESC_COL, query
from
(
    select index_name, subpartition_name as partname, 1 as ordervalue, 'alter index '||index_name||' rebuild subpartition '||subpartition_name||' tablespace '||tablespace_name||' parallel '||:parallel_number||';' as query
    from DBA_IND_SUBPARTITIONS
    where index_owner = :owner
    and index_name in(
    select index_name
    from DBA_INDEXES
    where owner = :owner
    and table_name = :table_name )
    union all
    select index_name, partition_name as partname, 2 as ordervalue, 'alter index '||index_name||' rebuild partition '||partition_name||' tablespace '||tablespace_name||' parallel '||:parallel_number||';'
    from DBA_IND_PARTITIONS
    where index_owner = :owner
    and index_name in(
    select index_name
    from DBA_INDEXES
    where owner = :owner
    and table_name = :table_name )
    and subpartition_count = 0
    union all
    select index_name, '-' as partname, 3 as ordervalue, 'alter index '||index_name||' rebuild tablespace '||tablespace_name||' parallel '||:parallel_number||';'
    from DBA_INDEXES
    where owner = :owner
    and table_name = :table_name
    and partitioned = 'NO'
)
union all
SELECT name, partname, ordervalue, desc_col, query
from (
    --CREATE STATS BACKUP TABLE
    select 'CREATE STATISTICS BACKUP TABLE' as name, '-' as partname, 0 as ordervalue, '0_STATS SCRIPTS' as DESC_COL, 'Y' as preserve_stats_yn,
    'EXEC DBMS_STATS.CREATE_STAT_TABLE('''||:owner||''', ''STATS_BAK_'||:yyyymmdd||''');' as query
    from dual
    union all
    --TABLE STATS BACKUP SCRIPTS
    select 'BACKUP STATISTICS' as name, '-' as partname, 1 as ordervalue, '0_STATS SCRIPTS' as DESC_COL, 'Y' as preserve_stats_yn,
    'EXEC DBMS_STATS.EXPORT_TABLE_STATS('''||:owner||''', '''||:table_name||''', STATTAB=>''STATS_BAK_'||:yyyymmdd||''');' as query
    from dual
    union all
    --INDEX STATS BACKUP SCRIPTS
    select 'BACKUP STATISTICS' as name, '-' as partname, 2 as ordervalue, '0_STATS SCRIPTS' as DESC_COL, 'Y' as preserve_stats_yn,
    'EXEC DBMS_STATS.EXPORT_INDEX_STATS('''||owner||''', '''||index_name||''', STATTAB=>''STATS_BAK_'||:yyyymmdd||''');' as query
    from DBA_INDEXES
    where table_name = :table_name
    and owner = :owner
    union all
    --TABLE STATS IMPORT SCRIPTS
    select 'IMPORT STATISTICS' as name, '-' as partname, 4 as ordervalue, '3_STATS SCRIPTS' as DESC_COL, 'Y' as preserve_stats_yn,
    'EXEC DBMS_STATS.IMPORT_TABLE_STATS('''||:owner||''', '''||:table_name||''', STATTAB=>''STATS_BAK_'||:yyyymmdd||''');' as query
    from dual
    union all
    --INDEX STATS IMPORT SCRIPTS
    select 'IMPORT STATISTICS' as name, '-' as partname, 5 as ordervalue, '3_STATS SCRIPTS' as DESC_COL, 'Y' as preserve_stats_yn,
    'EXEC DBMS_STATS.IMPORT_INDEX_STATS('''||:owner||''', '''||index_name||''', STATTAB=>''STATS_BAK_'||:yyyymmdd||''');' as query
    from DBA_INDEXES
    where table_name = :table_name
    and owner = :owner
    union all
    --STATS GENERATE SCRIPTS
    select 'GENERATE STATISTICS' as name, '-' as partname, 0 as ordervalue, '3_STATS SCRIPTS' as DESC_COL, 'N' as preserve_stats_yn,
    'EXEC DBMS_STATS.GATHER_TABLE_STATS('''||:owner||''', '''||:table_name||''', ESTIMATE_PERCENT=>1, DEGREE=>'||:parallel_number||', CASCADE=>True);' as query
    from dual
)
where preserve_stats_yn = upper(:preserve_stats_yn)
order by desc_col, ordervalue, name, partname;
```
위 쿼리 수행시, 통계정보 백업테이블을 생성해주는 쿼리는 한번만 수행하도록 하자.


# 마치며
서두에 얘기한대로, TBS를 확보하는 세가지 방법을 알아보았다.  
- DATAFILE을 늘리는 방법
- 미사용 객체를 DROP, TRUNCATE 하는방법
- 객체를 REORG하는 방법

이 방법들은 일반적으로 많이 쓰이지만 위험한 작업이기도 하다. 
왜 위험한지 케이스별로 아는 이유는....누군가 겪어봤기 때문이다. 위의 작업을 준비하고자 하는 분이 있다면 최대한 문제를 겪지 않길 바라며 자세히 적어보았다.  
위 작업을 처음 수행하는 작업자라면, 가능한 시나리오를 미리 그려보고 작업동안  마음을 차분히 가다듬고 진행하기를 바란다. 문제가 생기지 않기를 빌며 새벽에 목욕재계를 하는것도 권장한다.
<br>

> 아래는 추가로 알아볼만한 것들이다.

  - 누군가는 이렇게 생각할 것이다.

"데이터를 입력할 공간이 없다면 데이터를 삭제(DELETE)만 하면 될 것 같은데 굳이 이렇게 해야하는가? "

그렇지 않다. DATA를 DELETE한다고 해서 공간이 새로 생겨나지 않는다. 이를 설명하려면 **block 구조와 HWM 개념에 대한 이해가 필요하다.** 이부분은 추가로 공부해서 (언젠가) 별도 포스트를 적어보겠다.



  - 오라클은 10g 버전부터 SHRINK 기능을 지원한다. 

TBS 차지 양을 줄이고 Table Size를 컴팩트하게 줄여주는 새로운 기능이다. 구문도 **MOVE, REBUILD와 달리 훨씬 간단하고, 무엇보다도 작업이 진행되는 동안 Object가 INAVLID되지 않는다!** (위의 REBUILD는 작업중에 Object를 사용할 수 없다... 운영 시스템의 Rebuild가 위험작업인 이유중 하나)

그러나 개인적으로는 이 기능을 사용하지 않는다. 가장 크리티컬한건, Parallel이 안되는 것이다. 위의 Reorg 쿼리는 parallel을 줄 수 있는 반면 Shrink는 아무 옵션도 줄 수 없다. 이 차이로, 실제 작업시간은 최소 2배~10배 가까이 생긴다. 서버의 코어수가 충분해야 parallel을 사용할 수 있겠지만, 대용량 테이블이라면 무조건 parallel 주는 rebuild가 빠르다.



 - 위에도 얘기했지만 REORG 작업시에는 OBJECT가 INVALID된다.
  
따라서, 현재 그 OBJECT를 참조하는 작업이 돌고 있다면 문제가 생긴다. SESSION이 필요 이상으로 유지되거나 (WAIT이 걸리거나) 에러를 뱉기도 한다. 가능하면 BATCH를 잘 피해서 작업하도록 하자.

​

* 참고자료

http://www.gurubee.net/lecture/1153