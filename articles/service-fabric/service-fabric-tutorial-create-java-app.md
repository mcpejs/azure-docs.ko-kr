---
title: '자습서: Azure Service Fabric에서 Java 앱 만들기'
description: 이 자습서에서는 프런트 엔드를 사용하여 신뢰할 수 있는 서비스 Java 애플리케이션을 만들고 신뢰할 수 있는 상태 저장 백 엔드를 만들고 애플리케이션을 클러스터에 배포하는 방법을 알아봅니다.
author: suhuruli
ms.topic: tutorial
ms.date: 09/01/2018
ms.author: suhuruli
ms.custom: mvc, seo-java-july2019, seo-java-august2019, seo-java-september2019
ms.openlocfilehash: 019e50057497c9f98d303a93dfa3f905226fa246
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/24/2020
ms.locfileid: "75465458"
---
# <a name="tutorial-create-an-application-with-a-java-api-front-end-service-and-a-stateful-back-end-service-on-azure-service-fabric"></a>자습서: Azure Service Fabric에서 Java API 프런트 엔드 서비스 및 상태 저장 백 엔드 서비스로 애플리케이션 만들기

이 자습서는 시리즈의 1부입니다. 완료하면, Azure Service Fabric의 상태 저장 백 엔드 서비스에 투표 결과를 저장하는 Java 웹 프런트 엔드가 있는 Voting 애플리케이션이 생깁니다. 이 자습서 시리즈는 작업 중인 Linux 또는 Mac OSX 개발자 컴퓨터가 있어야 합니다. 수동으로 투표 애플리케이션을 만들지 않으려면 완성된 애플리케이션에서 [소스 코드를 다운로드](https://github.com/Azure-Samples/service-fabric-java-quickstart)하고 [투표 샘플 애플리케이션을 설명](service-fabric-tutorial-create-java-app.md#walk-through-the-voting-sample-application)하기 위해 바로 건너뛸 수 있습니다. 또한 [Java Reliable Services 빠른 시작](service-fabric-quickstart-java-reliable-services.md)을 따르는 것이 좋습니다.

![Service Fabric 투표 샘플](./media/service-fabric-tutorial-create-java-app/service-fabric-java-voting-app-sample.png)

이 자습서 시리즈에서는 다음 방법에 대해 알아봅니다.
> [!div class="checklist"]
> * Java Service Fabric Reliable Services 애플리케이션 빌드
> * [로컬 클러스터에서 애플리케이션 배포 및 디버그](service-fabric-tutorial-debug-log-local-cluster.md)
> * [Azure 클러스터에 애플리케이션 배포](service-fabric-tutorial-java-deploy-azure.md)
> * [애플리케이션에 대한 모니터링 및 진단 설정](service-fabric-tutorial-java-elk.md)
> * [CI/CD를 설정합니다](service-fabric-tutorial-java-jenkins.md).


시리즈 1부에서는 다음 방법에 대해 알아봅니다.

> [!div class="checklist"]
> * Java 상태 저장 신뢰할 수 있는 서비스 만들기
> * Java 상태 비저장 웹 애플리케이션 서비스 만들기
> * 원격 서비스를 사용하여 상태 저장 서비스와 통신
> * 로컬 Service Fabric 클러스터에 애플리케이션 배포

## <a name="prerequisites"></a>사전 요구 사항

이 자습서를 시작하기 전에:

* Azure 구독이 아직 없는 경우 [체험 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.
* [Mac](service-fabric-get-started-mac.md) 또는 [Linux](service-fabric-get-started-linux.md)용 개발 환경을 설정합니다. 지침에 따라 Eclipse 플러그 인, Gradle, Service Fabric SDK 및 Service Fabric CLI(sfctl)를 설치합니다.

## <a name="create-the-front-end-java-stateless-service"></a>프런트 엔드 Java 상태 비저장 서비스 만들기

먼저 Voting 애플리케이션의 웹 프런트 엔드를 만듭니다. AngularJS 제공 웹 UI는 경량 HTTP 서버를 실행하는 Java 상태 비저장 서비스에 요청을 보냅니다. 이 서비스는 각 요청을 처리하고 투표를 저장하는 상태 저장 서비스에 원격 프로시저 호출을 보냅니다. 

1. Eclipse를 엽니다.

2. **파일** > **새로 만들기** > **다른** > **Service Fabric** > **Service Fabric 프로젝트**를 사용하여 프로젝트를 만듭니다.

    ![Eclipse의 새 Service Fabric 프로젝트](./media/service-fabric-tutorial-create-java-app/service-fabric-project-wizard.png)

3. **ServiceFabric 프로젝트 마법사** 대화 상자에서 프로젝트 이름을 **Voting**으로 지정하고 **다음**을 선택합니다.

    ![새 서비스 대화 상자에서 Java 상태 비저장 서비스 선택](./media/service-fabric-tutorial-create-java-app/name-service-fabric-project-wizard.png) 

4. **서비스 추가** 페이지에서 **상태 비저장 서비스**를 선택하고 서비스 이름을 **VotingWeb**으로 지정합니다. **마침**을 선택하여 프로젝트를 만듭니다.

    ![Service Fabric 프로젝트에 대한 상태 비저장 서비스 만들기]( ./media/service-fabric-tutorial-create-java-app/add-service-fabric-votingweb-service.png)

    Eclipse는 애플리케이션 및 서비스 프로젝트를 만들고 패키지 탐색기에 표시합니다.

    ![애플리케이션을 만든 후 Eclipse 패키지 탐색기]( ./media/service-fabric-tutorial-create-java-app/eclipse-package-explorer.png)

테이블은 이전 스크린샷에서 패키지 탐색기의 각 항목에 대한 간단한 설명을 제공합니다. 

| **패키지 탐색기 항목** | **설명** |
| --- | --- |
| PublishProfiles | 로컬 및 Azure Service Fabric 클러스터의 프로필 세부 정보를 설명하는 JSON 파일을 포함합니다. 이러한 파일의 내용은 애플리케이션을 배포하는 경우 플러그 인에서 사용합니다. |
| 스크립트 | 클러스터를 사용하여 애플리케이션을 신속하게 관리하려면 명령줄에서 사용될 수 있는 도우미 스크립트를 포함합니다. |
| VotingApplication | Service Fabric 클러스터에 밀어넣은 Service Fabric 애플리케이션을 포함합니다. |
| VotingWeb | 관련 Gradle 빌드 파일과 함께 프런트 엔드 상태 비저장 서비스 소스 파일을 포함합니다. |
| build.gradle | 프로젝트를 관리하는 데 사용되는 Gradle 파일입니다. |
| settings.gradle | 이 폴더의 Gradle 프로젝트의 이름을 포함합니다. |

### <a name="add-html-and-javascript-to-the-votingweb-service"></a>HTML 및 Javascript를 VotingWeb 서비스에 추가

상태 비저장 서비스로 렌더링할 수 있는 UI를 추가하려면 HTML 파일을 추가합니다. 이 HTML 파일은 상태 비저장 Java 서비스에 포함된 경량 HTTP 서버에서 렌더링합니다.

1. *VotingApplication* 디렉터리를 확장해 *VotingApplication/VotingWebPkg/Code* 디렉터리에 이르게 합니다.

2. *코드* 디렉터리를 마우스 오른쪽 단추로 클릭하고 **새로 만들기** > **폴더**를 선택합니다.

3. 폴더의 이름을 *wwwroot*로 지정하고 **마침**을 선택합니다.

    ![Eclipse로 wwwroot 폴더 만들기](./media/service-fabric-tutorial-create-java-app/create-wwwroot-folder.png)

4. **index.html**이라는 **wwwroot**에 파일을 추가하고 이 파일에 다음 내용을 붙여넣습니다.

```html
<!DOCTYPE html>
<html>
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/0.13.4/ui-bootstrap-tpls.min.js"></script>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
<body>

<script>
var app = angular.module('VotingApp', ['ui.bootstrap']);
app.controller("VotingAppController", ['$rootScope', '$scope', '$http', '$timeout', function ($rootScope, $scope, $http, $timeout) {
    $scope.votes = [];
    
    $scope.refresh = function () {
        $http.get('getStatelessList')
            .then(function successCallback(response) {
        $scope.votes = Object.assign(
            {},
            ...Object.keys(response.data) .
            map(key => ({[decodeURI(key)]: response.data[key]}))
        )
        },
        function errorCallback(response) {
            alert(response);
        });
    };

    $scope.remove = function (item) {
       $http.get("removeItem", {params: { item: encodeURI(item) }})
            .then(function successCallback(response) {
                $scope.refresh();
            },
            function errorCallback(response) {
                alert(response);
            });
    };

    $scope.add = function (item) {
        if (!item) {return;}
        $http.get("addItem", {params: { item: encodeURI(item) }})
            .then(function successCallback(response) {
                $scope.refresh();
            },
            function errorCallback(response) {
                alert(response);
            });
    };
}]);
</script>

<div ng-app="VotingApp" ng-controller="VotingAppController" ng-init="refresh()">
    <div class="container-fluid">
        <div class="row">
            <div class="col-xs-8 col-xs-offset-2 text-center">
                <h2>Service Fabric Voting Sample</h2>
            </div>
        </div>

        <div class="row">
            <div class="col-xs-offset-2">
                <form style="width:50% ! important;" class="center-block">
                    <div class="col-xs-6 form-group">
                        <input id="txtAdd" type="text" class="form-control" placeholder="Add voting option" ng-model="item" />
                    </div>
                    <button id="btnAdd" class="btn btn-default" ng-click="add(item)">
                        <span class="glyphicon glyphicon-plus" aria-hidden="true"></span>
                        Add
                    </button>
                </form>
            </div>
        </div>

        <hr />

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <div class="row">
                    <div class="col-xs-4">
                        Click to vote
                    </div>
                </div>
                <div class="row top-buffer" ng-repeat="(key, value)  in votes">
                    <div class="col-xs-8">
                        <button class="btn btn-success text-left btn-block" ng-click="add(key)">
                            <span class="pull-left">
                                {{key}}
                            </span>
                            <span class="badge pull-right">
                                {{value}} Votes
                            </span>
                        </button>
                    </div>
                    <div class="col-xs-4">
                        <button class="btn btn-danger pull-right btn-block" ng-click="remove(key)">
                            <span class="glyphicon glyphicon-remove" aria-hidden="true"></span>
                            Remove
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

</body>
</html>
```

### <a name="update-the-votingwebjava-file"></a>VotingWeb.java 파일 업데이트

**VotingWeb** 하위 프로젝트에서 *VotingWeb/src/statelessservice/VotingWeb.java* 파일을 엽니다. **VotingWeb** 서비스는 상태 비저장 서비스에 대한 게이트웨이이며 프런트 엔드 API에 대한 통신 수신기 설정을 책임집니다.

해당 파일에서 기존 **createServiceInstanceListeners** 메서드를 다음으로 바꾸고 변경 내용을 저장합니다.

```java
@Override
protected List<ServiceInstanceListener> createServiceInstanceListeners() {

    EndpointResourceDescription endpoint = this.getServiceContext().getCodePackageActivationContext().getEndpoint(webEndpointName);
    int port = endpoint.getPort();

    List<ServiceInstanceListener> listeners = new ArrayList<ServiceInstanceListener>();
    listeners.add(new ServiceInstanceListener((context) -> new HttpCommunicationListener(context, port)));
    return listeners;
}
```

### <a name="add-the-httpcommunicationlistenerjava-file"></a>HTTPCommunicationListener.java 파일 추가

HTTP 통신 수신기는 HTTP 서버를 설정하고 컨트롤러로서 투표 작업을 정의하는 API를 공개하는 역할을 합니다. *VotingWeb/src/statelessservice* 폴더에서 *statelessservice* 패키지를 마우스 오른쪽 단추로 클릭한 다음, **새로 만들기** > **파일**을 선택합니다.  *HttpCommunicationListener.java* 파일 이름을 지정하고 **마침**을 선택합니다.

파일 내용을 다음으로 바꾸고 변경 내용을 저장합니다.  나중에 이 파일은 HttpCommunicationListener.java 파일 업데이트에서 백 엔드 서비스의 투표 데이터를 렌더링하고 읽고 쓰도록 수정됩니다.  지금은 수신기가 Voting 앱에 대한 고정적인 HTML을 반환합니다.

```java
// ------------------------------------------------------------
//  Copyright (c) Microsoft Corporation.  All rights reserved.
//  Licensed under the MIT License (MIT). See License.txt in the repo root for license information.
// ------------------------------------------------------------

package statelessservice;

import com.google.gson.Gson;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.Headers;

import java.io.File;
import java.io.OutputStream;
import java.io.FileInputStream;

import java.net.InetSocketAddress;
import java.net.URI;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CompletableFuture;
import java.util.logging.Level;
import java.util.logging.Logger;

import microsoft.servicefabric.services.communication.runtime.CommunicationListener;
import microsoft.servicefabric.services.runtime.StatelessServiceContext;
import microsoft.servicefabric.services.client.ServicePartitionKey;
import microsoft.servicefabric.services.remoting.client.ServiceProxyBase;
import microsoft.servicefabric.services.communication.client.TargetReplicaSelector;
import system.fabric.CancellationToken;

public class HttpCommunicationListener implements CommunicationListener {

    private static final Logger logger = Logger.getLogger(HttpCommunicationListener.class.getName());

    private static final String HEADER_CONTENT_TYPE = "Content-Type";
    private static final int STATUS_OK = 200;
    private static final int STATUS_NOT_FOUND = 404;
    private static final int STATUS_ERROR = 500;
    private static final String RESPONSE_NOT_FOUND = "404 (Not Found) \n";
    private static final String MIME = "text/html";
    private static final String ENCODING = "UTF-8";

    private static final String ROOT = "wwwroot/";
    private static final String FILE_NAME = "index.html";
    private StatelessServiceContext context;
    private com.sun.net.httpserver.HttpServer server;
    private ServicePartitionKey partitionKey;
    private final int port;

    public HttpCommunicationListener(StatelessServiceContext context, int port) {
        this.partitionKey = new ServicePartitionKey(0);
        this.context = context;
        this.port = port;
    }

    // Called by openAsync when the class is instantiated
    public void start() {
        try {
            logger.log(Level.INFO, "Starting Server");
            server = com.sun.net.httpserver.HttpServer.create(new InetSocketAddress(this.port), 0);
        } catch (Exception ex) {
            logger.log(Level.SEVERE, null, ex);
            throw new RuntimeException(ex);
        }

        // Responsible for rendering the HTML layout described in the previous step
        server.createContext("/", new HttpHandler() {
            @Override
            public void handle(HttpExchange t) {
                try {
                    File file = new File(ROOT + FILE_NAME).getCanonicalFile();

                    if (!file.isFile()) {
                      // Object does not exist or is not a file: reject with 404 error.
                      t.sendResponseHeaders(STATUS_NOT_FOUND, RESPONSE_NOT_FOUND.length());
                      OutputStream os = t.getResponseBody();
                      os.write(RESPONSE_NOT_FOUND.getBytes());
                      os.close();
                    } else {
                      Headers h = t.getResponseHeaders();
                      h.set(HEADER_CONTENT_TYPE, MIME);
                      t.sendResponseHeaders(STATUS_OK, 0);
    
                      OutputStream os = t.getResponseBody();
                      FileInputStream fs = new FileInputStream(file);
                      final byte[] buffer = new byte[0x10000];
                      int count = 0;
                      while ((count = fs.read(buffer)) >= 0) {
                        os.write(buffer,0,count);
                      }

                      fs.close();
                      os.close();
                    }
                } catch (Exception e) {
                    logger.log(Level.WARNING, null, e);
                }
            }
        });

        /*
        [Replace this entire comment block in the 'Connect the services' section]
        */

        server.setExecutor(null);
        server.start();
    }

    //Helper method to parse raw HTTP requests
    private Map<String, String> queryToMap(String query){
        Map<String, String> result = new HashMap<String, String>();
        for (String param : query.split("&")) {
            String pair[] = param.split("=");
            if (pair.length>1) {
                result.put(pair[0], pair[1]);
            }else{
                result.put(pair[0], "");
            }
        }
        return result;
    }

    //Called closeAsync when the service is shut down
    private void stop() {
        if (null != server)
            server.stop(0);
    }

    //Called by the Service Fabric runtime when this service is created on a node
    @Override
    public CompletableFuture<String> openAsync(CancellationToken cancellationToken) {
        this.start();
                    logger.log(Level.INFO, "Opened Server");
        String publishUri = String.format("http://%s:%d/", this.context.getNodeContext().getIpAddressOrFQDN(), port);
        return CompletableFuture.completedFuture(publishUri);
    }

    //Called by the Service Fabric runtime when the service is shut down
    @Override
    public CompletableFuture<?> closeAsync(CancellationToken cancellationToken) {
        this.stop();
        return CompletableFuture.completedFuture(true);
    }

    //Called by the Service Fabric runtime to forcibly shut this listener down
    @Override
    public void abort() {
        this.stop();
    }
}
```

### <a name="configure-the-listening-port"></a>수신 대기 포트 구성

VotingWeb 서비스 프런트 엔드 서비스를 만들면 Service Fabric에서는 수신할 서비스에 대한 포트를 선택합니다.  VotingWeb 서비스는 이 애플리케이션에 대한 프런트 엔드로 작동하고 외부 트래픽을 허용하므로 이 서비스를 고정된 잘 알려진 포트에 바인딩하겠습니다. 패키지 탐색기에서 *VotingApplication/VotingWebPkg/ServiceManifest.xml*을 엽니다.  **리소스** 섹션에서 **엔드포인트** 리소스를 찾아 **포트** 값을 8080으로 변경합니다(자습서 전체에서 이 포트를 계속 사용). 애플리케이션을 로컬로 배포하고 실행하려면 애플리케이션 수신 대기 포트가 열려 있고 컴퓨터에서 사용할 수 있어야 합니다. **ServiceManifest** 요소 내에서 다음 코드 조각을 붙여넣습니다(예: ```<DataPackage>``` 요소 바로 아래).

```xml
<Resources>
    <Endpoints>
        <!-- This endpoint is used by the communication listener to obtain the port on which to
            listen. Please note that if your service is partitioned, this port is shared with
            replicas of different partitions that are placed in your code. -->
        <Endpoint Name="WebEndpoint" Protocol="http" Port="8080" />
    </Endpoints>
  </Resources>
```

## <a name="add-a-stateful-back-end-service-to-your-application"></a>상태 저장 백 엔드 서비스를 애플리케이션에 추가

이제 Java Web API 서비스의 기본 구조가 완료되면 계속해서 상태 저장 백 엔드 서비스를 완료합니다.

Service Fabric을 통해 신뢰할 수 있는 컬렉션을 사용하여 일관되고 안정적으로 서비스 내에 데이터를 바로 저장할 수 있습니다. 신뢰할 수 있는 컬렉션은 고가용성의 신뢰할 수 있는 컬렉션 클래스의 집합입니다. Java 컬렉션을 사용하는 사람에게는 이러한 클래스의 사용이 친숙합니다.

1. 패키지 탐색기에서 애플리케이션 프로젝트 내의 **Voting**을 마우스 오른쪽 단추로 클릭하고 **Service Fabric** > **Service Fabric 서비스 추가**를 선택합니다.

2. **서비스 추가** 대화 상자에서 **상태 저장 서비스**를 선택하고 서비스 이름을 **VotingDataService**로 지정하고 **서비스 추가**를 선택합니다.

    서비스 프로젝트를 만들었으면 애플리케이션에 두 개의 서비스가 있을 것입니다. 계속해서 애플리케이션을 빌드하는 동안 같은 방법으로 더 많은 서비스를 추가할 수 있습니다. 각 서비스를 독립적으로 버전 지정 및 업그레이드할 수 있습니다.

3. Eclipse는 서비스 프로젝트를 만들고 패키지 탐색기에 표시합니다.

    ![Eclipse 프로젝트 탐색기](./media/service-fabric-tutorial-create-java-app/service-fabric-package-explorer-java.png)

### <a name="add-the-votingdataservicejava-file"></a>VotingDataService.java 파일 추가

*VotingDataService.java* 파일은 신뢰할 수 있는 컬렉션에서 투표를 검색하고 추가하고 제거하기 위한 논리를 포함하는 메서드를 포함합니다. *VotingDataService/src/statefulservice/VotingDataService.java* 파일에 다음 **VotingDataService** 클래스 메서드를 추가합니다.

```java
package statefulservice;

import java.util.HashMap;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.atomic.AtomicInteger;

import microsoft.servicefabric.services.communication.runtime.ServiceReplicaListener;

import microsoft.servicefabric.services.runtime.StatefulService;

import microsoft.servicefabric.services.remoting.fabrictransport.runtime.FabricTransportServiceRemotingListener;

import microsoft.servicefabric.data.ReliableStateManager;
import microsoft.servicefabric.data.Transaction;
import microsoft.servicefabric.data.collections.ReliableHashMap;
import microsoft.servicefabric.data.utilities.AsyncEnumeration;
import microsoft.servicefabric.data.utilities.KeyValuePair;

import system.fabric.StatefulServiceContext;

import rpcmethods.VotingRPC;

class VotingDataService extends StatefulService implements VotingRPC {
    private static final String MAP_NAME = "votesMap";
    private ReliableStateManager stateManager;

    protected VotingDataService (StatefulServiceContext statefulServiceContext) {
        super (statefulServiceContext);
    }

    @Override
    protected List<ServiceReplicaListener> createServiceReplicaListeners() {
        this.stateManager = this.getReliableStateManager();
        ArrayList<ServiceReplicaListener> listeners = new ArrayList<>();

        listeners.add(new ServiceReplicaListener((context) -> {
            return new FabricTransportServiceRemotingListener(context,this);
        }));

        return listeners;
    }

    // Method that will be invoked via RPC from the front end to retrieve the complete set of votes in the map
    public CompletableFuture<HashMap<String,String>> getList() {
        HashMap<String, String> tempMap = new HashMap<String, String>();

        try {

            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();

            Transaction tx = stateManager.createTransaction();
            AsyncEnumeration<KeyValuePair<String, String>> kv = votesMap.keyValuesAsync(tx).get();
            while (kv.hasMoreElementsAsync().get()) {
                KeyValuePair<String, String> k = kv.nextElementAsync().get();
                tempMap.put(k.getKey(), k.getValue());
            }

            tx.close();


        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(tempMap);
    }

    // Method that will be invoked via RPC from the front end to add an item to the votes list or to increase the
    // vote count for a particular item
    public CompletableFuture<Integer> addItem(String itemToAdd) {
        AtomicInteger status = new AtomicInteger(-1);

        try {

            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();

            Transaction tx = stateManager.createTransaction();
            votesMap.computeAsync(tx, itemToAdd, (k, v) -> {
                if (v == null) {
                    return "1";
                }
                else {
                    int numVotes = Integer.parseInt(v);
                    numVotes = numVotes + 1;
                    return Integer.toString(numVotes);
                }
            }).get();

            tx.commitAsync().get();
            tx.close();

            status.set(1);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(new Integer(status.get()));
    }

    // Method that will be invoked via RPC from the front end to remove an item
    public CompletableFuture<Integer> removeItem(String itemToRemove) {
        AtomicInteger status = new AtomicInteger(-1);
        try {
            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();

            Transaction tx = stateManager.createTransaction();
            votesMap.removeAsync(tx, itemToRemove).get();
            tx.commitAsync().get();
            tx.close();

            status.set(1);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(new Integer(status.get()));
    }

}
```

프런트 엔드 상태 비저장 서비스 및 백 엔드 서비스에 대한 기본 구성을 만듭니다.

## <a name="create-the-communication-interface-to-your-application"></a>애플리케이션의 통신 인터페이스 만들기

 다음 단계에서는 프런트 엔드 상태 비저장 서비스 및 백 엔드 서비스를 연결합니다. 두 서비스 모두 Voting 애플리케이션의 작업을 정의하는 VotingRPC라는 인터페이스를 사용합니다. 이 인터페이스는 두 서비스 간의 원격 프로시저 호출(RPC)을 사용할 수 있도록 프런트 엔드 및 백 엔드 서비스에 의해 구현됩니다. 안타깝게도 Eclipse는 Gradle 하위 프로젝트를 추가하는 것을 지원하지 않으므로 이 인터페이스를 포함하는 패키지를 수동으로 추가해야 합니다.

1. 패키지 탐색기에서 **Voting** 프로젝트를 마우스 오른쪽 단추로 클릭하고 **새로 만들기** > **폴더**를 선택합니다. 폴더의 이름을 **VotingRPC/src/rpcmethods**로 지정합니다.

    ![Eclipse 패키지 탐색기에서 VotingRPC 패키지 만들기](./media/service-fabric-tutorial-create-java-app/create-voting-rpc-package-java.png)

3. *VotingRPC.java*라는 *Voting/VotingRPC/src/rpcmethods* 아래에 파일을 만들고 **VotingRPC.java** 파일 내부에 다음을 붙여넣습니다. 

    ```java
    package rpcmethods;
    
    import java.util.ArrayList;
    import java.util.concurrent.CompletableFuture;
    import java.util.List;
    import java.util.HashMap;
    
    import microsoft.servicefabric.services.remoting.Service;
    
    public interface VotingRPC extends Service {
        CompletableFuture<HashMap<String, String>> getList();
    
        CompletableFuture<Integer> addItem(String itemToAdd);
    
        CompletableFuture<Integer> removeItem(String itemToRemove);
    }
    ```

4. *Voting/VotingRPC* 디렉터리에 *build.gradle*이라는 빈 파일을 만들고 파일 내부에 다음을 붙여넣습니다. 이 Gradle 파일은 다른 서비스에서 가져온 jar 파일을 빌드하고 만드는 데 사용됩니다. 

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/out"
        delete "${projectDir}/VotingRPC.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0')
    }
    
    jar {
        from configurations.compile.collect {
            (it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}

        manifest {
                attributes(
                'Main-Class': 'rpcmethods.VotingRPC')
            baseName "VotingRPC"
            destinationDir = file('./')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA'
    }

    defaultTasks 'clean', 'jar'
    ```

5. *Voting/settings.gradle* 파일에서 선을 추가하여 빌드에 새로 만든 프로젝트를 포함합니다. 

    ```gradle
    include ':VotingRPC'
    ```

6. *Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java* 파일에서 주석 블록을 다음으로 바꿉니다.  

    ```java
    server.createContext("/getStatelessList", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                t.sendResponseHeaders(STATUS_OK,0);
                OutputStream os = t.getResponseBody();
    
                HashMap<String,String> list = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").getList().get();
                String json = new Gson().toJson(list);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
        }
    });
    
    server.createContext("/removeItem", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                OutputStream os = t.getResponseBody();
                URI r = t.getRequestURI();
    
                Map<String, String> params = queryToMap(r.getQuery());
                String itemToRemove = params.get("item");

                Integer num = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").removeItem(itemToRemove).get();
    
                if (num != 1)
                {
                    t.sendResponseHeaders(STATUS_ERROR, 0);
                } else {
                    t.sendResponseHeaders(STATUS_OK,0);
                }
    
                String json = new Gson().toJson(num);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
    
        }
    });
    
    server.createContext("/addItem", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                URI r = t.getRequestURI();
                Map<String, String> params = queryToMap(r.getQuery());
                String itemToAdd = params.get("item");

                OutputStream os = t.getResponseBody();
                Integer num = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").addItem(itemToAdd).get();
                if (num != 1)
                {
                    t.sendResponseHeaders(STATUS_ERROR, 0);
                } else {
                    t.sendResponseHeaders(STATUS_OK,0);
                }
    
                String json = new Gson().toJson(num);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
        }
    });
    ```
7. *Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java* 파일 상단에 적절한 가져오기 문을 추가합니다. 

    ```java
    import rpcmethods.VotingRPC; 
    ```

이 단계에서 프런트 엔드와 백 엔드, RPC 인터페이스에 대한 기능이 완료됩니다. 다음 단계는 Service Fabric 클러스터에 배포하기 전에 Gradle 스크립트를 적절하게 구성하는 것입니다. 

## <a name="walk-through-the-voting-sample-application"></a>투표 애플리케이션 예제 연습
투표 애플리케이션은 두 가지 서비스로 구성됩니다.
- 웹 프런트 엔드 서비스(VotingWeb) - Java 웹 프런트 엔드 서비스로, 웹 페이지를 제공하며 백 엔드 서비스와 통신하기 위한 API를 공개합니다.
- 백 엔드 서비스(VotingDataService) - Java 웹 서비스로, 투표를 유지하도록 원격 프로시저 호출(RPC)을 통해 호출되는 메서드를 정의합니다.

![투표 샘플 다이어그램](./media/service-fabric-tutorial-create-java-app/walkthrough-java-voting.png)

애플리케이션에서 작업을 수행하는 경우(항목 추가, 투표, 항목 제거) 다음 이벤트가 발생합니다.
1. JavaScript가 적절한 요청을 웹 프런트 엔드 서비스의 Web API에 HTTP 요청으로 보냅니다.

2. 웹 프런트 엔드 서비스는 백 엔드 서비스에 대한 요청을 찾아 전달하기 위해 Service Fabric의 기본 제공 원격 서비스 기능을 사용합니다. 

3. 백 엔드 서비스는 신뢰할 수 있는 사전에서 결과를 업데이트하는 메서드를 정의합니다. 이 신뢰할 수 있는 사전의 내용은 클러스터 내에서 여러 노드에 복제되고 디스크에 저장됩니다. 애플리케이션의 모든 데이터가 클러스터에 저장됩니다. 

## <a name="configure-gradle-scripts"></a>Gradle 스크립트 구성 

이 섹션에서는 프로젝트에 대한 Gradle 스크립트가 구성됩니다. 

1. *Voting/build.gradle* 파일 내용을 다음으로 바꿉니다.

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    subprojects {
        apply plugin: 'java'
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

2. *Voting/VotingWeb/build.gradle* 파일 내용을 다음으로 바꿉니다.

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/../lib"
        delete "${projectDir}/out"
        delete "${projectDir}/../VotingApplication/VotingWebPkg/Code/lib"
        delete "${projectDir}/../VotingApplication/VotingWebPkg/Code/VotingWeb.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0-preview1')
        compile project(':VotingRPC')
    }
    
    task explodeDeps(type: Copy, dependsOn:configurations.compile) { task ->
        configurations.compile.filter {!it.toString().contains("native")}.each{
            from it
        }
    
        configurations.compile.filter {it.toString().contains("native")}.each{
            from zipTree(it)
        }
        into "../lib/"
        include "lib*.so", "*.jar"
    }
    
    task copyDeps<< {
        copy {
            from("../lib/")
            into("../VotingApplication/VotingWebPkg/Code/lib")
            include('lib*.so')
        }
    }
    
    compileJava.dependsOn(explodeDeps)
    
    jar {
        from configurations.compile.collect {(it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}
    
        manifest {
            attributes(
                'Main-Class': 'statelessservice.VotingWebServiceHost')
            baseName "VotingWeb"
            destinationDir = file('../VotingApplication/VotingWebPkg/Code/')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA'
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

3. *Voting/VotingDataService/build.gradle* 파일 내용을 바꿉니다. 

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/../lib"
        delete "${projectDir}/out"
        delete "${projectDir}/../VotingApplication/VotingDataServicePkg/Code/lib"
        delete "${projectDir}/../VotingApplication/VotingDataServicePkg/Code/VotingDataService.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0-preview1')
        compile project(':VotingRPC')
    }
    
    task explodeDeps(type: Copy, dependsOn:configurations.compile) { task ->
        configurations.compile.filter {!it.toString().contains("native")}.each{
            from it
        }
    
        configurations.compile.filter {it.toString().contains("native")}.each{
            from zipTree(it)
        }
        into "../lib/"
        include "lib*.so", "*.jar"
    }
    
    compileJava.dependsOn(explodeDeps)
    
    task copyDeps<< {
        copy {
            from("../lib/")
            into("../VotingApplication/VotingDataServicePkg/Code/lib")
            include('lib*.so')
        }
    }
    
    jar {
        from configurations.compile.collect {
            (it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}
    
        manifest {
            attributes('Main-Class': 'statefulservice.VotingDataServiceHost')
    
            baseName "VotingDataService"
            destinationDir = file('../VotingApplication/VotingDataServicePkg/Code/')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA'
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

## <a name="deploy-application-to-local-cluster"></a>로컬 클러스터에 애플리케이션 배포

이 지점에서 애플리케이션은 로컬 Service Fabric 클러스터에 배포될 준비가 됐습니다.

1. 패키지 탐색기에서 **Voting** 프로젝트를 마우스 오른쪽 단추로 클릭하고 **Service Fabric** > **애플리케이션 빌드**를 선택하여 애플리케이션을 빌드합니다.

2. 로컬 Service Fabric 클러스터를 실행합니다. 이 단계는 개발 환경(Mac 또는 Linux)에 따라 달라집니다.

    Mac을 사용하는 경우 다음 명령을 사용하여 로컬 클러스터를 실행합니다. **-v** 매개 변수로 전달된 명령을 사용자의 작업 영역 경로로 바꿉니다.

    ```bash
    docker run -itd -p 19080:19080 -p 8080:8080 -p --name sfonebox servicefabricoss/service-fabric-onebox
    ```
    [OS X 설치 가이드](service-fabric-get-started-mac.md)에서 자세한 지침을 확인하세요.

    Linux 컴퓨터에서 실행하는 경우 다음 명령을 사용하여 로컬 클러스터를 시작합니다. 

    ```bash 
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```
    [Linux 설치 가이드](service-fabric-get-started-linux.md)에서 자세한 지침을 확인하세요.

4. Eclipse용 패키지 탐색기에서 **Voting** 프로젝트를 마우스 오른쪽 단추로 클릭하고 **Service Fabric** > **애플리케이션 게시**를 선택 
5. **애플리케이션 게시** 창의 드롭다운에서 **Local.json**을 선택하고, **게시**를 선택합니다.
6. 웹 브라우저로 이동하고 http:\//localhost:8080에 액세스하여 로컬 Service Fabric 클러스터에서 실행 중인 애플리케이션을 봅니다. 

## <a name="next-steps"></a>다음 단계

자습서의 이 부분에서는 다음 방법에 대해 알아봅니다.

> [!div class="checklist"]
> * Java 서비스를 상태 저장 신뢰할 수 있는 서비스로 만들기
> * Java 서비스를 상태 비저장 신뢰할 수 있는 서비스로 만들기
> * 서비스 간 원격 프로시저 호출(RPC)을 처리하려면 Java 인터페이스 추가
> * Gradle 스크립트 구성
> * 로컬 Service Fabric 클러스터에 애플리케이션 빌드 및 배포

다음 자습서를 진행합니다.
> [!div class="nextstepaction"]
> [로컬 클러스터에서 애플리케이션 로그 및 디버그](service-fabric-tutorial-debug-log-local-cluster.md)
