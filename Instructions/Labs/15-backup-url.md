---
lab:
  title: 랩 15 - URL로 백업 및 URL에서 복원
  module: Plan and implement a high availability and disaster recovery solution
---

# URL로 백업

**예상 시간:** 30분

AdventureWorks의 DBA는 Azure의 URL에 데이터베이스를 백업하고 사용자 오류가 발생한 후 Azure Blob 스토리지에서 복원해야 합니다.

## 환경을 설정합니다.

랩 가상 머신이 제공되고 미리 구성된 경우 **C:\LabFiles** 폴더에서 랩 파일을 준비해야 합니다. *파일이 이미 있는지 잠시 확인하려면 이 섹션을 건너뜁니다.* 그러나 사용자 개인의 컴퓨터를 사용 중이거나 랩 파일이 없는 경우 계속하려면 *GitHub*에서 복제해야 합니다.

1. 랩 가상 머신 또는 로컬 컴퓨터에 제공되지 않은 경우 Visual Studio Code 세션을 시작합니다.

1. 명령 팔레트(Ctrl+Shift+P)를 열고 **Git: Clone**을 입력합니다. **Git: Clone** 옵션을 선택합니다.

1. 다음 URL을 **리포지토리 URL** 필드에 붙여넣고 **Enter** 키를 선택합니다.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 **C:\LabFiles** 폴더에 저장합니다(없는 경우 폴더 만들기).

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

## URL에 백업 구성

1. 랩 가상 머신 또는 로컬 컴퓨터에 제공되지 않은 경우 Visual Studio Code 세션을 시작합니다.

1. **C:\LabFiles\dp-300-database-administrator**에서 복제된 리포지토리를 엽니다.

1. **Allfiles** 폴더를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기**를 선택합니다. 그러면 올바른 위치에 터미널 창이 열립니다.

1. 터미널에서 다음을 입력하고 **Enter** 키를 누릅니다.

    ```bash
    az login
    ```

1. 브라우저를 열고 코드를 입력하라는 메시지가 표시됩니다. 지침에 따라 Azure 계정에 로그인합니다.

1. *리소스 그룹이 이미 있는 경우 이 단계를 건너뜁니다.* 리소스 그룹이 없는 경우 터미널에서 다음 명령을 실행하여 리소스 그룹을 만듭니다. *contoso-rgXXX######* 을 리소스 그룹의 고유한 이름으로 바꿉니다. 이름은 Azure에서 고유해야 합니다. 위치(-l)를 리소스 그룹의 위치로 바꿉니다.

    ```bash
    az group create -n "contoso-rglod#######" -l eastus2
    ```

    **######** 을 일부 임의 문자로 바꿉니다.

1. 터미널에서 다음을 입력하고 **Enter** 키를 눌러 스토리지 계정을 만듭니다. 스토리지 계정에는 고유한 이름을 사용해야 합니다. *이 이름은 3자에서 24자 사이여야 하고 숫자 및 소문자만 포함할 수 있습니다.* *########* 을 8개의 임의 숫자로 바꿉니다. 이름은 Azure에서 고유해야 합니다. contoso-rgXXX######을 리소스 그룹의 이름으로 바꿉니다. 마지막으로 위치(-l)를 리소스 그룹의 위치로 바꿉니다.

    ```bash
    az storage account create -n "dp300bckupstrg########" -g "contoso-rgXXX########" --kind StorageV2 -l eastus2
    ```

1. 다음으로 후속 단계에서 사용할 스토리지 계정의 키를 가져옵니다. 스토리지 계정 및 리소스 그룹의 고유 이름을 사용하여 터미널에서 다음 코드를 실행합니다.

    ```bash
    az storage account keys list -g contoso-rgXXX######## -n dp300bckupstrg########
    ```

    위 명령의 결과에 계정 키가 있습니다. 이전 명령에서 사용한 것과 동일한 이름(**-n** 뒤에 사용) 및 리소스 그룹(**-g** 뒤에 사용)을 사용해야 합니다. **key1**의 반환 값(큰따옴표 제외)을 복사합니다.

1. SQL Server 데이터베이스를 URL에 백업하는 경우 스토리지 계정 내의 컨테이너가 사용됩니다. 이 단계에서는 구체적으로 백업 스토리지용 컨테이너를 생성합니다. 이렇게 하려면 아래 명령을 실행합니다.

    ```bash
    az storage container create --name "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --fail-on-exist
    ```

    여기서 **dp300bckupstrg########** 은 스토리지 계정을 만들 때 사용된 고유한 스토리지 계정 이름이며 **storage_key**는 이전에 생성된 키입니다. 출력에 **true**가 반환되어야 합니다.

1. 컨테이너 백업이 제대로 만들어졌는지 확인하려면 다음을 실행합니다.

    ```bash
    az storage container list --account-name "dp300bckupstrg########" --account-key "storage_key"
    ```

    여기서 **dp300bckupstrg########** 은 스토리지 계정을 만들 때 사용된 고유한 스토리지 계정 이름이며 **storage_key**는 생성된 키입니다.

