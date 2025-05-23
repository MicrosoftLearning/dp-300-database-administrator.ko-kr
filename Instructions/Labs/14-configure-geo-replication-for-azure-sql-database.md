---
lab:
  title: 랩 14 - Azure SQL Database의 지역 복제 구성
  module: Plan and implement a high availability and disaster recovery solution
---

# Azure SQL Database의 지역 복제 구성

**예상 시간:** 30분

AdventureWorks 내의 DBA로서 Azure SQL Database의 지역 복제를 사용하도록 설정하고 제대로 작동하는지 확인해야 합니다. 또한 포털을 사용하여 수동으로 다른 지역으로 장애 조치합니다.

> &#128221; 이러한 연습에서는 T-SQL 코드를 복사하여 붙여넣고 기존 SQL 리소스를 활용하도록 요청할 수 있습니다. 코드를 실행하기 전에 코드를 올바르게 복사했는지 확인하세요.

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

1. 스크립트가 완료되면 리소스 그룹 이름, SQL Server 이름 및 데이터베이스 이름, 관리자 사용자 이름 및 암호를 반환합니다. *이러한 값은 랩의 뒷부분에서 필요하므로 기록해 두십시오*.

---

## 지리 복제 사용

1.  랩 가상 머신 또는 로컬 컴퓨터가 제공되지 않은 경우 브라우저 세션을 시작하고 [https://portal.azure.com](https://portal.azure.com/)으로 이동합니다. Azure 자격 증명을 사용하여 Portal에 연결합니다.

1. Azure Portal에서 **sql 데이터베이스**를 검색하여 데이터베이스로 이동합니다.

1. SQL 데이터베이스 **AdventureWorksLT**를 선택합니다.

1. 데이터베이스 블레이드의 **데이터 관리** 섹션에서 **복제본**을 선택합니다.

1. **+ 복제본 만들기**를 선택합니다.

1. **SQL Database 만들기 - 지역 복제본** 페이지에서 **프로젝트 세부 정보** 및 **주 데이터베이스** 섹션이 이미 구독, 리소스 그룹 및 데이터베이스 이름으로 채워져 있음을 알 수 있습니다.

1. **복제본 구성** 섹션의 경우 *복제본 유형*으로 **지역 복제본**을 선택합니다.

1. **지역 보조 데이터베이스 세부 정보**에 다음 값을 입력합니다.

    - **구독**: &lt;사용자의 구독 이름&gt;(주 데이터베이스와 동일).
    - **리소스 그룹**: &lt;주 데이터베이스와 동일한 리소스 그룹을 선택합니다.&gt; 
    - **데이터베이스 이름**: 데이터베이스 이름이 회색으로 표시되며 주 데이터베이스 이름과 동일합니다.
    - **서버**: **새로 만들기**를 선택합니다.
    - **SQL Database 서버 만들기** 페이지에서 다음과 같은 값을 입력합니다.

        - **서버 이름**: 보조 서버의 고유한 이름을 입력합니다. 이름은 모든 Azure SQL Database 서버에서 고유해야 합니다.
        - **위치**: 주 데이터베이스에서 다른 지역을 선택합니다. 해당 구독은 일부 지역에서 사용이 불가능할 수 있습니다.
        - **Azure 서비스의 서버 액세스 허용** 확인란을 선택합니다. 프로덕션 환경에서는 서버에 대한 액세스를 제한할 수 있습니다.
        - 인증 유형으로 **SQL 인증**을 선택합니다. 프로덕션 환경에서는 **Microsoft Entra 전용 인증**을 사용할 수 있습니다. 관리자 로그인 이름 및 보안 암호에 **sqladmin*을 입력합니다. 암호는 12자 이상의 Azure SQL 암호 복잡성 요구 사항을 충족해야 하며 대문자 1개, 소문자 1개, 숫자 1개, 특수 문자 1개 이상을 포함해야 합니다.
        - **확인**을 선택하여 서버를 생성합니다.

    - **탄력적 풀을 사용하시겠습니까?** 아니요.
    - **컴퓨팅 + 스토리지**: 범용, 5세대, 2개의 vCore, 32GB 스토리지.
    - **백업 스토리지 중복**: LRS(로컬 중복 스토리지). 프로덕션 환경에서는 **GRS(지역 중복 스토리지)** 를 사용할 수 있습니다.

1. **검토 + 생성**를 선택합니다.

1. **만들기**를 선택합니다. 보조 서버 및 데이터베이스를 만드는 데 몇 분 정도 걸립니다. 완료되면 진행률이 **진행 중인 배포**에서 **배포 완료**로 변경됩니다.

1. **리소스로 이동**을 선택하여 다음 단계를 위해 보조 서버의 데이터베이스로 이동합니다.

## SQL Database를 보조 지역으로 장애 조치(failover)

이제 Azure SQL Database 복제본이 만들어졌으므로 장애 조치(failover)를 수행합니다.

1. 보조 서버의 데이터베이스에 아직 없는 경우 Azure Portal에서 **sql databases**를 검색하고 보조 서버에서 SQL Database **AdventureWorksLT**를 선택합니다.

1. SQL 데이터베이스 기본 블레이드의 **데이터 관리** 섹션에서 **복제본**을 선택합니다.

1. 이제 지역 복제 링크가 설정되었습니다. 주 데이터베이스의 *복제본 상태* 값은 **Online**이며 지역 복제본의 *복제본 상태* 값은 **Readable**입니다.

1. 보조 지역 복제본 서버에 대해 **...** 메뉴를 선택하고 **강제 장애 조치(failover)** 를 선택합니다.

    > &#128221; 강제 장애 조치(failover)는 보조 데이터베이스를 주 역할로 전환합니다. 이 작업 중에는 모든 세션의 연결이 끊어집니다.

1. 경고 메시지가 표시되면 **예**를 클릭합니다.

1. 주 복제본의 상태는 **보류 중**으로 전환되고 보조 복제본의 상태는 **장애 조치(failover)** 로 전환됩니다. 

     > &#128221; 데이터베이스가 작기 때문에 장애 조치(failover)가 빨라집니다. 프로덕션 환경에서 이 프로세스는 몇 분 정도 걸릴 수 있습니다.

1. 완료되면 역할이 전환되어 보조 서버는 새로운 주 서버가 되고 기존 주 서버는 보조 서버가 됩니다. 새 상태를 보려면 페이지를 새로 고쳐야 할 수도 있습니다.

읽기 가능한 보조 데이터베이스는 주 데이터베이스와 동일한 Azure 지역에 있을 수 있고 더 일반적으로는 다른 지역에 있을 수도 있음을 확인했습니다. 이러한 종류의 읽기 가능한 보조 데이터베이스를 지역 보조 또는 지역 복제본이라고도 합니다.

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

지금까지 Azure SQL Database의 지역 복제본을 사용하도록 설정하고 포털을 사용하여 다른 지역에 수동으로 장애 조치(failover)하는 방법을 살펴보았습니다.
