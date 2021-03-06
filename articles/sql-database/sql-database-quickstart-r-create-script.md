---
title: 간단한 R 스크립트 만들기 및 실행
titleSuffix: Azure SQL Database Machine Learning Services (preview)
description: Azure SQL Database Machine Learning Services(미리 보기)에서 간단한 R 스크립트를 실행합니다.
services: sql-database
ms.service: sql-database
ms.subservice: machine-learning
ms.custom: ''
ms.devlang: r
ms.topic: quickstart
author: garyericson
ms.author: garye
ms.reviewer: davidph
manager: cgronlun
ms.date: 04/11/2019
ms.openlocfilehash: 5b2f8231952d25f5858f8e06a957f1056ecc3651
ms.sourcegitcommit: c2065e6f0ee0919d36554116432241760de43ec8
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/26/2020
ms.locfileid: "76768502"
---
# <a name="quickstart-create-and-run-simple-r-scripts-in-azure-sql-database-machine-learning-services-preview"></a>빠른 시작: Azure SQL Database Machine Learning Services(미리 보기)에서 간단한 R 스크립트 만들기 및 실행

이 빠른 시작에서는 Azure SQL Database에서 Machine Learning Services(R 포함)를 사용하여 R 스크립트 세트를 만들고 실행합니다.

[!INCLUDE[ml-preview-note](../../includes/sql-database-ml-preview-note.md)]

## <a name="prerequisites"></a>사전 요구 사항

