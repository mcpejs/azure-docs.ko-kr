---
title: Azure 데이터 팩터리를 사용하여 DB2에서 데이터 복사
description: Azure Data Factory 파이프라인의 복사 작업을 사용하여 DB2에서 지원되는 싱크 데이터 저장소로 데이터를 복사하는 방법에 대해 알아봅니다.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 02/17/2020
ms.author: jingwang
ms.openlocfilehash: 22ecac12e049e58e533cdde0078f4a25f6bb2aa6
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77423830"
---
# <a name="copy-data-from-db2-by-using-azure-data-factory"></a>Azure Data Factory를 사용하여 DB2에서 데이터 복사
> [!div class="op_single_selector" title1="사용 중인 Data Factory 서비스 버전을 선택합니다."]
> * [버전 1](v1/data-factory-onprem-db2-connector.md)
> * [현재 버전](connector-db2.md)

이 문서에서는 Azure Data Factory의 복사 작업을 사용하여 DB2 데이터베이스에서 데이터를 복사하는 방법을 설명합니다. 이 문서는 복사 작업에 대한 일반적인 개요를 제공하는 [복사 작업 개요](copy-activity-overview.md) 문서를 기반으로 합니다.

## <a name="supported-capabilities"></a>지원되는 기능

이 DB2 데이터베이스 커넥터는 다음 활동에 대해 지원됩니다.

- [지원되는 소스/싱크 매트릭스로](copy-activity-overview.md) [활동 복사](copy-activity-overview.md)
- [조회 활동](control-flow-lookup-activity.md)

