---
lab:
  title: 랩 8 – 차단 문제 식별 및 해결
  module: Optimize query performance in Azure SQL
---

# 차단 문제 식별 및 해결

**예상 소요 시간:** 15분

학생들은 수업에서 얻은 정보를 가져와 AdventureWorks 내에서 디지털 변환 프로젝트에 대한 결과물을 범위로 지정합니다. 학생들은 Azure Portal 및 기타 도구를 검토하여 네이티브 도구를 활용하여 성능 관련 문제를 식별하고 해결하는 방법을 결정합니다. 마지막으로 학생들은 차단 문제를 적절하게 식별하고 해결할 수 있습니다.

데이터베이스 관리자는 성능 관련 문제를 식별하고 발견된 문제를 해결하는 실행 가능한 솔루션을 제공하도록 고용됩니다. 성능 문제를 조사하고 해결 방법을 제안해야 합니다.

**참고:** 이 연습을 진행할 때는 T-SQL 코드를 복사하여 붙여넣어야 합니다. 코드를 실행하기 전에 코드를 올바르게 복사했는지 확인하세요.

## 데이터베이스 복원

1. **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak**에 있는 데이터베이스 백업 파일을 랩 가상 머신의 **C:\LabFiles\Monitor and optimize** 경로에 다운로드합니다(폴더 구조가 없는 경우 새로 만들기).

    ![그림 03](../images/dp-300-module-07-lab-03.png)

1. Windows 시작 단추를 선택하고 SSMS를 입력합니다. 목록에서 **Microsoft SQL Server Management Studio 18**을 선택합니다.  

    ![그림 01](../images/dp-300-module-01-lab-34.png)

1. SSMS가 열리면 **서버에 연결** 대화 상자가 기본 인스턴스 이름으로 미리 채워집니다. **연결**을 선택합니다.

    ![그림 02](../images/dp-300-module-07-lab-01.png)

1. **데이터베이스** 폴더를 선택한 다음, **새 쿼리**를 선택합니다.

    ![그림 03](../images/dp-300-module-07-lab-04.png)

