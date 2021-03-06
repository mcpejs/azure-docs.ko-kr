---
title: AES-128로 비디오 암호화
titleSuffix: Azure Media Services
description: AES 128비트 암호화로 비디오를 암호화하는 방법과 Azure Media 서비스에서 키 배달 서비스를 사용하는 방법에 대해 알아봅니다.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/21/2019
ms.author: juliako
ms.openlocfilehash: 126700e6290650221a9cb9711b22472301409fca
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74974175"
---
# <a name="tutorial-encrypt-video-with-aes-128-and-use-the-key-delivery-service"></a>자습서: AES-128로 비디오를 암호화하고 키 배달 서비스를 사용합니다.

> [!NOTE]
> 이 자습서에서 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet) 예제를 사용하더라도 일반적인 단계는 [REST API](https://docs.microsoft.com/rest/api/media/liveevents), [CLI](https://docs.microsoft.com/cli/azure/ams/live-event?view=azure-cli-latest) 또는 지원되는 기타 [SDK](media-services-apis-overview.md#sdks)에 대해 동일합니다.

Media Services에서는 128비트 암호화 키를 사용하여 AES로 암호화된 HLS(HTTP 라이브 스트리밍), MPEG-DASH 및 부드러운 스트리밍을 배달할 수 있습니다. Media Services는 권한 있는 사용자에게 암호화 키를 제공하는 키 배달 서비스도 제공합니다. Media Services에서 비디오를 동적으로 암호화하려면 암호화 키를 스트리밍 로케이터와 연결하고 콘텐츠 키 정책을 구성할 수도 있습니다. 플레이어가 스트림을 요청하면 Media Services는 지정된 키를 사용하여 AES-128로 콘텐츠를 동적으로 암호화합니다. 스트림을 해독하기 위해 플레이어는 키 배달 서비스에서 키를 요청합니다. 사용자에게 키를 얻을 수 있는 권한이 있는지 여부를 결정하기 위해 서비스는 키에 지정된 콘텐츠 키 정책을 평가합니다.

여러 암호화 유형(AES-128, PlayReady, Widevine, FairPlay)을 사용하여 각 자산을 암호화할 수 있습니다. 결합에 적합한 것을 확인하려면 [스트리밍 프로토콜 및 암호화 유형](content-protection-overview.md#streaming-protocols-and-encryption-types)을 참조합니다. 또한 [DRM으로 보호하는 방법을](protect-with-drm.md)참조하십시오.

이 문서의 샘플 출력에는 Azure Media Player에 대한 URL, 매니페스트 URL 및 콘텐츠를 재생하는 데 필요한 AES 토큰이 포함됩니다. 이 샘플에서는 JSON 웹 토큰(JWT) 토큰의 만료 시간을 1시간으로 설정합니다. 브라우저를 열고 결과 URL을 붙여넣어 이미 다음 형식으로 작성된 토큰 및 URL을 사용하여 Azure Media Player 데모 페이지를 시작할 수 있습니다. ```https://ampdemo.azureedge.net/?url= {dash Manifest URL} &aes=true&aestoken=Bearer%3D{ JWT Token here}```.

이 자습서에서는 다음을 수행하는 방법에 대해 설명합니다.

> [!div class="checklist"]
> * 이 문서에 설명된 [암호화WithAES](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES) 샘플을 다운로드합니다.
> * .NET SDK를 사용하여 미디어 서비스 API를 사용하기 시작합니다.
> * 출력 자산을 만듭니다.
> * 인코딩 변환을 만듭니다.
> * 작업을 제출합니다.
> * 작업이 완료될 때까지 기다립니다.
> * 콘텐츠 키 정책 만들기
> * JWT 토큰 제한을 사용하도록 정책을 구성합니다.
> * 스트리밍 로케이터를 만듭니다.
> * AES(ClearKey)로 비디오를 암호화하도록 스트리밍 로케이터를 구성합니다.
> * 테스트 토큰을 가져옵니다.
> * 스트리밍 URL을 빌드합니다.
> * 리소스를 정리합니다.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>사전 요구 사항

자습서를 완료하는 데 필요한 조건은 다음과 같습니다.

* [콘텐츠 보호 개요](content-protection-overview.md) 문서를 검토합니다.
* Visual Studio Code 또는 Visual Studio 설치
* [Media Services 계정 만들기](create-account-cli-quickstart.md)
* [액세스 API를](access-api-cli-how-to.md)따라 미디어 서비스 API를 사용하는 데 필요한 자격 증명을 가져옵니다.

## <a name="download-code"></a>코드 다운로드

다음 명령을 사용하여 이 항목에서 설명한 전체 .NET 샘플이 포함된 GitHub 리포지토리를 머신에 복제합니다.

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

"AES-128로 암호화" 샘플은 [EncryptWithAES](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES) 폴더에 있습니다.

> [!NOTE]
> 샘플은 앱을 실행할 때마다 고유한 리소스를 만듭니다. 일반적으로 기존 리소스에 필요한 구성이 있는 경우 변환 및 정책과 같은 기존 리소스를 다시 사용합니다.

## <a name="start-using-media-services-apis-with-net-sdk"></a>.NET SDK로 Media Services API 사용하기

.NET을 사용하여 미디어 서비스 API를 사용하려면 **AzureMediaServicesClient** 개체를 만듭니다. 개체를 만들려면 Azure AD를 사용하여 클라이언트가 Azure에 연결하는 데 필요한 자격 증명을 제공해야 합니다. 아티클의 시작 부분에서 복제한 코드에서 **GetCredentialsAsync** 함수는 로컬 구성 파일에 제공된 자격 증명에 따라 ServiceClientCredentials 개체를 만듭니다.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateMediaServicesClient)]

