---
title: ' [!DNL Asset Compute Service] 확장에 대한 이해'
description: 사용자 지정 에셋 처리를 위해  [!DNL Asset Compute Service] 기능을 확장하는 시기와 방법.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 0%

---

# 확장성 소개 {#introduction-to-extensibilty}

형식으로 변환 및 이미지 크기 조정과 같은 많은 렌디션 요구 사항은 [처리 프로필 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview)을 통해 해결됩니다. 보다 복잡한 비즈니스 요구 사항에는 조직의 요구 사항에 맞는 맞춤형 솔루션이 필요할 수 있습니다. [!DNL Experience Manager]의 처리 프로필에서 호출되는 사용자 지정 응용 프로그램을 만들어 [!DNL Asset Compute Service]을(를) 확장할 수 있습니다. 이러한 사용자 지정 응용 프로그램은 [지원되는 사용 사례](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)를 충족합니다.

>[!NOTE]
>
>[!DNL Asset Compute Service]은(는) [!DNL Cloud Service](으)로 [!DNL Experience Manager]에 대해서만 사용할 수 있습니다.

사용자 지정 응용 프로그램은 headless [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) 앱입니다. 사용자 지정 응용 프로그램으로 [!DNL Asset Compute Service]을(를) 확장하는 방법은 [SDK Asset compute](https://github.com/adobe/asset-compute-sdk) 및 Adobe Developer App Builder 개발자 도구를 통해 간단하게 수행됩니다. 이러한 도구를 사용하여 개발자는 비즈니스 논리에 집중할 수 있습니다. 사용자 지정 응용 프로그램을 만드는 것은 일반 서버리스 Adobe [!DNL I/O Runtime] 작업을 만드는 것만큼 간단합니다. 단일 Node.js JavaScript 함수입니다. [기본 사용자 지정 응용 프로그램 예제](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)가 이를 보여 줍니다.

## 사전 요구 사항 및 프로비저닝 요구 사항 {#prerequisites-and-provisioning}

다음 전제 조건을 충족하는지 확인하십시오.

* Adobe Developer App Builder 도구는 컴퓨터에 설치됩니다.
* [!DNL Experience Cloud] 조직입니다. 자세한 내용을 보려면 [App Builder 여정 시작](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials)(으)로 이동하십시오.
* 경험 조직에 [!DNL Cloud Service](으)로 [!DNL Experience Manager]이(가) 활성화되어 있어야 합니다.
* [!DNL Adobe Experience Cloud] 조직은 [!DNL Adobe Developer App Builder] developer skin peek 프로그램의 일부입니다. [액세스 신청 방법](https://developer.adobe.com/app-builder/docs/overview/getting_access)(으)로 이동합니다.
* 개발자를 위한 조직에서 개발자 역할 또는 관리자 권한을 확인합니다.
* [[!DNL aio-cli]](https://github.com/adobe/aio-cli) Adobe이 로컬에 설치되어 있는지 확인하십시오.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
