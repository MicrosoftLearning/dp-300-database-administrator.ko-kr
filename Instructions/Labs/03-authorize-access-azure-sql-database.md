---
lab:
  title: 랩 3 - Microsoft Entra ID를 사용하여 Azure SQL Database에 대한 액세스 권한 부여
  module: Implement a Secure Environment for a Database Service
---

# 데이터베이스 인증 및 권한 부여 구성

**예상 시간: 25분**

학생들은 단원에서 파악한 정보를 사용하여 Azure Portal 및 *AdventureWorks* 내에서 보안 기능을 구성한 후 구현합니다.

데이터베이스 환경의 보안을 유지하는 데 도움을 주는 선임 데이터베이스 관리자로 고용되었습니다.

> &#128221; 이러한 연습에서는 T-SQL 코드를 복사하여 붙여넣고 기존 SQL 리소스를 활용하도록 요청합니다. 코드를 실행하기 전에 코드를 올바르게 복사했는지 확인하세요.

## 환경을 설정합니다.

랩 가상 머신이 제공되고 미리 구성된 경우 **C:\LabFiles** 폴더에서 랩 파일을 준비해야 합니다. *파일이 이미 있는지 잠시 확인하려면 이 섹션을 건너뜁니다.* 그러나 사용자 개인의 컴퓨터를 사용 중이거나 랩 파일이 없는 경우 계속하려면 *GitHub*에서 복제해야 합니다.

1. 랩 가상 머신 또는 로컬 컴퓨터에 제공되지 않은 경우 Visual Studio Code 세션을 시작합니다.

1. 명령 팔레트(Ctrl+Shift+P)를 열고 **Git: Clone**을 입력합니다. **Git: Clone** 옵션을 선택합니다.

1. 다음 URL을 **리포지토리 URL** 필드에 붙여넣고 **Enter** 키를 선택합니다.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 **C:\LabFiles** 폴더에 저장합니다(없는 경우 폴더 만들기).

## Azure에서 SQL Server 설정

Azure에 로그인하고 Azure에서 실행 중인 기존 Azure SQL Server 인스턴스가 있는지 확인합니다. *Azure에서 실행 중인 SQL Server 인스턴스가 이미 있는 경우 이 섹션을 건너뜁니다.*

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 Visual Studio Code 세션을 시작하고 이전 섹션에서 복제된 리포지토리로 이동합니다.

1. **/Allfiles/Labs** 폴더를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기**를 선택합니다.

1. Azure CLI를 사용하여 Azure에 연결해 보겠습니다. 다음 명령을 입력하고 **Enter** 키를 선택합니다.

    ```bash
    az login
    ```

    > &#128221; 브라우저 창이 열립니다. Azure 자격 증명을 사용하여 로그인합니다.

1. Azure에 로그인한 후에는 리소스 그룹이 아직 없는 경우 리소스 그룹을 만들고 해당 리소스 그룹 아래에 SQL Server 및 데이터베이스를 만듭니다. 다음 명령을 입력하고 **Enter** 키를 선택합니다. *스크립트를 완료하는 데 몇 분 정도 걸립니다.*

    ```bash
    cd ./Setup
    ./deploy-sql-database.ps1
    ```

    > &#128221; 기본적으로 이 스크립트는 **contoso-rg**라는 리소스 그룹을 만들거나, 있는 경우 *contoso-rg*로 시작하는 리소스를 사용합니다. 기본적으로 **미국 서부 2** 지역(westus2)에도 모든 리소스가 만들어집니다. 마지막으로 **SQL 관리자 암호**에 대한 임의의 12자 암호를 생성합니다. 이러한 값은 고유 값으로 **-rgName**, **-location** 및 **-sqlAdminPw** 매개 변수 중 하나 이상을 사용하여 변경할 수 있습니다. 암호는 12자 이상의 Azure SQL 암호 복잡성 요구 사항을 충족해야 하며 대문자 1개, 소문자 1개, 숫자 1개, 특수 문자 1개 이상을 포함해야 합니다.

    > &#128221; 스크립트는 SQL Server 방화벽 규칙에 현재 공용 IP 주소를 추가합니다.