DB2 데이터베이스에서 지원되는 모든 싱크 데이터 저장소로 데이터를 복사할 수 있습니다. 복사 작업의 원본/싱크로 지원되는 데이터 저장소 목록은 [지원되는 데이터 저장소](copy-activity-overview.md#supported-data-stores-and-formats) 표를 참조하세요.

특히, 이 DB2 커넥터는 DRDA(분산 관계형 데이터베이스 아키텍처) SQLAM(SQL Access Manager) 버전 9, 10 및 11과 함께 다음 IBM DB2 플랫폼 버전을 지원합니다.

* Z/OS용 IBM DB2 12.1
* z/OS용 IBM DB2 11.1
* z/OS용 IBM DB2 10.1
* i용 IBM DB2 7.3
* i용 IBM DB2 7.2
* i용 IBM DB2 7.1
* LUW용 IBM DB2 11
* LUW용 IBM DB2 10.5
* LUW용 IBM DB2 10.1

>[!TIP]
>DB2 커넥터는 DB2용 Microsoft OLE DB 공급자 위에 빌드됩니다. DB2 커넥터 오류 문제를 해결하려면 [데이터 공급자 오류 코드를](https://docs.microsoft.com/host-integration-server/db2oledbv/data-provider-error-codes#drda-protocol-errors)참조하십시오.

## <a name="prerequisites"></a>사전 요구 사항

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

통합 런타임은 기본 제공 DB2 드라이버를 제공하므로 DB2에서 데이터를 복사할 때 수동으로 드라이버를 설치할 필요가 없습니다.

## <a name="getting-started"></a>시작

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

다음 섹션에서는 DB2 커넥터에 한정된 Data Factory 엔터티를 정의하는 데 사용되는 속성에 대해 자세히 설명합니다.

## <a name="linked-service-properties"></a>연결된 서비스 속성

DB2 연결된 서비스에 다음 속성이 지원됩니다.

| 속성 | 설명 | 필수 |
|:--- |:--- |:--- |
| type | 형식 속성은 **Db2**로 설정해야 합니다. | yes |
| 서버 |DB2 서버의 이름입니다. 콜론으로 구분된 서버 이름 뒤에 포트 번호를 지정할 수 있습니다(예: `server:port`). |yes |
| 데이터베이스 |DB2 데이터베이스의 이름입니다. |yes |
| authenticationType |DB2 데이터베이스에 연결하는 데 사용되는 인증 형식입니다.<br/>허용되는 값은 **Basic**입니다. |yes |
| 사용자 이름 |DB2 데이터베이스에 연결할 사용자 이름을 지정합니다. |yes |
| password |사용자 이름에 지정한 사용자 계정의 암호를 지정합니다. 이 필드를 SecureString으로 표시하여 Data Factory에 안전하게 저장하거나, [Azure Key Vault에 저장된 비밀을 참조](store-credentials-in-key-vault.md)합니다. |yes |
| 패키지 컬렉션 | 데이터베이스를 쿼리할 때 ADF에서 필요한 패키지가 자동으로 생성되는 위치를 지정합니다. | 예 |
| 인증서공통 이름 | SSL(보안 소켓 계층) 또는 TLS(전송 계층 보안) 암호화를 사용하는 경우 인증서 공통 이름에 대한 값을 입력해야 합니다. | 예 |
| connectVia | 데이터 저장소에 연결하는 데 사용할 [통합 런타임입니다.](concepts-integration-runtime.md) [필수 구성 조건](#prerequisites) 섹션에서 자세히 알아보십시오. 지정하지 않으면 기본 Azure Integration Runtime을 사용합니다. |예 |

> [!TIP]
> 사용자에게 필요한 패키지가 만들어지지 않는다는 오류 메시지가 `The package corresponding to an SQL statement execution request was not found. SQLSTATE=51002 SQLCODE=-805`나타납니다. 기본적으로 ADF는 DB2에 연결하는 데 사용한 사용자로 명명된 컬렉션에서 패키지를 만들려고 합니다. ADF가 데이터베이스를 쿼리할 때 필요한 패키지를 만들 위치를 나타내도록 패키지 컬렉션 속성을 지정합니다.

**예제:**

```json
{
    "name": "Db2LinkedService",
    "properties": {
        "type": "Db2",
        "typeProperties": {
            "server": "<servername:port>",
            "database": "<dbname>",
            "authenticationType": "Basic",
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>데이터 세트 속성

데이터 집합을 정의하는 데 사용할 수 있는 섹션 및 속성의 전체 목록은 [데이터 집합](concepts-datasets-linked-services.md) 문서를 참조하세요. 이 섹션에서는 DB2 데이터 세트에서 지원하는 속성의 목록을 제공합니다.

DB2에서 데이터를 복사하려면 다음 속성이 지원됩니다.

| 속성 | 설명 | 필수 |
|:--- |:--- |:--- |
| type | 데이터 집합의 형식 속성을 **DB2Table으로** 설정해야 합니다. | yes |
| 스키마 | 스키마의 이름입니다. |아니요(작업 원본에서 "query"가 지정된 경우)  |
| 테이블 | 테이블 이름입니다. |아니요(작업 원본에서 "query"가 지정된 경우)  |
| tableName | 스키마가 있는 테이블의 이름입니다. 이 속성은 이전 버전과의 호환성을 위해 지원됩니다. 새 `schema` `table` 워크로드에 사용하고 사용합니다. | 아니요(작업 원본에서 "query"가 지정된 경우) |

**예제**

```json
{
    "name": "DB2Dataset",
    "properties":
    {
        "type": "Db2Table",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<DB2 linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

형식이 지정된 데이터 집합을 사용하는 `RelationalTable` 경우 계속 지원되지만 앞으로 새 데이터 집합을 사용하는 것이 좋습니다.

## <a name="copy-activity-properties"></a>복사 작업 속성

작업 정의에 사용할 수 있는 섹션 및 속성의 전체 목록은 [파이프라인](concepts-pipelines-activities.md) 문서를 참조하세요. 이 섹션에서는 DB2 원본에서 지원하는 속성의 목록을 제공합니다.

### <a name="db2-as-source"></a>DB2를 원본으로

DB2에서 데이터를 복사하려면 복사 활동 **소스** 섹션에서 다음 속성이 지원됩니다.

| 속성 | 설명 | 필수 |
|:--- |:--- |:--- |
| type | 복사 활동 소스의 형식 속성을 **DB2Source로** 설정해야 합니다. | yes |
| Query | 사용자 지정 SQL 쿼리를 사용하여 데이터를 읽습니다. 예: `"query": "SELECT * FROM \"DB2ADMIN\".\"Customers\""` | 아니요(데이터 세트의 "tableName"이 지정된 경우) |

**예제:**

```json
"activities":[
    {
        "name": "CopyFromDB2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<DB2 input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "Db2Source",
                "query": "SELECT * FROM \"DB2ADMIN\".\"Customers\""
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

형식이 지정된 소스를 사용하는 `RelationalSource` 경우 계속 있는 것으로 지원되지만 앞으로 새 소스를 사용하는 것이 좋습니다.

## <a name="data-type-mapping-for-db2"></a>DB2에 대한 데이터 형식 매핑

DB2에서 데이터를 복사하는 경우 DB2 데이터 형식에서 Azure Data Factory 중간 데이터 형식으로 다음 매핑이 사용됩니다. 복사 작업에서 원본 스키마 및 데이터 형식을 싱크에 매핑하는 방법에 대한 자세한 내용은 [스키마 및 데이터 형식 매핑](copy-activity-schema-and-type-mapping.md)을 참조하세요.

| DB2 데이터베이스 형식 | Data Factory 중간 데이터 형식 |
|:--- |:--- |
| BigInt |Int64 |
| 이진 |Byte[] |
| Blob |Byte[] |
| Char |String |
| Clob |String |
| Date |DateTime |
| DB2DynArray |String |
| DbClob |String |
| Decimal |Decimal |
| DecimalFloat |Decimal |
| Double |Double |
| Float |Double |
| Graphic |String |
| 정수 |Int32 |
| LongVarBinary |Byte[] |
| LongVarChar |String |
| LongVarGraphic |String |
| 숫자 |Decimal |
| Real |Single |
| SmallInt |Int16 |
| Time |TimeSpan |
| 타임스탬프 |DateTime |
| VarBinary |Byte[] |
| VarChar |String |
| VarGraphic |String |
| xml |Byte[] |

## <a name="lookup-activity-properties"></a>조회 활동 속성

속성에 대한 자세한 내용을 보려면 [조회 활동을](control-flow-lookup-activity.md)선택합니다.

## <a name="next-steps"></a>다음 단계
Azure Data Factory에서 복사 작업의 원본 및 싱크로 지원되는 데이터 저장소 목록은 [지원되는 데이터 저장소](copy-activity-overview.md#supported-data-stores-and-formats)를 참조하세요.
