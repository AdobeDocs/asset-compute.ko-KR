---
title: 확장에 대한 이해 [!DNL Asset Compute Service]
description: 확장 시기 및 방법 [!DNL Asset Compute Service] 사용자 지정 에셋 처리를 수행하는 기능입니다.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 0%

---

# 확장성 소개 {#introduction-to-extensibilty}

형식으로 변환 및 이미지 크기 조정과 같은 많은 렌디션 요구 사항은 다음에서 처리됩니다. [에서 처리 프로필 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview). 보다 복잡한 비즈니스 요구 사항에는 조직의 요구 사항에 맞는 맞춤형 솔루션이 필요할 수 있습니다. [!DNL Asset Compute Service] 의 처리 프로필에서 호출되는 사용자 지정 응용 프로그램을 만들어 확장할 수 있습니다. [!DNL Experience Manager]. 이러한 사용자 정의 응용 프로그램은 [지원되는 사용 사례](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] 은(는) 에서만 사용할 수 있습니다. [!DNL Experience Manager] as a [!DNL Cloud Service].

맞춤형 애플리케이션은 Headless [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) 앱. 확장 [!DNL Asset Compute Service] 사용자 정의 응용 프로그램을 사용하면 [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) 및 Adobe Developer App Builder 개발자 도구. 이러한 도구를 사용하여 개발자는 비즈니스 논리에 집중할 수 있습니다. 사용자 정의 응용 프로그램을 만드는 것은 일반 서버리스 Adobe을 만드는 것만큼 간단합니다 [!DNL I/O Runtime] 작업. 단일 Node.js JavaScript 함수입니다. 다음 [기본 사용자 정의 애플리케이션 예](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 설명하십시오.

## 사전 요구 사항 및 프로비저닝 요구 사항 {#prerequisites-and-provisioning}

다음 전제 조건을 충족하는지 확인하십시오.

* Adobe Developer App Builder 도구는 컴퓨터에 설치됩니다.
* An [!DNL Experience Cloud] 조직. 자세한 내용을 보려면 [App Builder 여정 시작](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* 경험 조직에는 다음이 있어야 합니다. [!DNL Experience Manager] as a [!DNL Cloud Service] 활성화되었습니다.
* [!DNL Adobe Experience Cloud] 조직은 [!DNL Adobe Developer App Builder] 개발자 몰래 엿보기 프로그램. 다음으로 이동 [액세스 신청 방법](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* 개발자를 위한 조직에서 개발자 역할 또는 관리자 권한을 확인합니다.
* Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) 로컬에 설치됩니다.

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
