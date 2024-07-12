---
title: "[!DNL Asset Compute Service] HTTP API"
description: "[!DNL Asset Compute Service] 사용자 지정 응용 프로그램을 만들기 위한 HTTP API입니다."
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 2%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API의 사용은 개발 목적으로 제한됩니다. API는 사용자 정의 애플리케이션을 개발할 때 컨텍스트로 제공됩니다. [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]은(는) API를 사용하여 처리 정보를 사용자 지정 응용 프로그램에 전달합니다. 자세한 내용은 [자산 마이크로서비스 및 처리 프로필 사용](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)을 참조하세요.

>[!NOTE]
>
>[!DNL Asset Compute Service]은(는) [!DNL Cloud Service](으)로 [!DNL Experience Manager]에 대해서만 사용할 수 있습니다.

[!DNL Asset Compute Service] HTTP API의 모든 클라이언트는 다음 높은 수준의 흐름을 따라야 합니다.

1. 클라이언트가 IMS 조직에서 [!DNL Adobe Developer Console] 프로젝트로 프로비저닝되었습니다. 각 개별 클라이언트(시스템 또는 환경)에는 이벤트 데이터 흐름을 분리하기 위한 별도의 자체 프로젝트가 필요합니다.

1. 클라이언트는 [JWT(서비스 계정) 인증](https://developer.adobe.com/developer-console/docs/guides/)을 사용하여 기술 계정에 대한 액세스 토큰을 생성합니다.

1. 클라이언트가 [`/register`](#register)을(를) 한 번만 호출하여 저널 URL을 검색합니다.

1. 클라이언트는 렌디션을 생성할 각 에셋에 대해 [`/process`](#process-request)을(를) 호출합니다. 호출은 비동기적으로 수행됩니다.

1. 클라이언트가 [이벤트를 받기](#asynchronous-events)하도록 저널을 정기적으로 폴링합니다. 렌디션이 성공적으로 처리되거나(`rendition_created` 이벤트 유형) 오류가 있는 경우(`rendition_failed` 이벤트 유형) 요청된 각 렌디션에 대한 이벤트를 받습니다.

[adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) 모듈을 사용하면 Node.js 코드에서 API를 쉽게 사용할 수 있습니다.

## 인증 및 권한 부여 {#authentication-and-authorization}

모든 API에는 액세스 토큰 인증이 필요합니다. 요청은 다음 헤더를 설정해야 합니다.

1. 기술 계정 토큰인 전달자 토큰이 포함된 `Authorization` 헤더가 Adobe Developer Console 프로젝트에서 [JWT 교환](https://developer.adobe.com/developer-console/docs/guides/)을 통해 수신되었습니다. [범위](#scopes)는 아래에 설명되어 있습니다.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. IMS 조직 ID가 있는 `x-gw-ims-org-id` 헤더입니다.

1. [!DNL Adobe Developers Console] 프로젝트의 클라이언트 ID가 있는 `x-api-key`.

### 범위 {#scopes}

액세스 토큰에 대해 다음 범위를 확인하십시오.

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

이 범위를 사용하려면 [!DNL Adobe Developer Console] 프로젝트를 `Asset Compute`, `I/O Events` 및 `I/O Management API` 서비스에 구독해야 합니다. 개별 범위의 분류는 다음과 같습니다.

* 기본
   * 범위: `openid,AdobeID`

* Asset compute
   * metascope: `asset_compute_meta`
   * 범위: `asset_compute,read_organizations`

* Adobe [!DNL `I/O Events`]
   * metascope: `event_receiver_api`
   * 범위: `event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * metascope: `ent_adobeio_sdk`
   * 범위: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 등록 {#register}

서비스를 구독한 고유한 [!DNL Adobe Developer Console] 프로젝트인 [!DNL Asset Compute service]의 각 클라이언트는 처리 요청을 수행하기 전에 [등록](#register-request)해야 합니다. 등록 단계에서는 렌디션 처리에서 비동기 이벤트를 검색하는 데 필요한 고유한 이벤트 저널을 반환합니다.

라이프사이클이 끝나면 클라이언트는 [등록 취소](#unregister-request)할 수 있습니다.

### 요청 등록 {#register-request}

이 API 호출은 [!DNL Asset Compute] 클라이언트를 설정하고 이벤트 저널 URL을 제공합니다. 이 프로세스는 멱등 연산이며 각 클라이언트에 대해 한 번만 호출하면 됩니다. 저널 URL을 검색하기 위해 다시 호출할 수 있습니다.

| 매개변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/register` |
| 헤더 `Authorization` | 모든 [권한 부여 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 시스템 간 처리 요청에 대한 고유한 종단 간 식별자에 대해 클라이언트가 설정하는 선택 사항입니다. |
| 요청 본문 | 비어 있어야 합니다. |

### 응답 등록 {#register-response}

| 매개변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | `X-Request-Id` 요청 헤더와 동일하거나 고유하게 생성된 헤더입니다. 시스템 간 요청 식별, 지원 요청 또는 둘 다에 사용합니다. |
| 응답 본문 | `journal`, `ok` 또는 `requestId` 필드가 있는 JSON 개체입니다. |

HTTP 상태 코드는 다음과 같습니다.

* **200 Success**: 요청이 성공적으로 수행된 때입니다. `journal` URL은 `/process`을(를) 통해 시작된 비동기 처리 결과에 대한 알림을 받습니다. 완료 시 `rendition_created`개의 이벤트를 알려주거나 프로세스가 실패하면 `rendition_failed`개의 이벤트를 알려줍니다.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 권한 없음**: 요청에 유효한 [인증](#authentication-and-authorization)이 없는 경우 발생합니다. 예를 들어 잘못된 액세스 토큰 또는 잘못된 API 키가 있을 수 있습니다.

* **403 사용할 수 없음**: 요청에 유효한 [권한 부여](#authentication-and-authorization)가 없는 경우 발생합니다. 올바른 액세스 토큰이 예이지만 Adobe Developer Console 프로젝트(기술 계정)가 모든 필수 서비스를 구독하는 것은 아닙니다.

* **429 요청이 너무 많음**: 이 클라이언트가 시스템을 오버로드할 때 발생합니다. 클라이언트는 [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff)를 사용하여 다시 시도해야 합니다. 시체가 비어 있습니다.
* **4xx 오류**: 다른 클라이언트 오류가 발생하여 등록하지 못했습니다. 모든 오류에 대해 보장되지는 않지만 일반적으로 다음과 같은 JSON 응답이 반환됩니다.

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx 오류**: 다른 서버측 오류가 있고 등록하지 못한 경우 발생합니다. 모든 오류에 대해 보장되지는 않지만 일반적으로 다음과 같은 JSON 응답이 반환됩니다.

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### 요청 등록 해제 {#unregister-request}

이 API 호출은 [!DNL Asset Compute] 클라이언트의 등록을 취소합니다. 이 등록 취소가 발생하면 더 이상 `/process`을(를) 호출할 수 없습니다. 등록되지 않은 클라이언트 또는 아직 등록되지 않은 클라이언트에 대해 API 호출을 사용하면 `404` 오류가 반환됩니다.

| 매개변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/unregister` |
| 헤더 `Authorization` | 모든 [권한 부여 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 선택 사항입니다. 클라이언트는 시스템 전반에 걸쳐 처리 요청에 대한 고유한 종단 간 식별자에 대해 이를 설정할 수 있다. |
| 요청 본문 | 비어 있음. |

### 응답 등록 해제 {#unregister-response}

| 매개변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | `X-Request-Id` 요청 헤더와 동일하거나 고유하게 생성된 헤더입니다. 시스템 또는 지원 요청에서 요청을 식별하는 데 사용합니다. |
| 응답 본문 | `ok` 및 `requestId` 필드가 있는 JSON 개체입니다. |

상태 코드는 다음과 같습니다.

* **200 Success**: 등록 및 저널이 검색되고 제거될 때 발생합니다.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 권한 없음**: 요청에 유효한 [인증](#authentication-and-authorization)이 없는 경우 발생합니다. 예를 들어 잘못된 액세스 토큰 또는 잘못된 API 키가 있을 수 있습니다.

* **403 사용할 수 없음**: 요청에 유효한 [권한 부여](#authentication-and-authorization)가 없는 경우 발생합니다. 올바른 액세스 토큰이 예이지만 Adobe Developer Console 프로젝트(기술 계정)가 모든 필수 서비스를 구독하는 것은 아닙니다.

* **404 찾을 수 없음**: 제공된 자격 증명이 등록 취소되었거나 잘못된 경우 이 상태가 나타납니다.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 요청이 너무 많음**: 시스템이 오버로드될 때 발생합니다. 클라이언트는 [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff)를 사용하여 다시 시도해야 합니다. 시체가 비어 있습니다.

* **4xx 오류**: 다른 클라이언트 오류가 발생하여 등록을 취소하지 못했습니다. 모든 오류에 대해 보장되지는 않지만 일반적으로 다음과 같은 JSON 응답이 반환됩니다.

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx 오류**: 다른 서버측 오류가 있고 등록하지 못한 경우 발생합니다. 모든 오류에 대해 보장되지는 않지만 일반적으로 다음과 같은 JSON 응답이 반환됩니다.

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## 요청 처리 {#process-request}

`process` 작업은 요청의 지침에 따라 소스 에셋을 여러 표현물로 변환하는 작업을 제출합니다. 완료 성공(이벤트 유형 `rendition_created`) 또는 오류(이벤트 유형 `rendition_failed`)에 대한 알림이 이벤트 저널로 전송되며, 이벤트 저널은 `/process`개의 요청을 수행하기 전에 [`/register`](#register)을(를) 사용하여 한 번 검색해야 합니다. 형식이 잘못된 요청은 즉시 실패하고 400 오류 코드가 표시됩니다.

바이너리는 Amazon AWS S3 사전 서명된 URL 또는 Azure Blob Storage SAS URL과 같은 URL을 사용하여 참조됩니다. `source` 자산(`GET` URL)을 읽고 렌디션(`PUT` URL)을 쓰는 데 모두 사용됩니다. 클라이언트는 이러한 사전 서명된 URL을 생성할 책임이 있습니다.

| 매개변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/process` |
| MIME 유형 | `application/json` |
| 헤더 `Authorization` | 모든 [권한 부여 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 선택 사항입니다. 클라이언트는 고유한 종단 간 식별자를 설정하여 시스템 전반의 처리 요청을 추적할 수 있습니다. |
| 요청 본문 | 아래 설명된 대로 프로세스 요청 JSON 형식이어야 합니다. 처리할 에셋과 생성할 렌디션에 대한 지침을 제공합니다. |

### 프로세스 요청 JSON {#process-request-json}

`/process`의 요청 본문은 다음과 같은 높은 수준의 스키마를 가진 JSON 개체입니다.

```json
{
    "source": "",
    "renditions" : []
}
```

사용 가능한 필드는 다음과 같습니다.

| 이름 | 유형 | 설명 | 예 |
|--------------|----------|-------------|---------|
| `source` | `string` | 처리되는 소스 에셋의 URL. 필요한 표현물 형식(예: `fmt=zip`)을 기반으로 합니다. | `"http://example.com/image.jpg"` |
| `source` | `object` | 처리되는 소스 에셋을 설명합니다. 아래의 [Source 개체 필드](#source-object-fields)에 대한 설명을 참조하십시오. 요청된 렌디션 형식을 기반으로 하는 선택 사항입니다(예: `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 소스 파일에서 생성할 렌디션입니다. 각 렌디션 개체는 [렌디션 명령](#rendition-instructions)을(를) 지원합니다. 필수. | `[{ "target": "https://....", "fmt": "png" }]` |

`source`은(는) URL로 보이는 `<string>`이거나 추가 필드가 있는 `<object>`일 수 있습니다. 다음 변형은 유사합니다.

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Source 개체 필드 {#source-object-fields}

| 이름 | 유형 | 설명 | 예 |
|-----------|----------|-------------|---------|
| `url` | `string` | 처리할 소스 자산의 URL. 필수. | `"http://example.com/image.jpg"` |
| `name` | `string` | Source 자산 파일 이름입니다. MIME 유형이 검색되지 않는 경우 이름의 파일 확장자가 사용될 수 있습니다. URL 경로에 지정된 파일 이름보다 우선 순위가 높습니다. 또한 이진 리소스의 `content-disposition` 헤더에 있는 파일 이름보다 우선 순위가 높습니다. 기본값은 &quot;file&quot;입니다. | `"image.jpg"` |
| `size` | `number` | Source 에셋 파일 크기(바이트)입니다. 이진 리소스의 `content-length` 헤더보다 우선합니다. | `10234` |
| `mimetype` | `string` | Source 에셋 파일 MIME 유형. 이진 리소스의 `content-type` 헤더보다 우선합니다. | `"image/jpeg"` |

### 전체 `process` 요청 예 {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## 프로세스 응답 {#process-response}

`/process` 요청은 기본 요청 유효성 검사에 따라 성공 또는 실패로 즉시 반환됩니다. 실제 자산 처리는 비동기적으로 발생합니다.

| 매개변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | `X-Request-Id` 요청 헤더와 동일하거나 고유하게 생성된 헤더입니다. 시스템 또는 지원 요청에서 요청을 식별하는 데 사용합니다. |
| 응답 본문 | `ok` 및 `requestId` 필드가 있는 JSON 개체입니다. |

상태 코드:

* **200 성공**: 요청이 성공적으로 제출된 경우. 응답 JSON에 `"ok": true`이(가) 포함되어 있습니다.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 잘못된 요청**: 요청이 잘못 구성된 경우(예: JSON 페이로드의 필수 필드가 부족한 경우). 응답 JSON에 `"ok": false`이(가) 포함되어 있습니다.

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 권한 없음**: 요청에 유효한 [인증](#authentication-and-authorization)이 없는 경우. 예를 들어 잘못된 액세스 토큰 또는 잘못된 API 키가 있을 수 있습니다.
* **403 사용할 수 없음**: 요청에 유효한 [권한 부여](#authentication-and-authorization)가 없는 경우. 올바른 액세스 토큰이 예이지만 Adobe Developer Console 프로젝트(기술 계정)가 모든 필수 서비스를 구독하는 것은 아닙니다.
* **429 요청이 너무 많음**: 이 특정 클라이언트 또는 전체 수요로 인해 시스템이 초과될 때 발생합니다. 클라이언트는 [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff)를 사용하여 다시 시도할 수 있습니다. 시체가 비어 있습니다.
* **4xx 오류**: 다른 클라이언트 오류가 있는 경우입니다. 모든 오류에 대해 보장되지는 않지만 일반적으로 다음과 같은 JSON 응답이 반환됩니다.

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx 오류**: 다른 서버 측 오류가 있는 경우. 모든 오류에 대해 보장되지는 않지만 일반적으로 다음과 같은 JSON 응답이 반환됩니다.

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

대부분의 클라이언트는 401 또는 403과 같은 *제외* 구성 문제 또는 400과 같은 잘못된 요청에 대해 [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff)로 동일한 요청을 다시 시도할 수 있습니다. 429 응답을 통한 일반적인 속도 제한과 별도로, 일시적인 서비스 중단 또는 제한으로 인해 5xx 오류가 발생할 수 있습니다. 그런 다음 일정 시간 후 다시 시도하는 것이 좋습니다.

모든 JSON 응답(있는 경우)에는 `X-Request-Id` 헤더와 동일한 값인 `requestId`이(가) 포함됩니다. Adobe 헤더는 항상 존재하므로 헤더에서 읽는 것이 좋습니다. `requestId`은(는) 처리 요청과 관련된 모든 이벤트에서도 `requestId`(으)로 반환됩니다. 클라이언트는 이 문자열의 형식을 가정해서는 안 됩니다. 불투명 문자열 식별자입니다.

## 후 처리 옵트인 {#opt-in-to-post-processing}

[Asset compute SDK](https://github.com/adobe/asset-compute-sdk)은(는) 기본 이미지 후처리 옵션 집합을 지원합니다. 사용자 지정 작업자는 렌디션 개체의 `postProcess` 필드를 `true`(으)로 설정하여 사후 처리를 명시적으로 옵트인할 수 있습니다.

지원되는 사용 사례는 다음과 같습니다.

* Crop은 crop.w, crop.h, crop.x 및 crop.y별 제한이 정의된 사각형에 대한 렌디션입니다. 변환 개체의 `instructions.crop` 필드에 자르기 세부 정보가 지정되어 있습니다.
* 폭, 높이 또는 둘 다를 사용하여 이미지 크기를 조정합니다. `instructions.width` 및 `instructions.height`은(는) 렌디션 개체에서 이를 정의합니다. 폭 또는 높이만 사용하여 크기를 조정하려면 값을 하나만 설정합니다. 컴퓨팅 서비스는 종횡비를 유지합니다.
* JPEG 이미지에 대한 품질을 설정합니다. `instructions.quality`은(는) 렌디션 개체에 정의합니다. 품질 레벨 100은 가장 높은 품질을 나타내고 낮은 숫자는 품질 감소를 나타냅니다.
* 인터레이스 이미지를 만듭니다. `instructions.interlace`은(는) 렌디션 개체에 정의합니다.
* DPI를 설정하여 픽셀에 적용된 비율을 조정하여 데스크탑 게시용으로 렌더링된 크기를 조정합니다. `instructions.dpi`은(는) dpi 해상도를 변경하기 위해 렌디션 개체에 정의합니다. 그러나 이미지의 크기를 다른 해상도로 같은 크기로 조정하려면 `convertToDpi` 지침을 사용하십시오.
* 렌더링된 폭 또는 높이가 지정된 대상 해상도(DPI)에서 원본과 동일하게 유지되도록 이미지의 크기를 조정합니다. `instructions.convertToDpi`은(는) 렌디션 개체에 정의합니다.

## 자산에 워터마크 추가 {#add-watermark}

[Asset compute SDK](https://github.com/adobe/asset-compute-sdk)에서는 PNG, JPEG, TIFF 및 GIF 이미지 파일에 워터마크를 추가할 수 있습니다. 변환의 `watermark` 개체에 있는 변환 지침에 따라 워터마크가 추가됩니다.

워터마크는 렌디션 후처리 중에 수행됩니다. 에셋에 워터마크를 지정하려면 사용자 지정 작업자 [은(는) 렌디션 개체의 필드 `postProcess`을(를) `true`(으)로 설정하여 사후 처리](#opt-in-to-post-processing)를 옵트인합니다. 작업자가 옵트인하지 않으면 워터마크 객체가 요청의 렌디션 객체에 설정되어 있더라도 워터마크가 적용되지 않습니다.

## 렌디션 지침 {#rendition-instructions}

[`/process`](#process-request)의 `renditions` 배열에 사용할 수 있는 옵션은 다음과 같습니다.

### 공통 필드 {#common-fields}

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 변환 대상 형식은 텍스트 추출의 경우 `text`이고 XMP 메타데이터를 xml로 추출하는 경우 `xmp`일 수도 있습니다. [지원되는 형식](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support)을 참조하세요. | `png` |
| `worker` | `string` | [사용자 지정 응용 프로그램](develop-custom-application.md)의 URL. `https://` URL이어야 합니다. 이 필드가 있으면 사용자 정의 응용 프로그램에서 렌디션을 만듭니다. 그런 다음 사용자 정의 애플리케이션에서 다른 모든 렌디션 설정 필드를 사용합니다. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | HTTP PUT을 사용하여 생성된 렌디션을 업로드해야 하는 URL입니다. | `http://w.com/img.jpg` |
| `target` | `object` | 생성된 렌디션에 대한 다중 부분 사전 서명된 URL 업로드 정보입니다. 이 정보는 이 [다중 부분 업로드 동작](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)을 사용하는 [AEM/Oak 다이렉트 이진 업로드](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html)에 대한 것입니다.<br>필드:<ul><li>`urls`: 사전 서명된 각 부분 URL에 대해 하나씩 문자열 배열</li><li>`minPartSize`: 한 부분에 사용할 최소 크기 = url</li><li>`maxPartSize`: 한 부분에 사용할 최대 크기 = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 선택 사항입니다. 클라이언트가 예약된 공간을 제어하고 있는 그대로 렌디션 이벤트에 전달합니다. 클라이언트가 사용자 지정 정보를 추가하여 렌디션 이벤트를 식별할 수 있습니다. 클라이언트는 언제든지 변경할 수 있으므로 사용자 정의 애플리케이션에서 수정하거나 의존해서는 안 됩니다. | `{ ... }` |

### 렌디션별 필드 {#rendition-specific-fields}

현재 지원되는 파일 형식 목록은 [지원되는 파일 형식](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support)을 참조하십시오.

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `*` | `*` | [사용자 지정 응용 프로그램](develop-custom-application.md)에서 인식하는 고급 사용자 지정 필드를 추가할 수 있습니다. | |
| `embedBinaryLimit` | `number`바이트 | 렌디션의 파일 크기가 지정된 값보다 작은 경우 렌디션 생성이 완료된 후 전송되는 이벤트에 포함됩니다. 임베드에 허용되는 최대 크기는 32KB(32 x 1024바이트)입니다. 렌디션의 크기가 `embedBinaryLimit` 제한보다 큰 경우 클라우드 저장소의 위치에 표시되고 이벤트에 포함되지 않습니다. | `3276` |
| `width` | `number` | 너비(픽셀 단위). 이미지 표현물에만 해당됩니다. | `200` |
| `height` | `number` | 높이(픽셀 단위). 이미지 표현물에만 해당됩니다. | `200` |
|                   |          | 종횡비는 다음과 같은 경우 항상 유지됩니다. <ul> <li> `width`과(와) `height`을(를) 모두 지정한 다음 종횡비를 유지하면서 이미지가 크기에 맞게 조정됩니다. </li><li> `width` 또는 `height`만 지정된 경우 결과 이미지는 종횡비를 유지하면서 해당 차원을 사용합니다</li><li> `width` 또는 `height`을(를) 지정하지 않으면 원본 이미지 픽셀 크기가 사용됩니다. 소스 유형에 따라 다릅니다. PDF 파일과 같은 일부 형식의 경우 기본 크기가 사용됩니다. 최대 크기 제한이 있을 수 있습니다.</li></ul> | |
| `quality` | `number` | `1`에서 `100` 사이의 jpeg 품질을 지정하십시오. 이미지 표현물에만 적용됩니다. | `90` |
| `xmp` | `string` | XMP 메타데이터 원본에 쓰기(writeback)에서만 사용되며 지정된 렌디션에 다시 쓸 수 있도록 base64로 인코딩된 XMP입니다. | |
| `interlace` | `bool` | 인터레이스 PNG나 GIF 또는 프로그레시브 JPEG을 `true`(으)로 설정하여 만듭니다. 다른 파일 형식에는 영향을 주지 않습니다. | |
| `jpegSize` | `number` | JPEG 파일의 대략적인 크기(바이트)입니다. `quality` 설정을 재정의합니다. 다른 형식에는 영향을 주지 않습니다. | |
| `dpi` | `number` 또는 `object` | x 및 y DPI를 설정합니다. 단순성을 위해, 그것은 또한 x와 y 모두에 사용되는 단일 숫자로 설정될 수 있다. 이미지 자체에는 영향을 주지 않습니다. | `96` 또는 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 또는 `object` | x 및 y DPI는 실제 크기를 유지하면서 값을 다시 샘플링합니다. 단순성을 위해, 그것은 또한 x와 y 모두에 사용되는 단일 숫자로 설정될 수 있다. | `96` 또는 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | ZIP 보관 파일(`fmt=zip`)에 포함할 파일 목록입니다. 각 항목은 URL 문자열이거나 필드가 있는 객체일 수 있습니다.<ul><li>`url`: 파일을 다운로드할 URL</li><li>`path`: ZIP에서 이 경로에 파일을 저장합니다.</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP 아카이브(`fmt=zip`)에 대한 중복 처리입니다. 기본적으로 ZIP의 동일한 경로에 저장된 여러 파일은 오류를 생성합니다. `duplicate`을(를) `ignore`(으)로 설정하면 첫 번째 자산만 저장되고 나머지 자산은 무시됩니다. | `ignore` |
| `watermark` | `object` | [워터마크](#watermark-specific-fields)에 대한 지침을 포함합니다. |  |

### 워터마크별 필드 {#watermark-specific-fields}

PNG 형식은 워터마크로 사용됩니다.

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 워터마크 크기(`0.0`에서 `1.0` 사이)입니다. `1.0`은(는) 워터마크의 원래 크기(1:1)를 나타내며 값이 낮을수록 워터마크 크기가 감소함을 의미합니다. | `0.5` 값은 원래 크기의 절반을 의미합니다. |
| `image` | `url` | 워터마크에 사용할 PNG 파일의 URL입니다. | |

## 비동기 이벤트 {#asynchronous-events}

렌디션 처리가 완료되거나 오류가 발생하면 Adobe [!DNL `I/O Events Journal`](으)로 이벤트가 전송됩니다. 클라이언트는 [`/register`](#register)을(를) 통해 제공된 저널 URL을 수신해야 합니다. 저널 응답에는 각 이벤트에 대해 하나의 개체로 구성된 `event` 배열이 포함되며, 그 중 `event` 필드에는 실제 이벤트 페이로드가 포함됩니다.

[!DNL Asset Compute Service]의 모든 이벤트에 대한 Adobe [!DNL `I/O Events`] 형식은 `asset_compute`입니다. 저널은 이 이벤트 유형만 자동으로 구독되며 [!DNL Adobe Developer] 이벤트 유형을 기준으로 필터링할 추가 요구 사항은 없습니다. 서비스별 이벤트 유형은 이벤트의 `type` 속성에서 사용할 수 있습니다.

### 이벤트 유형 {#event-types}

| 이벤트 | 설명 |
|---------------------|-------------|
| `rendition_created` | 정상적으로 처리 및 업로드된 각 렌디션에 대해 전송됩니다. |
| `rendition_failed` | 처리하거나 업로드하지 못한 각 렌디션에 대해 전송됩니다. |

### 이벤트 속성 {#event-attributes}

| 속성 | 유형 | 이벤트 | 설명 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)에서 정의한 대로, 이벤트가 간소화된 확장 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 형식으로 전송된 시점의 타임스탬프. |
| `requestId` | `string` | `*` | `X-Request-Id` 헤더와 동일한 `/process`에 대한 원래 요청의 요청 ID입니다. |
| `source` | `object` | `*` | `/process` 요청의 `source`. |
| `userData` | `object` | `*` | 설정된 경우 `/process` 요청의 `userData` 렌디션입니다. |
| `rendition` | `object` | `rendition_*` | `/process`에서 전달된 해당 렌디션 개체입니다. |
| `metadata` | `object` | `rendition_created` | 렌디션의 [메타데이터](#metadata) 속성입니다. |
| `errorReason` | `string` | `rendition_failed` | 필요한 경우 렌디션 실패 [reason](#error-reasons). |
| `errorMessage` | `string` | `rendition_failed` | 렌디션 실패에 대한 자세한 내용을 알려주는 텍스트입니다. |

### 메타데이터 {#metadata}

| 속성 | 설명 |
|--------|-------------|
| `repo:size` | 렌디션의 크기입니다(바이트). |
| `repo:sha1` | 렌디션의 sha1 다이제스트입니다. |
| `dc:format` | 렌디션의 MIME 유형입니다. |
| `repo:encoding` | 텍스트 기반 형식인 경우 렌디션의 charset 인코딩입니다. |
| `tiff:ImageWidth` | 렌디션의 폭(픽셀 단위)입니다. 이미지 표현물에만 표시됩니다. |
| `tiff:ImageLength` | 렌디션 길이(픽셀 단위)입니다. 이미지 표현물에만 표시됩니다. |

### 오류 이유 {#error-reasons}

| 이유 | 설명 |
|---------|-------------|
| `RenditionFormatUnsupported` | 요청한 렌디션 형식이 지정된 소스에 대해 지원되지 않습니다. |
| `SourceUnsupported` | 유형이 지원되더라도 특정 소스는 지원되지 않습니다. |
| `SourceCorrupt` | 소스 데이터가 손상되었습니다. 여기에는 빈 파일이 포함됩니다. |
| `RenditionTooLarge` | `target`에 제공된 사전 서명된 URL을 사용하여 렌디션을 업로드할 수 없습니다. 실제 렌디션 크기는 `repo:size`에서 메타데이터로 사용할 수 있으며, 클라이언트가 올바른 수의 사전 서명된 URL로 이 렌디션을 재처리하는 데 사용됩니다. |
| `GenericError` | 기타 예기치 않은 오류 |
