---
lab:
  title: 랩 2 - Azure SQL Database 프로비저닝
  module: Plan and Implement Data Platform Resources
---

# Azure SQL Database 프로비전

**예상 시간: 40분**

학생들은 가상 네트워크 엔드포인트를 사용하여 Azure SQL Database를 배포하는 데 필요한 기본 리소스를 구성합니다. SQL Database에 대한 연결은 사용 가능한 경우 랩 VM의 SQL Server Management Studio를 사용하거나 로컬 컴퓨터 설정에서 유효성을 검사합니다.

AdventureWorks의 데이터베이스 관리자로서 배포 보안을 늘리고 단순화하기 위해 가상 네트워크 엔드포인트를 포함하는 새 SQL Database를 설정합니다. SQL Server Management Studio는 데이터 쿼리 및 결과 보존을 위해 SQL Notebook의 사용을 평가하는 데 사용됩니다.

## Azure Portal에서 탐색

1. 랩 가상 머신이 제공된 경우 랩 가상 머신에서, 그렇지 않은 경우 로컬 컴퓨터에서 브라우저 창을 엽니다.

1. [https://portal.azure.com](https://portal.azure.com/)에서 Azure 포털을 탐색합니다. Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인하거나 사용 가능한 경우 자격 증명을 제공하여 로그인합니다.

1. Azure Portal의 맨 위에 있는 검색 상자에서 *리소스 그룹*을 검색한 후 옵션 목록에서 **리소스 그룹**을 선택합니다.

1. **리소스 그룹** 페이지에서 *contoso-rg*로 시작하는 리소스 그룹을 선택합니다(제공된 경우). 이 리소스 그룹이 없는 경우 해당 지역에서 *contoso-rg*로 명명된 새 리소스 그룹을 만들거나 기존 리소스 그룹을 사용하고 해당 지역을 기록해 둡니다.

## Virtual Network 만들기

1. Azure Portal 홈페이지에서 왼쪽 메뉴를 선택합니다.  

1. 왼쪽 탐색 창에서 **Virtual Networks**를 선택합니다.  

1. **+ 만들기**를 클릭하여 **Virtual Networks 만들기** 페이지를 엽니다. **기본** 탭에서 다음 작업을 완료합니다.

    - **구독**: &lt;사용자의 구독&gt;
    - **리소스 그룹**: *DP300* 또는 이전에 선택한 리소스 그룹으로 시작
    - **이름:** lab02-vnet
    - **지역:** 리소스 그룹이 만들어진 지역과 동일한 지역 선택

1. **검토 + 만들기**를 선택하고 새 가상 네트워크에 대한 설정을 검토한 후 **만들기**를 선택합니다.

## Azure Portal에서 Azure SQL Database 프로비전

1. Azure Portal의 맨 위에 있는 검색 상자에서 *SQL Database*를 검색한 후 옵션 목록에서 **SQL Database**를 선택합니다.

1. **SQL 데이터베이스** 블레이드에서 **+ 만들기**를 선택합니다.

1. **SQL Database 만들기** 페이지의 **기본** 탭에서 다음 옵션을 선택하고 **다음: 네트워킹**을 선택합니다.

    - **구독**: &lt;사용자의 구독&gt;
    - **리소스 그룹**: *DP300* 또는 이전에 선택한 리소스 그룹으로 시작
    - **데이터베이스 이름:** AdventureWorksLT
    - **서버:** **새 링크 만들기**를 선택합니다. **SQL Database 서버 만들기** 페이지가 열립니다. 다음과 같은 서버 세부 정보를 제공합니다.
        - **서버 이름:** dp300-lab-&lt;사용자 이니셜(소문자)&gt; 및 임의의 5자리 숫자가 필요한 경우(서버 이름은 전역적으로 고유해야 함)
        - **위치:** &lt;사용자 리소스 그룹의 선택한 지역과 동일한 사용자 지역, 동일하지 않으면 실패할 수 있음&gt;
        - **인증 방법**: SQL 인증 사용
        - **서버 관리자 로그인:** dp300admin
        - **암호:** 복잡한 암호를 선택하고 기록해 둠
        - **암호 확인:** 이전에 선택한 암호와 동일한 암호 선택
    - **확인**을 선택하여 **SQL Database 만들기** 페이지로 돌아갑니다.
    - **탄력적 풀을 사용하시겠습니까?** 를 **아니요**로 설정합니다.
    - **워크로드 환경**: 개발
    - **컴퓨팅 + 스토리지** 옵션에서 **데이터베이스 구성** 링크를 선택합니다. **구성** 페이지의 **서비스 계층** 드롭다운에서 **기본**을 선택한 다음 **적용**을 선택합니다.

1. **백업 스토리지 중복** 옵션의 경우 기본값인 **지역 중복 백업 스토리지**를 유지합니다.

1. 그런 다음 **다음: 네트워킹**을 선택합니다.

1. **네트워킹** 탭의 **네트워크 연결** 옵션에서 **프라이빗 엔드포인트** 라디오 단추를 선택합니다.

1. 그런 다음 **프라이빗 엔드포인트** 옵션에서 **+ 프라이빗 엔드포인트 추가** 링크를 선택합니다.

1. 다음과 같이 **프라이빗 엔드포인트 만들기** 오른쪽 창을 완료합니다.

    - **구독**: &lt;사용자의 구독&gt;
    - **리소스 그룹**: *DP300* 또는 이전에 선택한 리소스 그룹으로 시작
    - **위치:** &lt;사용자 리소스 그룹의 선택한 지역과 동일한 사용자 지역, 동일하지 않으면 실패할 수 있음&gt;
    - **이름:** DP-300-SQL-Endpoint
    - **대상 하위 리소스:** SqlServer
    - **가상 네트워크:** lab02-vnet
    - **서브넷:** lab02-vnet/default(10.x.0.0/24)
    - **프라이빗 DNS 영역과 통합:** 예
    - **프라이빗 DNS 영역:** 기본값 유지
    - 설정을 검토한 후 **확인**을 선택합니다.  

1. 새 엔드포인트가 **프라이빗 엔드포인트** 목록에 표시됩니다.

1. **다음: 보안**을 선택한 후 **다음: 추가 설정**을 선택합니다.  

1. **추가 설정** 페이지의 **기존 데이터 사용** 옵션에서 **샘플**을 선택합니다. 샘플 데이터베이스의 팝업 메시지가 표시되면 **확인**을 선택합니다.

1. **검토 + 생성**를 선택합니다.

1. **생성**를 선택하기 전에 설정을 검토합니다.

1. 배포가 완료되면 **리소스로 이동**을 선택합니다.

## Azure SQL Database에 대한 액세스 사용

1. **SQL Database** 페이지에서 **개요** 섹션을 선택한 후 상단 섹션에서 서버 이름의 링크를 선택합니다.

1. SQL 서버 탐색 블레이드의 **보안** 섹션에서 **네트워킹**을 선택합니다.

1. **퍼블릭 액세스** 탭에서 **선택한 네트워크**를 선택합니다.

1. **+클라이언트 IPv4 주소 추가**를 선택합니다. 그러면 현재 IP 주소가 SQL Server에 액세스할 수 있도록 허용하는 방화벽 규칙이 추가됩니다.

1. **Azure 서비스 및 리소스에서 이 서버에 액세스할 수 있도록 허용** 속성을 선택합니다.

1. **저장**을 선택합니다.

---

## SQL Server Management Studio에서 Azure SQL Database에 연결

1. Azure Portal의 왼쪽 탐색 창에서 **SQL Database**를 선택합니다. 그런 다음 **AdventureWorksLT** 데이터베이스를 선택합니다.

1. **개요** 페이지에서 **서버 이름** 값을 복사합니다.

1. 랩 가상 머신에서(제공된 경우) SQL Server Management Studio를 시작하거나 그렇지 않은 경우 로컬 컴퓨터에서 시작합니다.

1. **서버에 연결** 대화 상자에 Azure Portal에서 복사한 **서버 이름** 값을 붙여넣습니다.

1. **인증** 드롭다운에서 **SQL Server 인증**을 선택합니다.

1. **로그인** 필드에 **dp300admin**을 입력합니다.

1. **암호** 필드에 SQL Server를 만드는 동안 선택한 암호를 입력합니다.

1. **연결**을 선택합니다.

1. SQL Server Management Studio가 Azure SQL Database 서버에 연결됩니다. 서버를 확장한 후 **데이터베이스** 노드를 확장하여 *AdventureWorksLT* 데이터베이스를 확인할 수 있습니다.

## SQL Server Management Studio를 사용한 Azure SQL Database 쿼리

1. SQL Server Management Studio에서 *AdventureWorksLT* 데이터베이스를 마우스 오른쪽 단추로 클릭하고 **새 쿼리**를 선택합니다.

1. 쿼리 창에 다음 SQL 문을 붙여 넣습니다.

    ```sql
    SELECT TOP 10 cust.[CustomerID], 
        cust.[CompanyName], 
        SUM(sohead.[SubTotal]) as OverallOrderSubTotal
    FROM [SalesLT].[Customer] cust
        INNER JOIN [SalesLT].[SalesOrderHeader] sohead
             ON sohead.[CustomerID] = cust.[CustomerID]
    GROUP BY cust.[CustomerID], cust.[CompanyName]
    ORDER BY [OverallOrderSubTotal] DESC
    ```

1. 쿼리를 실행하려면 도구 모음에서 **실행** 단추를 선택합니다.

1. **결과** 창에서 쿼리 결과를 검토합니다.

1. *AdventureWorksLT* 데이터베이스를 마우스 오른쪽 단추로 클릭하고 **새 쿼리**를 선택합니다.

1. 쿼리 창에 다음 SQL 문을 붙여 넣습니다.

    ```sql
    SELECT TOP 10 cat.[Name] AS ProductCategory, 
        SUM(detail.[OrderQty]) AS OrderedQuantity
    FROM salesLT.[ProductCategory] cat
        INNER JOIN [SalesLT].[Product] prod
            ON prod.[ProductCategoryID] = cat.[ProductCategoryID]
        INNER JOIN [SalesLT].[SalesOrderDetail] detail
            ON detail.[ProductID] = prod.[ProductID]
    GROUP BY cat.[name]
    ORDER BY [OrderedQuantity] DESC
    ```

1. 쿼리를 실행하려면 도구 모음에서 **실행** 단추를 선택합니다.

1. **결과** 창에서 쿼리 결과를 검토합니다.

1. SQL Server Management Studio를 닫습니다. 변경 내용을 저장하라는 메시지가 표시되면 **아니요**를 선택합니다.

---

## 리소스 정리

다른 용도로 가상 머신을 사용하지 않는 경우 이 랩에서 만든 리소스를 정리할 수 있습니다.

### 리소스 그룹 삭제

이 랩에 대한 새 리소스 그룹을 만든 경우 리소스 그룹을 삭제하여 이 랩에서 만든 모든 리소스를 제거할 수 있습니다.

1. Azure Portal의 왼쪽 탐색 창에서 **리소스 그룹**을 선택하거나 검색 창에서 **리소스 그룹**을 검색하고 결과에서 선택합니다.

1. 이 랩을 위해 만든 리소스 그룹으로 이동합니다. 리소스 그룹에는 가상 머신 및 이 랩에서 만든 기타 리소스가 포함됩니다.

1. 위쪽 메뉴에서 **리소스 그룹 삭제**를 선택합니다.

1. **리소스 그룹 삭제** 대화 상자에서 확인할 리소스 그룹의 이름을 입력한 다음 **삭제**를 선택합니다.

1. 리소스 그룹이 삭제될 때까지 기다립니다.

1. Azure Portal을 닫습니다.

### 랩 리소스만 삭제합니다.

이 랩에 대한 새 리소스 그룹을 만들지 않았고 리소스 그룹과 이전 리소스를 그대로 유지하려는 경우 이 랩에서 만든 리소스를 계속 삭제할 수 있습니다.

1. Azure Portal의 왼쪽 탐색 창에서 **리소스 그룹**을 선택하거나 검색 창에서 **리소스 그룹**을 검색하고 결과에서 선택합니다.

1. 이 랩을 위해 만든 리소스 그룹으로 이동합니다. 리소스 그룹에는 가상 머신 및 이 랩에서 만든 기타 리소스가 포함됩니다.

1. 랩에서 이전에 지정한 SQL Server 이름이 접두사로 지정된 모든 리소스를 선택합니다. 또한 만든 가상 네트워크 및 프라이빗 DNS 영역을 선택합니다.

1. 위쪽 메뉴에서 **삭제**를 선택합니다.

1. **리소스 삭제** 대화 상자에서 **삭제**를 입력하고 **삭제**를 선택합니다.

1. 리소스 삭제를 확인하려면 **삭제**를 다시 선택합니다.

1. 리소스가 삭제될 때까지 기다립니다.

1. Azure Portal을 닫습니다.

---

이 랩을 성공적으로 완료하셨습니다.

이 연습에서는 Virtual Network Endpoint를 사용하여 Azure SQL Database를 배포하는 방법을 알아보았습니다. 또한 SQL Server Management Studio를 사용하여 만든 SQL Database에 연결할 수 있었습니다.