1. 새 쿼리 창에서 아래 T-SQL을 복사하여 붙여넣습니다. 쿼리를 실행하여 데이터베이스를 복원합니다.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017_log.ldf';
    ```

    **참고:** 데이터베이스 백업 파일 이름 및 경로는 1단계에서 다운로드한 것과 일치해야 합니다. 일치하지 않으면 명령이 실패합니다.

1. 복원이 완료되면 성공 메시지가 표시되어야 합니다.

    ![그림 03](../images/dp-300-module-07-lab-05.png)

## 차단된 쿼리 실행 보고서

1. **새 쿼리**를 선택합니다. 다음 T-SQL 코드를 복사하여 쿼리 창에 붙여넣습니다. 이 쿼리를 실행하려면 **실행**을 선택합니다.

    ```sql
    USE MASTER

    GO

    CREATE EVENT SESSION [Blocking] ON SERVER 
    ADD EVENT sqlserver.blocked_process_report(
    ACTION(sqlserver.client_app_name,sqlserver.client_hostname,sqlserver.database_id,sqlserver.database_name,sqlserver.nt_username,sqlserver.session_id,sqlserver.sql_text,sqlserver.username))
    ADD TARGET package0.ring_buffer
    WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS, MAX_DISPATCH_LATENCY=30 SECONDS, MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE, TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)
    GO

    -- Start the event session 
    ALTER EVENT SESSION [Blocking] ON SERVER 
    STATE = start; 
    GO
    ```

    위의 T-SQL 코드는 차단 이벤트를 캡처하는 확장 이벤트 세션을 생성합니다. 데이터에는 다음과 같은 요소가 포함되어 있습니다.

    - 클라이언트 애플리케이션 이름
    - 클라이언트 호스트 이름
    - 데이터베이스 ID
    - 데이터베이스 이름
    - NT 사용자 이름
    - 세션 ID
    - T-SQL 텍스트
    - 사용자 이름

1. **새 쿼리**를 선택합니다. 다음 T-SQL 코드를 복사하여 쿼리 창에 붙여넣습니다. 이 쿼리를 실행하려면 **실행**을 선택합니다.

    ```sql
    EXEC sys.sp_configure N'show advanced options', 1
    RECONFIGURE WITH OVERRIDE;
    GO
    EXEC sp_configure 'blocked process threshold (s)', 60
    RECONFIGURE WITH OVERRIDE;
    GO
    ```

    **참고:** 위의 명령은 차단된 프로세스 보고서가 생성되는 임계값(초)을 지정합니다. 결과적으로 이 단원에서 *blocked_process_report*가 발생할 때까지 기다릴 필요가 없습니다.

1. **새 쿼리**를 선택합니다. 다음 T-SQL 코드를 복사하여 쿼리 창에 붙여넣습니다. 이 쿼리를 실행하려면 **실행**을 선택합니다.

    ```sql
    USE AdventureWorks2017
    GO

    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;

    GO
    ```

1. **새 쿼리** 단추를 선택하여 또 다른 쿼리 창을 엽니다. 다음 T-SQL 코드를 복사하여 새 쿼리 창에 붙여넣습니다. 이 쿼리를 실행하려면 **실행**을 선택합니다.

    ```sql
    USE AdventureWorks2017
    GO

    SELECT TOP (1000) [LastName]
      ,[FirstName]
      ,[Title]
    FROM Person.Person
    WHERE FirstName = 'David'
    ```

    **참고:** 이 쿼리는 결과를 반환하지 않으며 무기한으로 실행되는 것처럼 보입니다.

1. **개체 탐색기**에서 **관리** -> **확장 이벤트** -> **세션**을 확장합니다.

    방금 만든 *Blocking*이라는 확장 이벤트가 목록에 있습니다.

    ![그림 01](../images/dp-300-module-08-lab-01.png)

1. **package0.ring_buffer**를 마우스 오른쪽 단추로 클릭하고 **대상 데이터 보기**를 선택합니다.

    ![그림 02](../images/dp-300-module-08-lab-02.png)

1. 하이퍼링크를 선택합니다.

    ![그림 03](../images/dp-300-module-08-lab-03.png)

1. XML은 차단 중인 프로세스와 차단을 초래하는 프로세스를 표시합니다. 시스템 정보뿐 아니라 이 프로세스에서 실행되는 쿼리를 볼 수 있습니다.

    ![그림 04](../images/dp-300-module-08-lab-04.png)

1. 또는 아래 쿼리를 실행하여 *session_id*당 차단된 세션 ID 목록을 포함하여 다른 세션을 차단하는 세션을 식별할 수 있습니다.

    ```sql
    WITH cteBL (session_id, blocking_these) AS 
    (SELECT s.session_id, blocking_these = x.blocking_these FROM sys.dm_exec_sessions s 
    CROSS APPLY    (SELECT isnull(convert(varchar(6), er.session_id),'') + ', '  
                    FROM sys.dm_exec_requests as er
                    WHERE er.blocking_session_id = isnull(s.session_id ,0)
                    AND er.blocking_session_id <> 0
                    FOR XML PATH('') ) AS x (blocking_these)
    )
    SELECT s.session_id, blocked_by = r.blocking_session_id, bl.blocking_these
    , batch_text = t.text, input_buffer = ib.event_info, * 
    FROM sys.dm_exec_sessions s 
    LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
    INNER JOIN cteBL as bl on s.session_id = bl.session_id
    OUTER APPLY sys.dm_exec_sql_text (r.sql_handle) t
    OUTER APPLY sys.dm_exec_input_buffer(s.session_id, NULL) AS ib
    WHERE blocking_these is not null or r.blocking_session_id > 0
    ORDER BY len(bl.blocking_these) desc, r.blocking_session_id desc, r.session_id;
    ```

    ![그림 05](../images/dp-300-module-08-lab-05.png)

1. **Blocking**이라는 확장 이벤트를 마우스 오른쪽 단추로 클릭한 다음 **세션 중지**를 선택합니다.

    ![그림 06](../images/dp-300-module-08-lab-06.png)

1. 차단을 유발하는 쿼리 세션으로 다시 이동하고 쿼리 아래 줄에 `ROLLBACK TRANSACTION`을 입력합니다. `ROLLBACK TRANSACTION`을 강조 표시하고 **실행**을 선택합니다.

    ![그림 07](../images/dp-300-module-08-lab-07.png)

1. 차단된 쿼리 세션으로 다시 이동합니다. 이제 쿼리가 완료된 것을 확인할 수 있습니다.

    ![그림 08](../images/dp-300-module-08-lab-08.png)

## 커밋된 읽기 스냅샷 격리 수준을 사용으로 설정

1. SQL Server Management Studio에서 **새 쿼리**를 선택합니다. 다음 T-SQL 코드를 복사하여 쿼리 창에 붙여넣습니다. **실행** 단추를 선택하여 이 쿼리를 실행합니다.

    ```sql
    USE master
    GO
    
    ALTER DATABASE AdventureWorks2017 SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE;
    GO
    ```

1. 새 쿼리 편집기에서 차단을 유발하는 쿼리를 다시 실행합니다.

    ```sql
    USE AdventureWorks2017
    GO
    
    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;
    GO
    ```

1. 새 쿼리 편집기에서 차단된 쿼리를 다시 실행합니다.

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT TOP (1000) [LastName]
     ,[FirstName]
     ,[Title]
    FROM Person.Person
    WHERE firstname = 'David'
    ```

    ![그림 09](../images/dp-300-module-08-lab-09.png)

    이전 작업에서 update 문에 의해 차단된 반면 동일한 쿼리가 완료되는 이유는 무엇인가요?

    커밋된 읽기 스냅샷 격리 수준은 낙관적 형태의 트랜잭션 격리이며 마지막 쿼리는 차단되지 않고 최근 커밋된 버전의 데이터를 보여 줍니다.

이 연습에서는 차단되는 세션을 식별하고 이러한 시나리오를 완화하는 방법을 알아보았습니다.