1. 보안을 위해 컨테이너 수준의 SAS(공유 액세스 서명)가 필요합니다. 터미널에서 다음 명령을 실행합니다.

    ```bash
    az storage container generate-sas -n "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --permissions "rwdl" --expiry "date_in_the_future" -o tsv
    ```

    여기서 **dp300bckupstrg########** 는 스토리지 계정을 만들 때 사용된 고유한 스토리지 계정 이름이며 **storage_key**는 생성된 키이고 **date_in_the_future**는 지금 이후의 시간입니다. **date_in_the_future**는 UT로 표시되어야 합니다. 예를 들어 **2025-12-31T00:00Z**는 2025년 12월 31일 자정에 만료되는 것을 나타냅니다.

    출력은 다음과 비슷한 내용을 반환해야 합니다. 전체 공유 액세스 서명을 복사하여 **메모장**에 붙여넣으면 다음 작업에서 사용됩니다.

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

## 자격 증명 만들기

이제 기능이 구성되었으므로 Azure Storage 계정에서 백업 파일을 Blob으로 생성할 수 있습니다.

1. **SSMS(SQL Server Management Studio)** 를 시작합니다.

1. SQL Server에 연결하라는 메시지가 표시됩니다. **Windows 인증**이 선택되어 있는지 확인하고 **연결**을 선택합니다.

1. **새 쿼리**를 선택합니다.

1. 다음 Transact-SQL을 사용하여 클라우드의 스토리지에 액세스하는 데 사용할 자격 증명을 생성합니다. 적절한 값을 입력한 다음 **실행**을 선택합니다.

    ```sql
    IF NOT EXISTS  
    (SELECT * 
        FROM sys.credentials  
        WHERE name = 'https://<storage_account_name>.blob.core.windows.net/backups')  
    BEGIN
        CREATE CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]
        WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
        SECRET = '<key_value>'
    END;
    GO  
    ```

    여기서 **<storage_account_name>** 의 두 항목은 만들어진 고유한 스토리지 계정 이름이며 **<key_value>** 는 이전 작업이 끝날 때 다음으로 생성된 값입니다.

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

1. SSMS에서 **보안 -> 자격 증명**으로 이동하여 자격 증명 이 성공적으로 만들어졌는지 확인할 수 있습니다.

1. 자격 증명을 잘못 입력하여 다시 만들어야 하는 경우 다음 명령을 사용하여 자격 증명을 삭제할 수 있습니다. 스토리지 계정 이름을 변경해야 합니다.

    ```sql
    -- Only run this command if you need to go back and recreate the credential! 
    DROP CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]  
    ```

## 데이터베이스를 URL로 백업

1. SSMS를 통해 Transact-SQL로 다음 명령을 사용하여 **AdventureWorks2017** 데이터베이스를 Azure에 백업합니다.

    ```sql
    BACKUP DATABASE AdventureWorks2017   
    TO URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak';
    GO 
    ```

    여기서 **<storage_account_name>** 은 생성된 고유한 스토리지 계정 이름입니다. 

    오류가 발생하면 자격 증명을 만드는 동안 잘못 입력한 것이 없는지, 모두 성공적으로 만들어졌는지 확인합니다.

## Azure CLI를 통해 백업 유효성 검사

파일이 실제로 Azure에 있는지 확인하려면 Storage Explorer(미리 보기) 또는 Azure Cloud Shell을 사용합니다.

1. Visual Studio Code 터미널로 돌아가서 이 Azure CLI 명령을 실행합니다.

    ```bash
    az storage blob list -c "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --output table
    ```

    이전 명령에서 사용한 것과 동일한 고유한 스토리지 계정 이름(**--account-name** 뒤) 및 계정 키(**--account-key** 뒤)를 사용해야 합니다.

    백업 파일이 성공적으로 생성되었는지 확인할 수 있습니다.

## 스토리지 브라우저를 통해 백업 유효성 검사

1. 브라우저 창에서 Azure Portal로 이동하여 **스토리지 계정**을 검색하고 선택합니다.

1. 백업의 생성된 고유한 스토리지 계정 이름을 선택합니다.

1. 왼쪽 탐색에서 **스토리지 브라우저**를 선택합니다. **BLOB 컨테이너** 확장

1. **백업**을 선택합니다.

1. 백업 파일은 컨테이너에 저장됩니다.

## URL에서 복원

이 작업은 Azure Blob 스토리지에서 데이터베이스를 복원하는 방법을 보여 줍니다.

1. **SSMS(SQL Server Management Studio)** 에서 **새 쿼리**를 선택한 다음, 다음 쿼리를 붙여넣고 실행합니다.

    ```sql
    USE AdventureWorks2017;
    GO
    SELECT * FROM Person.Address WHERE AddressId = 1;
    GO
    ```

1. 다음 명령을 실행하여 해당 고객의 주소를 변경합니다.

    ```sql
    UPDATE Person.Address
    SET AddressLine1 = 'This is a human error'
    WHERE AddressId = 1;
    GO
    ```