## <a name="create-an-output-asset"></a>출력 자산 만들기  

출력 [자산은](https://docs.microsoft.com/rest/api/media/assets) 인코딩 작업의 결과를 저장합니다.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateOutputAsset)]

## <a name="get-or-create-an-encoding-transform"></a>인코딩 Transform을 가져오거나 만들기

새로운 [Transform](https://docs.microsoft.com/rest/api/media/transforms) 인스턴스를 만드는 경우 이 인스턴스를 통해 출력하려는 것이 무엇인지 지정해야 합니다. 필수 매개 변수는 아래 코드와 같이 **TransformOutput** 개체입니다. 각 **TransformOutput**에는 **Preset**이 포함됩니다. **Preset**은 원하는 **TransformOutput**을 생성하는 데 사용되는 비디오 및/또는 오디오 처리 작업에 대한 단계별 지침을 설명합니다. 이 자습서에서 설명하는 샘플은 **AdaptiveStreaming**이라는 기본 제공 Preset을 사용합니다. Preset은 입력 해상도 및 비트 레이트에 따라 자동 생성 된 비트 레이트 래더 (비트 레이트 해상도 쌍)로 입력 비디오를 인코딩 한 다음 각 비트 레이트 해상도 쌍에 해당하는 H.264 비디오 및 AAC 오디오가있는 ISO MP4 파일을 생성합니다.

새 [Transform을](https://docs.microsoft.com/rest/api/media/transforms)만들기 전에 다음 코드와 같이 **Get** 메서드를 사용하여 이미 존재하는지 먼저 확인합니다. Media Services v3에서 엔터티에 대한 메서드 **를 가져옵니다.엔터티가** 존재하지 않는 경우 **null을** 반환합니다(이름에 대한 대/소문자를 구분하지 않는 검사).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#EnsureTransformExists)]

## <a name="submit-job"></a>작업 제출

위에서 언급했듯이 [Transform](https://docs.microsoft.com/rest/api/media/transforms) 개체는 레시피이며 [Job](https://docs.microsoft.com/rest/api/media/jobs)은 주어진 입력 비디오 또는 오디오 콘텐츠에 **Transform**을 적용하라는 Media Services에 대한 실제 요청입니다. **Job**은 입력 비디오의 위치 및 출력 위치와 같은 정보를 지정합니다.

이 자습서에서는 [HTTPs 소스 URL에서](job-input-from-http-how-to.md)직접 인해 되는 파일을 기반으로 작업의 입력을 만듭니다.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>작업이 완료될 때까지 대기

작업이 완료될 때까지 시간이 조금 걸립니다. 적업이 완료될 때 알림을 받을 수 있습니다. 아래 코드 샘플에서는 [작업](https://docs.microsoft.com/rest/api/media/jobs)의 상태에 대한 서비스를 폴링하는 방법을 보여 주며. 폴링은 대기 시간이 발생할 가능성이 있기 때문에 프로덕션 앱에 권장하는 방법이 아닙니다. 폴링이 계정에서 초과 사용되면 정체될 수 있습니다. 대신 Event Grid를 사용해야 합니다. 자세한 내용은 [이벤트를 사용자 지정 웹 엔드포인트로 라우팅](job-state-events-cli-how-to.md)을 참조하세요.

**작업**은 일반적으로 **예약됨**, **큐에 대기됨**, **처리 중**, **마침**(최종 상태) 상태를 거칩니다. 작업에서 오류가 발생하면 **오류** 상태가 표시됩니다. 작업을 취소 중인 경우 **취소 중**이 표시되고, 취소가 완료되면 **취소됨**이 표시됩니다.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#WaitForJobToFinish)]

## <a name="create-a-content-key-policy"></a>콘텐츠 키 정책 만들기

콘텐츠 키는 자산에 대한 보안 액세스를 제공합니다. 콘텐츠 키가 최종 클라이언트에 전달되는 방법을 구성하는 콘텐츠 **키 정책을** 만들어야 합니다. 콘텐츠 키는 **스트리밍 로케이터**와 연결됩니다. Media Services는 권한 있는 사용자에게 암호화 키를 제공하는 키 배달 서비스도 제공합니다.

플레이어가 스트림을 요청하는 경우 Media Services는 지정된 키를 사용하여 콘텐츠를 동적으로 암호화합니다(이 경우 AES 암호화를 사용하여). 스트림의 암호를 해독하기 위해 플레이어는 키 배달 서비스에서 키를 요청합니다. 사용자에게 키를 얻을 수 있는 권한이 있는지 여부를 결정하기 위해 서비스는 키에 지정된 콘텐츠 키 정책을 평가합니다.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="create-a-streaming-locator"></a>스트리밍 로케이터 만들기

인코딩이 완료되고 콘텐츠 키 정책이 설정되면 다음 단계는 출력 자산의 비디오를 클라이언트가 재생할 수 있도록 만드는 것입니다. 다음 두 단계로 비디오를 사용할 수 있도록 합니다.

1. 스트리밍 [로케이터](https://docs.microsoft.com/rest/api/media/streaminglocators)를 만듭니다.
2. 클라이언트가 사용할 수 있는 스트리밍 URL을 빌드합니다.

**스트리밍 로케이터를** 만드는 프로세스를 게시라고 합니다. 기본적으로 스트리밍 **로케이터는** API 호출 직후에 유효합니다. 선택적 시작 및 종료 시간을 구성하지 않는 한 삭제될 때까지 지속됩니다.

[스트리밍 로케이터를](https://docs.microsoft.com/rest/api/media/streaminglocators)만들 때 원하는 **StreamingPolicyName을**지정해야 합니다. 이 자습서에서는 Azure Media Services에서 스트리밍용 콘텐츠를 게시하는 방법을 알려주는 사전 정의된 StreamingPolicy 중 하나를 사용하고 있습니다. 이 예제에서는 AES 봉투 암호화가 적용됩니다(이 암호화는 DRM 라이선스가 아닌 HTTPS를 통해 재생 클라이언트에 전달되기 때문에 ClearKey 암호화라고도 함).

> [!IMPORTANT]
> 사용자 지정 [StreamingPolicy](https://docs.microsoft.com/rest/api/media/streamingpolicies)를 사용하는 경우 Media Service 계정에 대해 이러한 정책을 제한적으로 설계하고 동일한 암호화 옵션 및 프로토콜이 필요할 때마다 Streaming Locator에 다시 사용해야 합니다. Media Service 계정에는 StreamingPolicy 항목의 수에 대한 할당량이 있습니다. 각 스트리밍 로케이터에 대해 새 스트리밍 정책을 만들면 안 됩니다.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateStreamingLocator)]

## <a name="get-a-test-token"></a>테스트 토큰 가져오기

이 자습서에서는 콘텐츠 키 정책이 토큰을 제한하도록 지정합니다. 토큰 제한 정책에는 STS(보안 토큰 서비스)에서 발급한 토큰이 수반되어야 합니다. 미디어 서비스는 [JWT](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_3) 형식의 토큰을 지원하며 샘플에서 구성하는 것입니다.

ContentKeyIdentifierClaim은 **콘텐츠 키 정책에서**사용되며, 이는 키 배달 서비스에 제공되는 토큰에 콘텐츠 키의 식별자가 있어야 함을 의미합니다. 샘플에서는 스트리밍 로케이터를 만들 때 콘텐츠 키를 지정하지 않았는데, 시스템은 우리를 위해 무작위로 생성했습니다. 테스트 토큰을 생성하려면 ContentKeyId를 사용하여 ContentKeyIdentifierClaim 클레임을 표시해야 합니다.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetToken)]

## <a name="build-a-dash-streaming-url"></a>DASH 스트리밍 URL 빌드

[이제 스트리밍 로케이터가](https://docs.microsoft.com/rest/api/media/streaminglocators) 만들어졌으니 스트리밍 URL을 얻을 수 있습니다. URL을 빌드하려면 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/streamingendpoints) 호스트 이름과 **스트리밍 로케이터** 경로를 연결해야 합니다. 이 샘플에서는 *기본* **스트리밍 엔드포인트**가 사용됩니다. Media Service 계정을 처음으로 만들면 이 *기본* **스트리밍 엔드포인트**가 중지된 상태이므로 **Start**를 호출해야 합니다.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetMPEGStreamingUrl)]

## <a name="clean-up-resources-in-your-media-services-account"></a>Media Services 계정의 리소스 정리

일반적으로 재사용하려는 개체를 제외한 모든 것을 정리해야 합니다(일반적으로 변환, 스트리밍 로케이터 등을 재사용). 실험 후 계정을 정리하려면 재사용하지 않을 리소스를 삭제해야 합니다. 예를 들어 다음 코드는 Job을 삭제합니다.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CleanUp)]

## <a name="clean-up-resources"></a>리소스 정리

이 자습서에서 만든 Media Services 및 스토리지 계정을 포함하여 리소스 그룹의 리소스가 더 이상 필요하지 않으면, 앞서 만든 리소스 그룹을 삭제합니다.

다음 CLI 명령을 실행합니다.

```azurecli
az group delete --name amsResourceGroup
```

## <a name="ask-questions-give-feedback-get-updates"></a>질문, 피드백 제공, 업데이트 받기

[Azure Media Services 커뮤니티](media-services-community.md) 문서를 체크 아웃하여 다양한 방법으로 질문을 하고, 피드백을 제공하고, Media Services에 대한 업데이트를 가져올 수 있습니다.

## <a name="additional-notes"></a>추가적인 참고 사항

* Widevine은 Google Inc.에서 제공하는 서비스로, Google Inc.의 서비스 약관 및 개인정보처리방침을 따릅니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [DRM으로 보호](protect-with-drm.md)
