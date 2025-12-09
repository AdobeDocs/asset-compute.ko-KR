---
title: ' [!DNL Asset Compute Service]에 필요한 개발 환경 설정'
description: 사용자 지정 코드를 만들고 테스트하려면  [!DNL Asset Compute Service] 의 개발자 환경을 설정하십시오.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 63f83ff33ac6cd090fac4f6db18000155f464643
workflow-type: tm+mt
source-wordcount: '360'
ht-degree: 1%

---

# 개발자 환경 설정 {#create-dev-environment}

[!DNL Asset Compute Service]에 대해 개발할 수 있는 설정을 만들려면 다음 요구 사항과 지침을 따르십시오.

1. [에 대한 &#x200B;](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#acquire-access-and-credentials)액세스 및 자격 증명을 획득[!DNL Adobe Developer App Builder]합니다.

1. [로컬 환경 설정](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#local-environment-set-up) 및 필요한 도구를 설정합니다.

1. 원활한 개발을 시작하는 데 도움이 되는 몇 가지 추가 도구는 다음과 같습니다.

   * [Git](https://git-scm.com/)
   * [Docker 데스크톱](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org)&#x200B;(v14 LTS, 홀수 버전은 권장되지 않음) 및 [NPM](https://www.npmjs.com). OS X HomeBrew의 사용자는 `brew install node`하여 두 가지를 모두 설치할 수 있습니다. 그렇지 않으면 [NodeJS 다운로드 페이지](https://nodejs.org/en/)에서 다운로드하십시오.
   * NodeJS에 적합한 IDE입니다. Adobe에서는 디버거에 지원되는 IDE이므로 [Visual Studio 코드(VS 코드)](https://code.visualstudio.com)을(를) 권장합니다. 다른 IDE를 코드 편집기로 사용할 수 있지만 고급 사용(예: 디버거)은 아직 지원되지 않습니다
   * 최신 Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli)&#x200B;(`aio`) 설치
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. [필수 구성 요소](/help/using/understand-extensibility.md#prerequisites-and-provisioning)를 충족하는지 확인하십시오.

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## App Builder 프로젝트 설정 {#create-App-Builder-project}

1. [!DNL Experience Cloud] 조직에 시스템 관리자 또는 개발자 역할이 있는지 확인하십시오. [Admin Console](https://adminconsole.adobe.com/overview)의 시스템 관리자가 이 역할을 설정합니다.

1. [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis)에 로그온합니다. [!DNL Experience Cloud] 통합으로 [!DNL Experience Manager]과(와) 동일한 [!DNL Cloud Service] 조직에 속해 있는지 확인하십시오. Adobe Developer Console에 대한 자세한 내용을 보려면 [콘솔 설명서](https://developer.adobe.com/developer-console/docs/guides/)로 이동하십시오.

1. [App Builder 프로젝트 만들기](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#). **[!UICONTROL 새 프로젝트 만들기]** > **[!UICONTROL 템플릿에서 프로젝트]**&#x200B;를 클릭합니다. App Builder을 선택합니다. `Production` 및 `Stage` 작업 영역의 새 App Builder 프로젝트를 만듭니다. 필요에 따라 `Development`과(와) 같은 추가 작업 공간을 추가합니다.

1. App Builder 프로젝트에서 작업 영역을 선택하고 Asset Compute에 필요한 서비스를 구독합니다. **프로젝트에 추가** > **API**&#x200B;를 클릭하고 `Asset Compute`, `IO Events` 및 `IO Events Management` 서비스를 추가하십시오. 첫 번째 API를 추가할 때 개인 키를 만들라는 메시지가 표시됩니다. 개발자 도구를 사용하여 사용자 정의 애플리케이션을 테스트하려면 이 키가 필요하므로 시스템에 이 정보를 저장하십시오.

   >[!NOTE]
   >
   >JWT는 더 이상 사용되지 않으며 개인 키를 다운로드할 수 없습니다. Adobe에서 테스트 도구를 업데이트하는 동안 OAuth를 사용하여 만든 사용자 지정 작업자를 배포할 수 있지만 devtools가 작동하지 않습니다.

## 다음 단계 {#next-step}

환경이 설정되면 [사용자 지정 응용 프로그램을 만들](develop-custom-application.md)준비가 되었습니다.

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