1. 스크립트가 완료되면 리소스 그룹 이름, SQL Server 이름 및 데이터베이스 이름, 관리자 사용자 이름 및 암호를 반환합니다. 이러한 값은 랩의 뒷부분에서 필요하므로 기록해 두세요.

---

## Microsoft Entra를 사용하여 Azure SQL Database에 대한 액세스 권한 부여

`CREATE USER [anna@contoso.com] FROM EXTERNAL PROVIDER` T-SQL 구문을 사용하여 포함된 데이터베이스 사용자로 Microsoft Entra 계정에서 로그인을 만들 수 있습니다. 포함된 데이터베이스 사용자는 데이터베이스와 연결된 Microsoft Entra 디렉터리의 ID에 매핑되며, `master` 데이터베이스에는 로그인이 없습니다.

Azure SQL Database에 Microsoft Entra 서버 로그인이 도입되면 SQL Database 가상 `master` 데이터베이스의 Microsoft Entra 보안 주체에서 로그인을 만들 수 있습니다. Microsoft Entra *사용자, 그룹 및 서비스 주체*에서 Microsoft Entra 로그인을 만들 수 있습니다. 자세한 내용은 [Microsoft Entra 서버 보안 주체](/azure/azure-sql/database/authentication-azure-ad-logins)를 참조하세요.

