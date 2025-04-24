---
lab:
  title: 랩 7 - 조각화 문제 감지 및 수정
  module: Monitor and optimize operational resources in Azure SQL
---

# 조각화 문제 감지 및 수정

**예상 소요 시간:** 20분

학생들은 단원에서 파악한 정보를 사용하여 AdventureWorks 내에서 진행되는 디지털 혁신 프로젝트의 결과물을 확인합니다. Azure Portal과 다른 도구를 살펴보며, 학생은 기본 도구를 활용하여 성능 관련 문제를 식별하고 해결하는 방법을 결정합니다. 마지막으로 학생은 데이터베이스 내에서 조각화를 식별하고 적절하게 해결하는 단계를 배울 수 있습니다.

데이터베이스 관리자는 성능 관련 문제를 식별하고 발견된 문제를 해결하는 실행 가능한 솔루션을 제공하도록 고용됩니다. AdventureWorks는 10년 이상 소비자 및 유통업체에 자전거와 자전거 부품을 직접 판매해 왔습니다. 최근 이 회사는 고객의 요청을 서비스하는 데 사용되는 제품의 성능 저하를 발견했습니다. SQL 도구를 사용하여 성능 문제를 식별한 다음 해결하는 방법을 제안해야 합니다.

> &#128221; 이 연습을 진행할 때는 T-SQL 코드를 복사하여 붙여넣어야 합니다. 코드를 실행하기 전에 코드를 올바르게 복사했는지 확인하세요.

## 환경을 설정합니다.

랩 가상 머신이 제공되고 미리 구성된 경우 **C:\LabFiles** 폴더에서 랩 파일을 준비해야 합니다. *파일이 이미 있는지 잠시 확인하려면 이 섹션을 건너뜁니다.* 그러나 사용자 개인의 컴퓨터를 사용 중이거나 랩 파일이 없는 경우 계속하려면 *GitHub*에서 복제해야 합니다.

1. 랩 가상 머신 또는 로컬 컴퓨터에 제공되지 않은 경우 Visual Studio Code 세션을 시작합니다.

1. 명령 팔레트(Ctrl+Shift+P)를 열고 **Git: Clone**을 입력합니다. **Git: Clone** 옵션을 선택합니다.

1. 다음 URL을 **리포지토리 URL** 필드에 붙여넣고 **Enter** 키를 선택합니다.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 **C:\LabFiles** 폴더에 저장합니다(없는 경우 폴더 만들기).

---

## 데이터베이스 복원

**AdventureWorks2017** 데이터베이스가 이미 복원된 경우 이 섹션을 건너뛸 수 있습니다.

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 SSMS(SQL Server Management Studio) 세션을 시작합니다.

1. SSMS가 열리면 기본적으로 **서버에 연결** 대화 상자가 나타납니다. 기본 인스턴스를 선택하고 **연결**을 선택합니다. **신뢰할 수 있는 서버 인증서** 확인란을 선택해야 할 수도 있습니다.

    > &#128221; 사용자 고유의 SQL Server 인스턴스를 사용하는 경우 적절한 서버 인스턴스 이름 및 자격 증명을 사용하여 연결해야 합니다.

1. **데이터베이스** 폴더를 선택한 다음, **새 쿼리**를 선택합니다.

1. 새 쿼리 창에서 아래 T-SQL을 복사하여 붙여넣습니다. 쿼리를 실행하여 데이터베이스를 복원합니다.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Shared\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\AdventureWorks2017_log.ldf';
    ```

    > &#128221; **C:\LabFiles**라는 이름의 폴더가 있어야 합니다. 이 폴더가 없는 경우 폴더를 만들거나 데이터베이스 및 백업 파일의 다른 위치를 지정합니다.

1. **메시지** 탭 아래에 데이터베이스가 성공적으로 복원되었음을 나타내는 메시지가 표시됩니다.

## 인덱스 조각화 조사

1. **새 쿼리**를 선택합니다. 다음 T-SQL 코드를 복사하여 쿼리 창에 붙여넣습니다. 이 쿼리를 실행하려면 **실행**을 선택합니다.

    ```sql
    USE AdventureWorks2017

    GO
    
    SELECT i.name Index_Name
     , avg_fragmentation_in_percent
     , db_name(database_id)
     , i.object_id
     , i.index_id
     , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
     INNER JOIN sys.indexes i ON ps.object_id = i.object_id 
     AND ps.index_id = i.index_id
    WHERE avg_fragmentation_in_percent > 50 -- find indexes where fragmentation is greater than 50%
    ```

    이 쿼리를 통해서는 조각화가 **50%** 를 초과하는 인덱스를 보고합니다. 쿼리는 결과를 반환하지 않아야 합니다.

1. 인덱스 조각화는 다음을 비롯한 여러 가지 요인으로 인해 발생할 수 있습니다.

    - 테이블 또는 인덱스를 자주 업데이트합니다.
    - 테이블 또는 인덱스에 자주 삽입하거나 삭제합니다.
    - 페이지 분할.

    Person.Address 테이블과 해당 인덱스의 조각화 수준을 높이려면 다수의 레코드를 삽입하고 삭제합니다. 이를 수행하려면 다음 쿼리를 실행합니다.

    **새 쿼리**를 선택합니다. 다음 T-SQL 코드를 복사하여 쿼리 창에 붙여넣습니다. 이 쿼리를 실행하려면 **실행**을 선택합니다.

    ```sql
    USE AdventureWorks2017

    GO
    
    -- Insert 60000 records into the Address table    

    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Split Avenue ' + CAST(v1.number AS VARCHAR(10)), 
        'Apt ' + CAST(v2.number AS VARCHAR(10)), 
        'PageSplitTown', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '88' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure unique rowguid
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 300 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;
    GO
    
    -- DELETE 25000 records from the Address table
    DELETE FROM [Person].[Address] WHERE AddressID BETWEEN 35001 AND 60000;

    GO

    -- Insert 40000 records into the Address table
    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Fragmented Street ' + CAST(v1.number AS VARCHAR(10)), 
        'Suite ' + CAST(v2.number AS VARCHAR(10)), 
        'FragmentCity', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '99' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure a unique rowguid per row
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 200 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;

    GO
    ```

    이 쿼리에서는 다수의 새 레코드를 추가하고 삭제하여 Person.Address 테이블과 해당 인덱스의 조각화 수준을 늘립니다.

1. 첫 번째 쿼리를 다시 실행합니다. 이제 크게 조각난 4개의 인덱스를 볼 수 있습니다.

1. **새 쿼리**를 선택하고 다음 T-SQL 코드를 복사하여 새 쿼리 창에 붙여넣습니다. 이 쿼리를 실행하려면 **실행**을 선택합니다.

    ```sql
    SET STATISTICS IO,TIME ON

    GO
        
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

    SQL Server Management Studio의 결과 창에서 **메시지** 탭을 선택합니다. **주소** 테이블에 쿼리에서 수행하는 논리 읽기 수를 기록해 둡니다.