1. **1단계**를 다시 실행하여 주소가 변경되었는지 확인합니다. 이제 누군가가 WHERE 절을 사용하지 않거나 잘못된 WHERE 절을 사용하여 수천 또는 수백만 개의 행을 변경했다고 가정합니다. 해결 방법 중 하나는 마지막으로 사용 가능한 백업에서 데이터베이스를 복원하는 것입니다.

1. 데이터베이스를 복원하여 고객 이름이 실수로 변경되기 전의 상태로 되돌리려면 다음을 실행합니다.

    > &#128221; **SET SINGLE_USER WITH ROLLBACK IMMEDIATE** 구문을 사용하면 열려 있는 트랜잭션이 모두 롤백됩니다. 이렇게 하면 활성 연결로 인해 복원이 실패하는 것을 방지할 수 있습니다.

    ```sql
    USE [master]
    GO

    ALTER DATABASE AdventureWorks2017 SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    GO

    RESTORE DATABASE AdventureWorks2017 
    FROM URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak'
    GO

    ALTER DATABASE AdventureWorks2017 SET MULTI_USER
    GO
    ```

    여기서 **<storage_account_name>** 은 생성된 고유한 스토리지 계정 이름입니다.

1. **1단계**를 다시 실행하여 고객 이름이 복원되었는지 확인합니다.

Azure Blob 스토리지 서비스에서 백업을 수행하거나 복원하려면 구성 요소 및 상호 작용을 이해하는 것이 중요합니다.

---

## 리소스 정리

다른 용도로 Azure SQL Server를 사용하지 않는 경우 이 랩에서 만든 리소스를 정리할 수 있습니다.

### 리소스 그룹 삭제

이 랩에 대한 새 리소스 그룹을 만든 경우 리소스 그룹을 삭제하여 이 랩에서 만든 모든 리소스를 제거할 수 있습니다.

1. Azure Portal의 왼쪽 탐색 창에서 **리소스 그룹**을 선택하거나 검색 창에서 **리소스 그룹**을 검색하고 결과에서 선택합니다.

1. 이 랩을 위해 만든 리소스 그룹으로 이동합니다. 리소스 그룹에는 Azure SQL Server 및 이 랩에서 만든 기타 리소스가 포함됩니다.

1. 위쪽 메뉴에서 **리소스 그룹 삭제**를 선택합니다.

1. **리소스 그룹 삭제** 대화 상자에서 확인할 리소스 그룹의 이름을 입력한 다음 **삭제**를 선택합니다.

1. 리소스 그룹이 삭제될 때까지 기다립니다.

1. Azure Portal을 닫습니다.

### 랩 리소스만 삭제합니다.

이 랩에 대한 새 리소스 그룹을 만들지 않았고 리소스 그룹과 이전 리소스를 그대로 유지하려는 경우 이 랩에서 만든 리소스를 계속 삭제할 수 있습니다.

1. Azure Portal의 왼쪽 탐색 창에서 **리소스 그룹**을 선택하거나 검색 창에서 **리소스 그룹**을 검색하고 결과에서 선택합니다.

1. 이 랩을 위해 만든 리소스 그룹으로 이동합니다. 리소스 그룹에는 Azure SQL Server 및 이 랩에서 만든 기타 리소스가 포함됩니다.

1. 랩에서 이전에 지정한 SQL Server 이름이 접두사로 지정된 모든 리소스를 선택합니다.

1. 위쪽 메뉴에서 **삭제**를 선택합니다.

1. **리소스 삭제** 대화 상자에서 **삭제**를 입력하고 **삭제**를 선택합니다.

1. 리소스 삭제를 확인하려면 **삭제**를 다시 선택합니다.

1. 리소스가 삭제될 때까지 기다립니다.

1. Azure Portal을 닫습니다.

데이터베이스 또는 랩 파일을 다른 용도로 사용하지 않는 경우 이 랩에서 만든 개체를 정리할 수 있습니다.

### C:\LabFiles 폴더 삭제

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 **파일 탐색기**를 엽니다.
1. **C:\\**로 이동합니다.
1. **C:\LabFiles** 폴더를 삭제합니다.

## AdventureWorks2017 데이터베이스 삭제

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 SSMS(SQL Server Management Studio) 세션을 시작합니다.
1. SSMS가 열리면 기본적으로 **서버에 연결** 대화 상자가 나타납니다. 기본 인스턴스를 선택하고 **연결**을 선택합니다. **신뢰할 수 있는 서버 인증서** 확인란을 선택해야 할 수도 있습니다.
1. **개체 탐색기**에서 **데이터베이스** 폴더를 확장합니다.
1. **AdventureWorks2017** 데이터베이스를 마우스 오른쪽 단추로 클릭하고 **삭제**를 선택합니다.
1. **개체 삭제** 대화 상자에서 **기존 연결 닫기** 확인란을 선택합니다.
1. **확인**을 선택합니다.

---

이 랩을 성공적으로 완료하셨습니다.

이제 Azure의 URL에 데이터베이스를 백업하고 필요한 경우 복원할 수 있음을 확인했습니다.
