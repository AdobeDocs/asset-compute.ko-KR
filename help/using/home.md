---
title: '[!DNL Adobe Asset Compute Service] 사용 안내서'
description: 이 설명서에서는 사용자 지정 코드의 소개, 개발, 관리, 배포 및 문제 해결 방법과 같은  [!DNL Asset Compute Service] 가지 작업을 다룹니다.
exl-id: 5acf87d1-a391-4802-bfce-e367fc8564df
source-git-commit: 63f83ff33ac6cd090fac4f6db18000155f464643
workflow-type: tm+mt
source-wordcount: '206'
ht-degree: 0%

---

# [!DNL Asset Compute Service] 정보

[!DNL Asset Compute Service]은(는) 디지털 자산을 처리할 수 있는 확장 가능한 Adobe Experience Cloud 서비스입니다. 이미지, 비디오, 문서 및 기타 파일 형식을 썸네일, 추출한 텍스트 및 메타데이터, 아카이브 등을 비롯한 다양한 변환으로 변환할 수 있습니다. 개발자는 사용자 지정 응용 프로그램(사용자 지정 작업자라고도 함)을 플러그인하여 [Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/intro_and_overview/#)을 사용하여 빌드되고 서버리스 Adobe [[!DNL I/O Runtime]](https://developer.adobe.com/runtime/)에서 실행되는 사용자 지정 사용 사례를 처리할 수 있습니다.

이 설명서에서는 사용자 지정 코드의 개발, 관리, 배포 및 문제 해결 방법과 같은 [!DNL Asset Compute Service] 주제를 다룹니다. [!DNL Asset Compute Service]이(가) 무엇인지 알아보려면 이 [소개](introduction.md)(으)로 이동하십시오. [서비스를 통해 수행할 수 있는 작업](introduction.md#possible-use-cases-benefits)도 참조하세요.

[!DNL Asset Compute Service]은(는) 다양한 파일 형식 변환을 지원하며 많은 Adobe 서비스와 통합됩니다. [지원되는 파일 형식 및 통합 서비스 목록](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/file-format-support)을 참조하세요.

[as a [!DNL Adobe Experience Manager]  [!DNL Cloud Service]에서 사용할 수 있는 &#x200B;](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview)자산 마이크로서비스 기능에 대한 개요와 [!DNL Experience Manager]에서 마이크로서비스를 사용하는 방법을 참조하십시오.

[!DNL Asset Compute Service] 확장성은 확장 개발자의 기여를 환영하는 [github.com/adobe](https://github.com/adobe)의 개방형 개발 모델로 개발되었습니다. 사용자 정의 애플리케이션 개발, 생성, 테스트 및 배포와 관련된 모든 구성 요소는 오픈 소스입니다. [Compute Service에 기여하는 방법 및 위치](contribute-to-compute-service.md)를 참조하세요.

<!--
Possible to record the below info here in this landing page to centralize the miscellaneous info about Asset Compute Service?
 List of dependencies and requirements SDK, CLI, Devtools, etc.? Or may be a link to the prerequisites.
 Introduction video when Tech Marketing team shares one.
-->