또한 Azure Portal을 사용하여 관리자를 만들 수 있으며 Azure 역할 기반 액세스 제어 역할은 Azure SQL Database 논리 서버에 전파되지 않습니다. T-SQL(Transact-SQL)을 사용하여 추가 서버 및 데이터베이스 사용 권한을 부여해야 합니다. SQL Server에 대한 Microsoft Entra 관리자를 만들어 보겠습니다.

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 브라우저 세션을 시작하고 [https://portal.azure.com](https://portal.azure.com/)으로 이동합니다. Azure 자격 증명을 사용하여 Portal에 연결합니다.

1. Azure Portal 홈페이지에서 **SQL Server**를 검색하고 선택합니다.

1. SQL Server **dp300-lab-xxxxxxxx**를 선택합니다. 여기서 *xxxxxxxx* 는 임의의 숫자 문자열입니다.

    > &#128221; 이 랩에서 만들지 않은 사용자 고유의 Azure SQL Server를 사용하는 경우 해당 SQL 서버의 이름을 선택합니다.

1. *개요* 블레이드에서 *Microsoft Entra 관리자* 옆에 있는 **구성되지 않음**을 선택합니다.

1. 다음 화면에서 **관리자 설정**을 선택합니다.

1. **Microsoft Entra ID** 사이드바에서 Azure Portal에 로그인한 Azure 사용자 이름을 검색한 후 **선택**을 클릭합니다.

1. **저장**을 선택하여 프로세스를 완료합니다. 그러면 사용자 이름이 서버의 Microsoft Entra 관리자가 됩니다.

1. 왼쪽에서 **개요**를 선택하고 **서버 이름**을 복사합니다.

1. SSMS(SQL Server Management Studio)를 열고 **연결** > **데이터베이스 엔진**을 선택합니다. **서버 이름**에서 서버 이름을 붙여넣습니다. 인증 유형을 **Microsoft Entra MFA**로 변경합니다.

1. **연결**을 선택합니다.

## 데이터베이스 개체에 대한 액세스 관리

이 작업에서는 데이터베이스 및 해당 개체에 대한 액세스를 관리합니다. 먼저 *AdventureWorksLT* 데이터베이스에서 두 명의 사용자를 생성합니다.

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 SSMS에서 Azure Server 관리자 계정 또는 Microsoft Entra 관리자 계정을 사용하여 *AdventureWorksLT* 데이터베이스에 로그인합니다.

1. **개체 탐색기**를 사용하고 **데이터베이스**를 확장합니다.

1. **AdventureWorksLT**를 마우스 오른쪽 단추로 클릭하고 **새 쿼리**를 선택합니다.

1. 새 쿼리 창에서 아래 T-SQL을 복사하여 붙여넣습니다. 쿼리를 실행하여 두 명의 사용자를 생성합니다.

    ```sql
    CREATE USER [DP300User1] WITH PASSWORD = 'Azur3Pa$$';
    GO

    CREATE USER [DP300User2] WITH PASSWORD = 'Azur3Pa$$';
    GO
    ```

    **참고:** 두 사용자는 AdventureWorksLT 데이터베이스의 범위에서 생성되었습니다. 사용자 지정 역할을 만들고 사용자를 역할에 추가합니다.

1. 쿼리 창에서 다음 T-SQL을 실행합니다.

    ```sql
    CREATE ROLE [SalesReader];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User1];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User2];
    GO
    ```

    **SalesLT** 스키마에서 새 저장 프로시저를 생성합니다.

1. 쿼리 창에서 아래 T-SQL을 실행합니다.

    ```sql
    CREATE OR ALTER PROCEDURE SalesLT.DemoProc
    AS
    SELECT P.Name, Sum(SOD.LineTotal) as TotalSales ,SOH.OrderDate
    FROM SalesLT.Product P
    INNER JOIN SalesLT.SalesOrderDetail SOD on SOD.ProductID = P.ProductID
    INNER JOIN SalesLT.SalesOrderHeader SOH on SOH.SalesOrderID = SOD.SalesOrderID
    GROUP BY P.Name, SOH.OrderDate
    ORDER BY TotalSales DESC
    GO
    ```

    다음으로 `EXECUTE AS USER` 구문을 사용하여 보안을 테스트합니다. 이렇게 하면 데이터베이스 엔진이 사용자 컨텍스트에서 쿼리를 실행할 수 있습니다.

1. 다음 T-SQL을 실행합니다.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

    실행에 실패하고 다음 메시지가 표시됩니다.

    <span style="color:red">Msg 229, Level 14, State 5, Procedure SalesLT.DemoProc, Line 1 [Batch Start Line 0]  The EXECUTE permission was denied on the object 'DemoProc', database 'AdventureWorksLT', schema 'SalesLT'.</span>

1. 역할에 저장 프로시저를 실행할 수 있는 권한을 부여합니다. 아래 T-SQL을 실행합니다.

    ```sql
    REVERT;
    GRANT EXECUTE ON SCHEMA::SalesLT TO [SalesReader];
    GO
    ```

    첫 번째 명령은 실행 컨텍스트를 데이터베이스 소유자로 되돌립니다.

1. 이전 T-SQL을 다시 실행합니다.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

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

### LabFiles 폴더 삭제

이 랩에 대해 새 LabFiles 폴더를 만들었고 더 이상 필요하지 않은 경우 LabFiles 폴더를 삭제하여 이 랩에서 만든 모든 파일을 제거할 수 있습니다.

1. 랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 파일 탐색기를 열고 **C:\\** 드라이브로 이동합니다.
1. **LabFiles** 폴더를 마우스 오른쪽 단추로 클릭하고 **삭제**를 선택합니다.
1. **예**를 선택하여 폴더 삭제를 확인합니다.

---

이 랩을 성공적으로 완료하셨습니다.

이 연습에서는 Microsoft Entra ID를 사용하여 Azure에서 호스트된 SQL Server에 대한 액세스 권한을 Azure 자격 증명에 부여하는 방법을 살펴보았습니다. 또한 T-SQL 문을 사용하여 새 데이터베이스 사용자를 만들고 저장 프로시저를 실행할 수 있는 권한을 부여했습니다.