## 조각화된 인덱스 다시 빌드

1. **새 쿼리**를 선택하고 다음 T-SQL 코드를 복사하여 새 쿼리 창에 붙여넣습니다. 이 쿼리를 실행하려면 **실행**을 선택합니다.

    ```sql
    USE AdventureWorks2017

    GO
    
    ALTER INDEX [IX_Address_StateProvinceID] ON [Person].[Address] REBUILD PARTITION = ALL 
    WITH (PAD_INDEX = OFF, 
        STATISTICS_NORECOMPUTE = OFF, 
        SORT_IN_TEMPDB = OFF, 
        IGNORE_DUP_KEY = OFF, 
        ONLINE = OFF, 
        ALLOW_ROW_LOCKS = ON, 
        ALLOW_PAGE_LOCKS = ON)
    ```

1. **새 쿼리**를 선택하고 다음 쿼리를 실행하여 **IX_Address_StateProvinceID** 인덱스에 더 이상 50%를 초과하는 조각화가 없는지 확인합니다.

    ```sql
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT i.name Index_Name
        , avg_fragmentation_in_percent
        , db_name(database_id)
        , i.object_id
        , i.index_id
        , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
        INNER JOIN sys.indexes i ON (ps.object_id = i.object_id AND ps.index_id = i.index_id)
    WHERE i.name = 'IX_Address_StateProvinceID'
    ```

    결과를 비교하면 **IX_Address_StateProvinceI** 조각화가 88%에서 0으로 감소한 것을 볼 수 있습니다.

1. 이전 섹션의 select 문을 다시 실행합니다. Management Studio에 있는 **결과** 창의 **메시지** 탭에서 논리 읽기 수를 기록합니다. *주소 테이블에 대한 인덱스를 다시 빌드하기 전에 발생한 논리 읽기 수가 변경되었나요*?

    ```sql
    SET STATISTICS IO,TIME ON

    GO
        
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

인덱스를 다시 빌드했으므로 이제 효율성이 극대화되고 논리 읽기가 감소되어야 합니다. 이제 인덱스 유지 관리가 쿼리 성능에 영향을 미칠 수 있음을 확인했습니다.

---

## 정리

데이터베이스 또는 랩 파일을 다른 용도로 사용하지 않는 경우 이 랩에서 만든 개체를 정리할 수 있습니다.

### C:\LabFiles 폴더 삭제

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 **파일 탐색기**를 엽니다.
1. **C:\\**로 이동합니다.
1. **C:\LabFiles** 폴더를 삭제합니다.

### AdventureWorks2017 데이터베이스 삭제

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 SSMS(SQL Server Management Studio) 세션을 시작합니다.
1. SSMS가 열리면 기본적으로 **서버에 연결** 대화 상자가 나타납니다. 기본 인스턴스를 선택하고 **연결**을 선택합니다. **신뢰할 수 있는 서버 인증서** 확인란을 선택해야 할 수도 있습니다.
1. **개체 탐색기**에서 **데이터베이스** 폴더를 확장합니다.
1. **AdventureWorks2017** 데이터베이스를 마우스 오른쪽 단추로 클릭하고 **삭제**를 선택합니다.
1. **개체 삭제** 대화 상자에서 **기존 연결 닫기** 확인란을 선택합니다.
1. **확인**을 선택합니다.

---

이 랩을 성공적으로 완료하셨습니다.

이 연습에서는 인덱스를 다시 작성하고 논리적 읽기를 분석하여 쿼리 성능을 향상시키는 방법을 알아보았습니다.
