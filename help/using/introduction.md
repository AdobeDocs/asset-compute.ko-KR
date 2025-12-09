---
title: ' [!DNL Asset Compute Service] 소개'
description: '[!DNL Asset Compute Service]은(는) 복잡성을 줄이고 확장성을 개선하는 클라우드 기반 에셋 처리 서비스입니다.'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 63f83ff33ac6cd090fac4f6db18000155f464643
workflow-type: tm+mt
source-wordcount: '278'
ht-degree: 1%

---

# [!DNL Asset Compute Service] 개요 {#overview}

[!DNL Asset Compute Service]은(는) 디지털 자산을 처리할 수 있는 확장 가능한 [!DNL Adobe Experience Cloud]의 서비스입니다. 이미지, 비디오, 문서 및 기타 파일 형식을 썸네일, 추출한 텍스트 및 메타데이터, 아카이브를 포함한 다양한 변환으로 변환할 수 있습니다.

개발자는 사용자 정의 에셋 애플리케이션(사용자 정의 작업자라고도 함)을 플러그인하여 사용자 정의 사용 사례를 처리할 수 있습니다. 서비스는 Adobe [!DNL I/O Runtime]에서 작동합니다. Node.js로 작성된 [!DNL Adobe Developer App Builder]개의 Headless 앱을 통해 확장할 수 있습니다. 외부 API를 호출하여 이미지 작업을 수행하거나 [!DNL Adobe Sensei] 지원을 활용하는 등의 사용자 지정 작업을 수행할 수 있습니다.

[!DNL Adobe Developer App Builder]은(는) Adobe Experience Cloud 솔루션을 확장하기 위해 Adobe [!DNL I/O Runtime]에서 사용자 지정 웹 응용 프로그램을 빌드하고 배포하는 프레임워크입니다. 사용자 정의 응용 프로그램을 만들기 위해 개발자는 [!DNL React Spectrum]&#x200B;(Adobe의 UI 툴킷)을 활용하고 마이크로서비스를 만들고 사용자 정의 이벤트를 만들고 API를 오케스트레이션할 수 있습니다. [Adobe Developer App Builder 설명서](https://developer.adobe.com/app-builder/docs/intro_and_overview/#)를 참조하세요.

>[!NOTE]
>
>현재 [!DNL Asset Compute Service]은(는) [!DNL Experience Manager]&#x200B;(으)로 [!DNL Cloud Service]을(를) 통해서만 사용할 수 있습니다. 관리자는 [!DNL Asset Compute Service]을(를) 호출하여 처리할 자산을 전달할 수 있는 처리 프로필을 만듭니다. [자산 마이크로서비스 및 처리 프로필 사용](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)을 참조하세요.

## [!DNL Asset Compute Service]의 지원되는 사용 사례 {#possible-use-cases-benefits}

[!DNL Asset Compute Service]은(는) 기본 이미지 처리, Adobe 응용 프로그램별 전환 및 복잡한 비즈니스 요구 사항을 오케스트레이션하는 사용자 지정 응용 프로그램 만들기와 같은 몇 가지 일반적인 비즈니스 사용 사례를 지원합니다.

[!DNL Asset Compute] 웹 서비스를 사용하여 다양한 파일 형식에 대한 썸네일, [지원되는 파일 형식에 대한 고품질 이미지 렌더링](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/file-format-support)을 생성할 수 있습니다. [사용자 지정 구성을 통해 지원되는 사용 사례](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)를 참조하세요.

>[!NOTE]
>
>서비스가 자산 저장소를 제공하지 않습니다. 사용자는 이를 제공하고 클라우드 스토리지의 소스 및 렌디션 파일 위치에 대한 참조를 제공합니다.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [자산 마이크로서비스를 사용한 자산 처리 개요 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview).
>* [Adobe Developer App Builder 설명서](https://developer.adobe.com/app-builder/docs/intro_and_overview/#).
>* [처리를 위해 지원되는 파일 형식](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/file-format-support).

<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