- 활성 구독이 있는 Azure 계정. [평가판 계정을 만듭니다](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [서버 수준 방화벽 규칙](sql-database-single-database-get-started.md)이 있는 [Azure SQL 데이터베이스](sql-database-server-level-firewall-rule.md)
- R이 활성화된 [Machine Learning Services](sql-database-machine-learning-services-overview.md). [미리 보기에 가입](sql-database-machine-learning-services-overview.md#signup)하세요.
- [SSMS(SQL Server Management Studio)](/sql/ssms/sql-server-management-studio-ssms)

> [!NOTE]
> Microsoft는 공개 미리 보기 기간에는 사용자를 온보딩하고 기존 또는 새 데이터베이스에 기계 학습을 사용하도록 설정합니다.

이 예제에서는 저장 프로시저 [sp_execute_external_script](/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql)를 사용하여 올바른 형식의 R 스크립트를 래핑합니다.

## <a name="run-a-simple-script"></a>간단한 스크립트 실행

R 스크립트를 실행하려면 이를 시스템 저장 프로시저 [sp_execute_external_script](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql)에 대한 인수로 전달합니다.

다음 단계에서는 SQL 데이터베이스에서 이 예제 R 스크립트를 실행합니다.

```r
a <- 1
b <- 2
c <- a/b
d <- a*b
print(c(c, d))
```

1. **SQL Server Management Studio**를 열고 SQL 데이터베이스에 연결합니다.

   연결에 도움이 필요한 경우 [빠른 시작: SQL Server Management Studio를 사용하여 Azure SQL 데이터베이스 연결 및 쿼리](sql-database-connect-query-ssms.md)를 참조하세요.

1. 전체 R 스크립트를 [sp_execute_external_script](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql) 저장 프로시저에 전달합니다.

   스크립트는 `@script` 인수를 통해 전달됩니다. `@script` 인수 안의 모든 항목이 유효한 R 코드여야 합니다.

    ```sql
    EXECUTE sp_execute_external_script @language = N'R'
        , @script = N'
    a <- 1
    b <- 2
    c <- a/b
    d <- a*b
    print(c(c, d))
    '
    ```

   오류가 발생하면 SQL 데이터베이스에 Machine Learning Services(R 포함) 공개 미리 보기를 사용하도록 설정되지 않은 것입니다. 위의 [필수 조건](#prerequisites)을 참조하세요.

   > [!NOTE]
   > 관리자는 외부 코드를 자동으로 실행할 수 있습니다. 다음 명령을 사용하여 다른 사용자에게 권한을 부여할 수 있습니다.
   <br>**GRANT EXECUTE ANY EXTERNAL SCRIPT TO** *\<사용자 이름\>* .

2. 올바른 결과가 계산되고 R `print` 함수가 결과를 **메시지** 창에 반환합니다.

   다음과 비슷합니다.

    **결과**

    ```text
    STDOUT message(s) from external script:
    0.5 2
    ```

## <a name="run-a-hello-world-script"></a>Hello World 스크립트 실행

일반적인 예제 스크립트는 "Hello World" 문자열만 출력하는 스크립트입니다. 다음 명령을 실행합니다.

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'OutputDataSet<-InputDataSet'
    , @input_data_1 = N'SELECT 1 AS hello'
WITH RESULT SETS(([Hello World] INT));
GO
```

이 저장 프로시저에 대한 입력은 다음과 같습니다.

| | |
|-|-|
| @language | 호출할 언어 확장을 정의합니다(이 경우에는 R). |
| @script | R 런타임으로 전달되는 명령을 정의합니다. 전체 R 스크립트는 이 인수에 유니코드 텍스트로 묶어야 합니다. **nvarchar** 형식의 변수에 텍스트를 추가한 다음, 변수를 호출할 수도 있습니다. |
| @input_data_1 | 쿼리에서 반환된 데이터는 R 런타임에 전달되고 R 런타임은 데이터를 SQL Server에 데이터 프레임으로 전달합니다. |
|WITH RESULT SETS | 절은 SQL Server에 대해 반환된 데이터 테이블의 스키마를 정의하고 "Hello World"를 열 이름 및 **int**(데이터 형식)로 추가합니다. |

이 명령은 다음 텍스트를 출력합니다.

| Hello World |
|-------------|
| 1 |

## <a name="use-inputs-and-outputs"></a>입력 및 출력 사용

기본적으로 [sp_execute_external_script](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql)는 일반적으로 유효한 SQL 쿼리 형식으로 제공하는 하나의 데이터 세트를 입력으로 받아들입니다. 그런 후 단일 R 데이터 프레임을 출력으로 반환합니다.

지금은 [sp_execute_external_script](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql)의 기본 입력 및 출력 변수인 **InputDataSet** 및 **OutputDataSet**를 사용하겠습니다.

1. 테스트 데이터의 작은 테이블을 만듭니다.

    ```sql
    CREATE TABLE RTestData (col1 INT NOT NULL)
    
    INSERT INTO RTestData
    VALUES (1);
    
    INSERT INTO RTestData
    VALUES (10);
    
    INSERT INTO RTestData
    VALUES (100);
    GO
    ```

1. `SELECT` 문을 사용하여 테이블을 쿼리합니다.
  
    ```sql
    SELECT *
    FROM RTestData
    ```

    **결과**

    ![RTestData 테이블의 콘텐츠](./media/sql-database-quickstart-r-create-script/select-rtestdata.png)

1. 다음 R 스크립트를 실행합니다. `SELECT` 문을 사용하여 테이블에서 데이터를 검색하고 R 런타임을 통해 이를 전달하고 데이터를 데이터 프레임으로 반환합니다. `WITH RESULT SETS` 절에서는 SQL Database에 대해 반환된 데이터 테이블의 스키마를 정의하고, *NewColName* 열 이름을 추가합니다.

    ```sql
    EXECUTE sp_execute_external_script @language = N'R'
        , @script = N'OutputDataSet <- InputDataSet;'
        , @input_data_1 = N'SELECT * FROM RTestData;'
    WITH RESULT SETS(([NewColName] INT NOT NULL));
    ```

    **결과**

    ![테이블에서 데이터를 반환하는 R 스크립트의 출력](./media/sql-database-quickstart-r-create-script/r-output-rtestdata.png)

1. 이제 입력 및 출력 변수의 이름을 변경하겠습니다. 기본 입력 및 출력 변수 이름은 **InputDataSet** 및 **OutputDataSet**입니다. 이 스크립트는 이름을 **SQL_in** 및 **SQL_out**으로 변경합니다.

    ```sql
    EXECUTE sp_execute_external_script @language = N'R'
        , @script = N' SQL_out <- SQL_in;'
        , @input_data_1 = N' SELECT 12 as Col;'
        , @input_data_1_name = N'SQL_in'
        , @output_data_1_name = N'SQL_out'
    WITH RESULT SETS(([NewColName] INT NOT NULL));
    ```

    R은 대/소문자를 구분합니다. R 스크립트(**SQL_out**, **SQL_in**)에서 사용되는 입력 및 출력 변수는 대/소문자를 포함하여 `@input_data_1_name` 및 `@output_data_1_name`으로 정의된 값과 일치해야 합니다.

   > [!TIP]
   > 하나의 입력 데이터 세트만 매개 변수로 전달할 수 있으며 하나의 데이터 세트만 반환할 수 있습니다. 그러나 R 코드 내에서 다른 데이터 세트를 호출할 수 있으며 데이터 세트 외에 다른 유형의 출력을 반환할 수 있습니다. 또한 모든 매개 변수에 OUTPUT 키워드를 추가하여 결과와 함께 반환되게 만들 수 있습니다.

1. 입력 데이터 없이 R 스크립트만 사용하여 값을 생성할 수도 있습니다(`@input_data_1`이 공백으로 설정됨).

   다음 스크립트는 "hello" 및 "world" 텍스트를 출력합니다.

    ```sql
    EXECUTE sp_execute_external_script @language = N'R'
        , @script = N'
    mytextvariable <- c("hello", " ", "world");
    OutputDataSet <- as.data.frame(mytextvariable);
    '
        , @input_data_1 = N''
    WITH RESULT SETS(([Col1] CHAR(20) NOT NULL));
    ```

    **결과**

    ![@script를 입력으로 사용하는 쿼리 결과](./media/sql-database-quickstart-r-create-script/r-data-generated-output.png)

## <a name="check-r-version"></a>R 버전 확인

SQL 데이터베이스에 설치된 R의 버전을 확인하려면 다음 스크립트를 실행합니다.

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'print(version)';
GO
```

R `print` 함수는 버전을 **메시지** 창으로 반환합니다. 아래 출력 예제에서 이 경우 3.4.4 R 버전이 SQL Database에 설치되어 있음을 확인할 수 있습니다.

**결과**

```text
STDOUT message(s) from external script:
                   _
platform       x86_64-w64-mingw32
arch           x86_64
os             mingw32
system         x86_64, mingw32
status
major          3
minor          4.4
year           2018
month          03
day            15
svn rev        74408
language       R
version.string R version 3.4.4 (2018-03-15)
nickname       Someone to Lean On
```

## <a name="list-r-packages"></a>R 패키지 나열

Microsoft는 SQL 데이터베이스에 Machine Learning Services가 미리 설치된 다양한 R 패키지를 제공합니다.

버전, 종속 항목, 라이선스 및 라이브러리 경로 정보를 포함하여 설치된 R 패키지 목록을 보려면 다음 스크립트를 실행합니다.

```SQL
EXEC sp_execute_external_script @language = N'R'
    , @script = N'
OutputDataSet <- data.frame(installed.packages()[,c("Package", "Version", "Depends", "License", "LibPath")]);'
WITH result sets((
            Package NVARCHAR(255)
            , Version NVARCHAR(100)
            , Depends NVARCHAR(4000)
            , License NVARCHAR(1000)
            , LibPath NVARCHAR(2000)
            ));
```

R의 `installed.packages()`에서 출력이 생성되고 결과 집합으로 반환됩니다.

**결과**

![R의 설치된 패키지](./media/sql-database-quickstart-r-create-script/r-installed-packages.png)

## <a name="next-steps"></a>다음 단계

SQL Database에서 R을 사용하여 기계 학습 모델을 만들려면

> [!div class="nextstepaction"]
> [Azure SQL Database Machine Learning Services(미리 보기)를 사용하여 R에서 예측 모델 만들기 및 학습](sql-database-quickstart-r-train-score-model.md) 빠른 시작을 수행합니다.

Azure SQL Database Machine Learning Services(R 포함)(미리 보기)에 대한 자세한 내용은 다음 문서를 참조하세요.

- [Azure SQL Database Machine Learning Services(R 포함)(미리 보기)](sql-database-machine-learning-services-overview.md)
- [Machine Learning Services(미리 보기)를 사용하여 Azure SQL Database에서 고급 R 함수 작성](sql-database-machine-learning-services-functions.md)
- [Azure SQL Database Machine Learning Services(미리 보기)에서 R 및 SQL 데이터 사용](sql-database-machine-learning-services-data-issues.md)
