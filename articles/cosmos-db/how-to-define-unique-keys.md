---
title: Azure Cosmos 컨테이너의 고유 키 정의
description: Azure 포털, PowerShell, .Net, Java 및 기타 다양한 SDK를 사용하여 Azure Cosmos 컨테이너에 대한 고유 키를 정의하는 방법을 알아봅니다.
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 12/02/2019
ms.author: thweiss
ms.openlocfilehash: fa62495a7b51c9a06a91102299378c15e811eae0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74872114"
---
# <a name="define-unique-keys-for-an-azure-cosmos-container"></a>Azure Cosmos 컨테이너의 고유 키 정의

이 문서에서는 Azure Cosmos 컨테이너를 만들 때 [고유 키](unique-keys.md)를 정의하는 여러 다른 방법을 제공합니다. 현재, Azure Portal을 사용하거나 SDK 중 하나를 통해 이 작업을 수행할 수 있습니다.

## <a name="use-the-azure-portal"></a>Azure Portal 사용

1. [Azure 포털에](https://portal.azure.com/)로그인합니다.

1. [새 Azure Cosmos 계정을 만들거나](create-sql-api-dotnet.md#create-account) 기존 계정을 선택합니다.

1. **Data Explorer** 창을 열고 작업할 컨테이너를 선택합니다.

1. **새 컨테이너**를 클릭합니다.

1. **컨테이너 추가** 대화 상자에서 **+ 고유 키 추가**를 클릭하여 고유 키 항목을 추가합니다.

1. 고유 키 제약 조건의 경로 입력

1. 필요한 경우 **+ 고유 키 추가**를 클릭하여 더 많은 고유 키 항목을 추가합니다.

    ![Azure Portal의 고유 키 제약 조건 항목 스크린샷](./media/how-to-define-unique-keys/unique-keys-portal.png)

## <a name="use-powershell"></a>PowerShell 사용

고유한 키가 있는 컨테이너를 만들려면 [고유한 키와 TTL을 사용하여 Azure Cosmos 컨테이너 만들기](manage-with-powershell.md#create-container-unique-key-ttl)

## <a name="use-the-net-sdk-v2"></a>.NET SDK V2 사용

[.NET SDK v2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/)를 사용하여 새 컨테이너를 만들 경우 `UniqueKeyPolicy` 개체를 사용하여 고유 키 제약 조건을 정의할 수 있습니다.

```csharp
client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("database"), new DocumentCollection
{
    Id = "container",
    PartitionKey = new PartitionKeyDefinition { Paths = new Collection<string>(new List<string> { "/myPartitionKey" }) },
    UniqueKeyPolicy = new UniqueKeyPolicy
    {
        UniqueKeys = new Collection<UniqueKey>(new List<UniqueKey>
        {
            new UniqueKey { Paths = new Collection<string>(new List<string> { "/firstName", "/lastName", "/emailAddress" }) },
            new UniqueKey { Paths = new Collection<string>(new List<string> { "/address/zipCode" }) }
        })
    }
});
```

## <a name="use-the-net-sdk-v3"></a>.NET SDK V3 사용

[.NET SDK v3를](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/)사용하여 새 컨테이너를 만들 때 SDK의 유창한 API를 사용하여 간결하고 읽기 쉬운 방식으로 고유 키를 선언합니다.

```csharp
await client.GetDatabase("database").DefineContainer(name: "container", partitionKeyPath: "/myPartitionKey")
    .WithUniqueKey()
        .Path("/firstName")
        .Path("/lastName")
        .Path("/emailAddress")
    .Attach()
    .WithUniqueKey()
        .Path("/address/zipCode")
    .Attach()
    .CreateIfNotExistsAsync();
```

## <a name="use-the-java-sdk"></a>Java SDK 사용

[Java SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb)를 사용하여 새 컨테이너를 만들 경우 `UniqueKeyPolicy` 개체를 사용하여 고유 키 제약 조건을 정의할 수 있습니다.

```java
// create a new DocumentCollection object
DocumentCollection container = new DocumentCollection();
container.setId("container");

// create array of strings and populate them with the unique key paths
Collection<String> uniqueKey1Paths = new ArrayList<String>();
uniqueKey1Paths.add("/firstName");
uniqueKey1Paths.add("/lastName");
uniqueKey1Paths.add("/emailAddress");
Collection<String> uniqueKey2Paths = new ArrayList<String>();
uniqueKey2Paths.add("/address/zipCode");

// create UniqueKey objects and set their paths
UniqueKey uniqueKey1 = new UniqueKey();
UniqueKey uniqueKey2 = new UniqueKey();
uniqueKey1.setPaths(uniqueKey1Paths);
uniqueKey2.setPaths(uniqueKey2Paths);

// create a new UniqueKeyPolicy object and set its unique keys
UniqueKeyPolicy uniqueKeyPolicy = new UniqueKeyPolicy();
Collection<UniqueKey> uniqueKeys = new ArrayList<UniqueKey>();
uniqueKeys.add(uniqueKey1);
uniqueKeys.add(uniqueKey2);
uniqueKeyPolicy.setUniqueKeys(uniqueKeys);

// set the unique key policy
container.setUniqueKeyPolicy(uniqueKeyPolicy);

// create the container
client.createCollection(String.format("/dbs/%s", "database"), container, null);
```

## <a name="use-the-nodejs-sdk"></a>Node.js SDK 사용

[Node.js SDK](https://www.npmjs.com/package/@azure/cosmos)를 사용하여 새 컨테이너를 만들 경우 `UniqueKeyPolicy` 개체를 사용하여 고유 키 제약 조건을 정의할 수 있습니다.

```javascript
client.database('database').containers.create({
    id: 'container',
    uniqueKeyPolicy: {
        uniqueKeys: [
            { paths: ['/firstName', '/lastName', '/emailAddress'] },
            { paths: ['/address/zipCode'] }
        ]
    }
});
```

## <a name="use-the-python-sdk"></a>Python SDK 사용

[Python SDK](https://pypi.org/project/azure-cosmos/)를 사용하여 새 컨테이너를 만들 때 매개 변수로 전달되는 사전의 일부로 고유 키 제약 조건을 지정할 수 있습니다.

```python
client.CreateContainer('dbs/' + config['DATABASE'], {
    'id': 'container',
    'uniqueKeyPolicy': {
        'uniqueKeys': [
            {'paths': ['/firstName', '/lastName', '/emailAddress']},
            {'paths': ['/address/zipCode']}
        ]
    }
})
```

## <a name="next-steps"></a>다음 단계

- [분할에](partition-data.md) 대해 자세히 알아보기
- [인덱싱 작동 원리](index-overview.md) 알아보기
