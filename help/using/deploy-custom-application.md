---
title: ' [!DNL Asset Compute Service] 사용자 지정 응용 프로그램 배포'
description: ' [!DNL Asset Compute Service] 사용자 지정 응용 프로그램을 배포합니다.'
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '170'
ht-degree: 0%

---

# 사용자 정의 애플리케이션 배포 {#deploy-custom-application}

응용 프로그램을 배포하려면 [aio 앱 배포](https://github.com/adobe/aio-cli#aio-appdeploy) 명령을 사용하십시오. 터미널에서 명령은 사용자 정의 애플리케이션에 액세스하기 위한 URL을 표시합니다. URL의 형식은 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`입니다.

응용 프로그램을 다시 배포하지 않고 동일한 URL을 가져오려면 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) 명령을 사용합니다.

[처리 프로필  [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/ko/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)의 URL을 사용하여 [!DNL Cloud Service](으)로 [!DNL Experience Manager]과(와) 응용 프로그램을 통합하세요.

App Builder 프로젝트 및 작업 영역이 작업을 사용하려는 [!DNL Cloud Service] 환경인 [!DNL Experience Manager]에 해당하는지 확인하십시오. 여기에는 개발, 스테이징 및 프로덕션을 위한 다양한 환경이 있습니다. Adobe Developer App Builder 애플리케이션의 루트에 있는 ENV 파일 내에 정의된 `AIO_runtime_*` 자격 증명을 확인하여 환경을 확인할 수 있습니다. 예를 들어 `Stage` 작업 영역에 배포하려면 `AIO_runtime_namespace`의 형식이 `xxxxxx_xxxxxxxxx_stage`입니다. [!DNL Experience Manager]을(를) [!DNL Cloud Service] 프로덕션 환경으로 통합하려면 Adobe Developer App Builder `Production` 작업 영역의 응용 프로그램 URL을 사용하십시오.

>[!CAUTION]
>
>중요한 [!DNL Experience Manager] 환경에서는 개인 작업 영역을 사용하지 마십시오.

>[!MORELIKETHIS]
>
>* [환경 이해 및 관리 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
