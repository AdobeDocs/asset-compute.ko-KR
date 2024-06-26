---
title: "[!DNL Adobe Asset Compute Service] 사용 안내서"
description: 이 설명서는 다음을 포함합니다 [!DNL Asset Compute Service] 소개, 사용자 지정 코드 개발, 관리, 배포 및 문제 해결 방법 등의 작업을 제공합니다.
exl-id: 5acf87d1-a391-4802-bfce-e367fc8564df
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '206'
ht-degree: 0%

---

# [!DNL Asset Compute Service] 정보

[!DNL Asset Compute Service] 는 디지털 자산을 처리할 수 있는 확장 가능한 Adobe Experience Cloud의 서비스입니다. 이미지, 비디오, 문서 및 기타 파일 형식을 썸네일, 추출한 텍스트 및 메타데이터, 아카이브 등을 비롯한 다양한 변환으로 변환할 수 있습니다. 개발자는 을 사용하여 빌드된 사용자 지정 사용 사례를 해결하기 위해 사용자 지정 애플리케이션(사용자 지정 작업자라고도 함)을 플러그인할 수 있습니다. [Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview) 서버를 사용하지 않는 Adobe에서 실행 [[!DNL I/O Runtime]](https://developer.adobe.com/runtime/).

이 설명서는 다음을 포함합니다 [!DNL Asset Compute Service] 사용자 지정 코드의 개발, 관리, 배포 및 문제 해결 방법과 같은 주제입니다. To know what [!DNL Asset Compute Service] 이(가) 이 (으)로 이동 [소개](introduction.md). 참조: [서비스가 제공하는 이점](introduction.md#possible-use-cases-benefits).

[!DNL Asset Compute Service] 는 다양한 파일 형식 변환을 지원하고 많은 Adobe 서비스와 통합됩니다. 목록 보기 [지원되는 파일 형식 및 통합 서비스](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

다음에 대한 개요 보기 [에서 사용할 수 있는 에셋 마이크로서비스 기능 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview) 및 의 마이크로 서비스 사용 방법 [!DNL Experience Manager].

[!DNL Asset Compute Service] 확장성은 의 개방형 개발 모델에 따라 개발됨 [github.com/adobe](https://github.com/adobe) 확장 개발자의 기여를 환영합니다. 사용자 정의 애플리케이션 개발, 생성, 테스트 및 배포와 관련된 모든 구성 요소는 오픈 소스입니다. 다음을 참조하십시오 [컴퓨팅 서비스에 기여하는 방법 및 위치](contribute-to-compute-service.md).

<!--
Possible to record the below info here in this landing page to centralize the miscellaneous info about Asset Compute Service?
 List of dependencies and requirements SDK, CLI, Devtools, etc.? Or may be a link to the prerequisites.
 Introduction video when Tech Marketing team shares one.
-->
