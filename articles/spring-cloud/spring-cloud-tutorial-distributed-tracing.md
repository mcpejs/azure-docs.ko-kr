---
title: 자습서 - Azure Spring Cloud에서 분산 추적 사용
description: 이 자습서에서는 Azure Application Insights를 통해 Spring Cloud의 분산 추적을 사용하는 방법을 보여줍니다.
author: bmitchell287
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 10/06/2019
ms.author: brendm
ms.openlocfilehash: 0815aa084462d1b829d64cd7c5d6fa7cebf534fc
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/24/2020
ms.locfileid: "78273202"
---
# <a name="use-distributed-tracing-with-azure-spring-cloud"></a>Azure Spring Cloud에서 분산 추적

Azure Spring Cloud의 분산 추적 도구를 사용하면 복잡한 문제를 쉽게 디버그하고 모니터링할 수 있습니다. Azure Spring Cloud는 [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth)를 Azure의 [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview)와 통합합니다. 이러한 통합은 Azure Portal의 강력한 분산 추적 기능을 제공합니다.

이 문서에서는 다음 방법을 알아봅니다.

> [!div class="checklist"]
> * Azure Portal에서 분산 추적을 사용하도록 설정
> * 애플리케이션에 Spring Cloud Sleuth 추가
> * 마이크로서비스 애플리케이션에 대한 종속성 맵 보기
> * 다양한 필터로 추적 데이터 검색

## <a name="prerequisites"></a>사전 요구 사항

이 자습서를 완료하려면 이미 프로비저닝되어 실행 중인 Azure Spring Cloud 서비스가 필요합니다. [Azure CLI를 통해 앱을 배포하는 빠른 시작](spring-cloud-quickstart-launch-app-cli.md)을 완료하면 Azure Spring Cloud 서비스를 프로비저닝하고 실행할 수 있습니다.
    
## <a name="add-dependencies"></a>종속성 추가

1. 다음 줄을 application.properties 파일에 추가합니다.

   ```xml
   spring.zipkin.sender.type = web
   ```

   이렇게 변경한 후 Zipkin 발신자를 웹에 보낼 수 있습니다.

1. [Azure Spring Cloud 애플리케이션 준비에 대한 가이드](spring-cloud-tutorial-prepare-app-deployment.md)를 수행한 경우, 이 단계를 건너뛸 수 있습니다. 그렇지 않으면 로컬 개발 환경으로 이동하고, 다음 Spring Cloud Sleuth 종속성을 포함하도록 pom.xml 파일을 편집합니다.

    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-sleuth</artifactId>
                <version>${spring-cloud-sleuth.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
    </dependencies>
    ```

1. Azure Spring Cloud 서비스에서 이러한 변경 내용을 반영하도록 다시 빌드하고 배포합니다.

## <a name="modify-the-sample-rate"></a>샘플 속도 수정

샘플 속도를 수정하여 원격 분석 데이터를 수집하는 속도를 변경할 수 있습니다. 예를 들어, 절반을 자주 샘플링하려면 application.properties 파일을 열어서 다음 줄을 변경합니다.

```xml
spring.sleuth.sampler.probability=0.5
```

애플리케이션을 이미 빌드하고 배포한 경우, 샘플 속도를 수정할 수 있습니다. Azure CLI나 Azure Portal에서 이전 행을 환경 변수로 추가하면 됩니다.

## <a name="enable-application-insights"></a>Application Insights 사용

1. Azure Portal에서 Azure Spring Cloud 서비스 페이지로 이동합니다.
1. **모니터링**  페이지에서 **분산 추적**을 선택합니다.
1. **설정 편집**을 선택하여 새 설정을 편집하거나 추가합니다.
1. 새 Application Insights 쿼리를 만들거나 기존 쿼리를 선택합니다.
1. 모니터링할 로깅 범주를 선택하고 보존 시간(일)을 지정합니다.
1. **적용**을 선택하여 추적을 적용합니다.

## <a name="view-the-application-map"></a>애플리케이션 맵 보기

**분산 추적** 페이지로 돌아가서 **애플리케이션 맵 보기**를 선택합니다. 애플리케이션 및 모니터링 설정의 시각적 표현을 검토합니다. 애플리케이션 맵 사용 방법을 알아보려면 [애플리케이션 맵: 분산 애플리케이션 심사](https://docs.microsoft.com/azure/azure-monitor/app/app-map)를 참조하세요.

## <a name="use-search"></a>검색 사용

검색 함수를 사용하여 기타 특정한 원격 분석 항목을 쿼리할 수 있습니다. **분산 추적** 페이지에서 **검색**을 선택합니다. 검색 함수를 사용하는 방법에 대한 자세한 내용은 [Application Insights에서 검색 사용](https://docs.microsoft.com/azure/azure-monitor/app/diagnostic-search)을 참조하세요.

## <a name="use-application-insights"></a>Application Insights 사용

Application Insights는 애플리케이션 맵과 검색 기능 외에도 모니터링 기능을 제공합니다. Azure Portal에서 애플리케이션 이름을 검색한 다음, Application Insights 페이지를 열어서 모니터링 정보를 찾을 수 있습니다. 이러한 도구를 사용하는 방법에 대한 자세한 내용은 [Azure Monitor 로그 쿼리](https://docs.microsoft.com/azure/azure-monitor/log-query/query-language)를 확인하세요.

## <a name="disable-application-insights"></a>Application Insights를 사용하지 않도록 설정

1. Azure Portal에서 Azure Spring Cloud 서비스 페이지로 이동합니다.
1. **모니터링**에서 **분산 추적**을 선택합니다.
1. **사용 안 함**을 선택하여 Application Insights를 사용하지 않도록 설정합니다.

## <a name="next-steps"></a>다음 단계

이 자습서에서는 Azure Spring Cloud에서 분산 추적을 사용하도록 설정하고 이해하는 방법을 알아보았습니다. 애플리케이션을 Azure Cosmos DB 데이터베이스에 바인딩하는 방법을 알아보려면 다음 자습서를 계속 진행하세요.

> [!div class="nextstepaction"]
> [Azure Cosmos DB 데이터베이스에 바인딩하는 방법 알아보기](spring-cloud-tutorial-bind-cosmos.md)
