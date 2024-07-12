---
title: 사용자 정의 애플리케이션 작업 이해
description: 작동 방식을 이해하는 데 도움이 되도록  [!DNL Asset Compute Service] 사용자 지정 응용 프로그램의 내부 작업.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '691'
ht-degree: 0%

---

# 사용자 정의 애플리케이션 내부 {#how-custom-application-works}

다음 일러스트레이션을 사용하여 클라이언트가 사용자 지정 애플리케이션을 사용하여 디지털 에셋을 처리할 때의 통합 워크플로를 이해합니다.

![사용자 지정 응용 프로그램 워크플로](assets/customworker.svg)

*그림: Adobe [!DNL Asset Compute Service]을(를) 사용하여 에셋을 처리할 때 관련된 단계*

## 등록 {#registration}

클라이언트가 [`/process`](api.md#process-request)에 대한 첫 번째 요청 전에 [`/register`](api.md#register)을(를) 한 번 호출해야 Adobe Asset compute에 대한 Adobe [!DNL I/O Events] 이벤트를 수신하기 위한 저널 URL을 설정하고 검색할 수 있습니다.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

[`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript 라이브러리는 NodeJS 응용 프로그램에서 등록, 처리부터 비동기 이벤트 처리에 이르기까지 필요한 모든 단계를 처리하는 데 사용할 수 있습니다. 필요한 헤더에 대한 자세한 내용은 [인증 및 권한 부여](api.md)를 참조하십시오.

## 처리 중 {#processing}

클라이언트가 [처리](api.md#process-request) 요청을 보냅니다.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

클라이언트는 사전 서명된 URL을 사용하여 렌디션의 서식을 올바르게 지정합니다. [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript 라이브러리는 NodeJS 응용 프로그램에서 URL을 사전 서명하는 데 사용할 수 있습니다. 현재 라이브러리는 Azure Blob Storage 및 AWS S3 컨테이너만 지원합니다.

처리 요청이 [!DNL Adobe I/O] 이벤트를 폴링하는 데 사용할 수 있는 `requestId`을(를) 반환합니다.

샘플 사용자 정의 애플리케이션 처리 요청은 다음과 같습니다.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

[!DNL Asset Compute Service]이(가) 사용자 지정 응용 프로그램에 사용자 지정 응용 프로그램 렌디션 요청을 보냅니다. 제공된 애플리케이션 URL(App Builder의 보안 웹 작업 URL)에 대한 HTTP POST을 사용합니다. 모든 요청은 HTTPS 프로토콜을 사용하여 데이터 보안을 극대화합니다.

사용자 지정 응용 프로그램에서 사용하는 [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)이(가) HTTP POST 요청을 처리합니다. 또한 소스 다운로드, 렌디션 업로드, Adobe [!DNL I/O Events] 전송 및 오류 처리도 처리합니다.

<!-- TBD: Add the application diagram. -->

### 애플리케이션 코드 {#application-code}

사용자 지정 코드는 로컬에서 사용할 수 있는 원본 파일(`source.path`)을 사용하는 콜백만 제공하면 됩니다. `rendition.path`은(는) 자산 처리 요청의 최종 결과를 배치할 위치입니다. 사용자 지정 응용 프로그램은 콜백을 사용하여 로컬에서 사용 가능한 소스 파일을 전달된 이름을 사용하여 렌디션 파일로 변환합니다(`rendition.path`). 사용자 지정 응용 프로그램에서 렌디션을 만들려면 `rendition.path`에 작성해야 합니다.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### 소스 파일 다운로드 {#download-source}

사용자 정의 응용 프로그램은 로컬 파일만 처리합니다. [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)에서 원본 파일 다운로드를 처리합니다.

### 렌디션 만들기 {#rendition-creation}

SDK는 각 변환에 대해 비동기 [변환 콜백 함수](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required)를 호출합니다.

콜백 함수에서 [소스](https://github.com/adobe/asset-compute-sdk#source) 및 [렌디션](https://github.com/adobe/asset-compute-sdk#rendition) 개체에 액세스할 수 있습니다. `source.path`이(가) 이미 있으며 원본 파일의 로컬 복사본 경로입니다. `rendition.path`은(는) 처리된 렌디션을 저장해야 하는 경로입니다. [disableSourceDownload 플래그](https://github.com/adobe/asset-compute-sdk#worker-options-optional)가 설정되지 않은 경우 응용 프로그램에서 정확히 `rendition.path`을(를) 사용해야 합니다. 그렇지 않으면 SDK에서 렌디션 파일을 찾거나 식별할 수 없으며, 실패합니다.

예제의 지나친 단순화는 사용자 정의 애플리케이션의 구조를 설명하고 집중하기 위해 수행됩니다. 소스 파일을 렌디션 대상에 복사하기만 하면 됩니다.

렌디션 콜백 매개 변수에 대한 자세한 내용은 [SDK API Asset compute](https://github.com/adobe/asset-compute-sdk#api-details)를 참조하십시오.

### 렌디션 업로드 {#upload-rendition}

각 렌디션을 만들어 `rendition.path`에서 제공한 경로로 파일에 저장하면 [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)에서 각 렌디션을 클라우드 저장소(AWS 또는 Azure)에 업로드합니다. 들어오는 요청에 동일한 애플리케이션 URL을 가리키는 여러 변환이 있는 경우에만 사용자 정의 애플리케이션이 동시에 여러 변환을 가져옵니다. 클라우드 스토리지로의 업로드는 각 렌디션 후 다음 렌디션에 대한 콜백을 실행하기 전에 수행됩니다.

`batchWorker()`에 다른 동작이 있습니다. 모든 렌디션을 처리하고, 모든 렌디션이 처리된 후에만 업로드합니다.

## [!DNL Adobe I/O]개 이벤트 {#aio-events}

SDK에서 각 렌디션에 대해 Adobe [!DNL I/O Events]을(를) 보냅니다. 이 이벤트는 결과에 따라 `rendition_created` 또는 `rendition_failed` 형식입니다. 자세한 내용은 [비동기 이벤트 Asset compute](api.md#asynchronous-events)을 참조하세요.

## [!DNL Adobe I/O]개 이벤트 수신 {#receive-aio-events}

클라이언트가 사용 논리에 따라 Adobe [!DNL I/O Events] 저널을 폴링합니다. 초기 저널 URL은 `/register` API 응답에 제공된 URL입니다. 이벤트에 있고 `/process`에서 반환된 것과 동일한 `requestId`을(를) 사용하여 이벤트를 식별할 수 있습니다. 모든 렌디션에는 렌디션이 업로드(또는 실패)되는 즉시 전송되는 별도의 이벤트가 있습니다. 일치하는 이벤트를 받으면 클라이언트가 결과 렌디션을 표시하거나 처리할 수 있습니다.

JavaScript 라이브러리 [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage)은(는) `waitActivation()` 메서드를 사용하여 모든 이벤트를 가져오도록 저널 폴링을 간단하게 만듭니다.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

저널 이벤트를 가져오는 방법에 대한 자세한 내용은 Adobe [[!DNL I/O Events] API](https://developer.adobe.com/events/docs/guides/api/journaling_api/)을(를) 참조하십시오.

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
