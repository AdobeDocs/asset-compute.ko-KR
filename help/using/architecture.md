---
title: ' [!DNL Asset Compute Service]의 아키텍처'
description: ' [!DNL Asset Compute Service] API, 응용 프로그램 및 SDK을 함께 사용하여 클라우드 기반 에셋 처리 서비스를 제공하는 방법입니다.'
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: f199cecfe4409e2370b30783f984062196dd807d
workflow-type: tm+mt
source-wordcount: '478'
ht-degree: 0%

---

# [!DNL Asset Compute Service]의 아키텍처 {#overview}

[!DNL Asset Compute Service]은(는) 서버리스 Adobe [!DNL `I/O Runtime`] 플랫폼 위에 만들어집니다. 자산에 대한 Adobe Sensei 컨텐츠 서비스 지원을 제공합니다. 호출하는 클라이언트([!DNL Experience Manager]&#x200B;(으)로서 [!DNL Cloud Service]만 지원됨)에는 자산에 대해 검색한 Adobe Sensei 생성 정보가 제공됩니다. 반환된 정보는 JSON 형식입니다.

[!DNL Asset Compute Service]을(를) 기반으로 사용자 지정 응용 프로그램을 만들어 [!DNL Adobe Developer App Builder]을(를) 확장할 수 있습니다. 이러한 사용자 지정 응용 프로그램은 [!DNL Project Adobe Developer App Builder] Headless 앱이며 사용자 지정 전환 도구를 추가하거나 외부 API를 호출하여 이미지 작업을 수행하는 등의 작업을 수행합니다.

[!DNL Project Adobe Developer App Builder]은(는) Adobe [!DNL `I/O Runtime`]에서 사용자 지정 웹 응용 프로그램을 빌드하고 배포하는 프레임워크입니다. 사용자 정의 응용 프로그램을 만들기 위해 개발자는 [!DNL React Spectrum]&#x200B;(Adobe의 UI 툴킷)을 활용하고 마이크로서비스를 만들고 사용자 정의 이벤트를 만들고 API를 오케스트레이션할 수 있습니다. [Adobe Developer App Builder 설명서](https://developer.adobe.com/app-builder/docs/intro_and_overview/#)를 참조하세요.

아키텍처의 기반이 되는 기반에는 다음이 포함됩니다.

* 주어진 작업에 필요한 것을 포함하는 응용 프로그램 전용 모듈화를 통해 응용 프로그램을 서로 분리하여 단순하게 유지할 수 있습니다.

* [!DNL Adobe I/O] 런타임의 서버를 사용하지 않는 개념으로 인해 비동기, 확장성이 뛰어난, 격리된, 작업 기반 처리 등 여러 가지 이점을 얻을 수 있으므로 자산 처리에 적합합니다.

* 이진 클라우드 스토리지는 사전 서명된 URL 참조를 사용하여 스토리지에 대한 전체 액세스 권한 없이도 에셋 파일 및 렌디션을 개별적으로 저장하고 액세스하는 데 필요한 기능을 제공합니다. 전송 가속화, CDN 캐싱 및 클라우드 스토리지와 함께 컴퓨팅 애플리케이션 공동 배치를 통해 대기 시간이 짧은 최적의 콘텐츠 액세스를 제공합니다. AWS 및 Azure 클라우드 모두 지원됩니다.

![Asset Compute 서비스의 아키텍처](assets/architecture-diagram.png)

*그림: [!DNL Asset Compute Service]의 아키텍처 및 [!DNL Experience Manager], 저장소 및 처리 응용 프로그램과 통합하는 방법*

아키텍처는 다음 부분으로 구성됩니다.

* **API 및 오케스트레이션 계층**&#x200B;은(는) 서비스에서 소스 에셋을 여러 표현물로 변환하도록 지시하는 요청(JSON 형식)을 받습니다. 이 요청은 비동기적으로 수행되며 작업 ID인 활성화 ID를 사용하여 반환됩니다. 지침은 순전히 선언적이며, 모든 표준 처리 작업(예: 썸네일 생성, 텍스트 추출)에 대해 소비자는 원하는 결과만 지정하고 특정 렌디션을 처리하는 애플리케이션은 지정하지 않습니다. 인증, 분석, 속도 제한과 같은 일반 API 기능은 서비스 앞의 Adobe API Gateway를 사용하여 처리되며 [!DNL Adobe I/O] 런타임으로 이동하는 모든 요청을 관리합니다. 응용 프로그램 라우팅은 오케스트레이션 계층에 의해 동적으로 수행됩니다. 클라이언트는 고유한 매개 변수 세트가 있는 특정 변환에 대한 사용자 정의 응용 프로그램을 정의합니다. 응용 프로그램 실행은 Adobe [!DNL `I/O Runtime`]에서 별도의 서버리스 함수이므로 완전히 병렬화할 수 있습니다.

* 특정 유형의 파일 형식이나 대상 변환을 전문으로 하는 **자산을 처리하는 응용 프로그램**. 개념적으로 응용 프로그램은 UNIX® 파이프 개념과 같습니다. 입력 파일은 하나 이상의 출력 파일로 변환됩니다.

* **일반 응용 프로그램 라이브러리 [2}](https://github.com/adobe/asset-compute-sdk)에서 일반 작업을 처리합니다.** 예를 들어 소스 파일 다운로드, 렌디션 업로드, 오류 보고, 이벤트 전송 및 모니터링이 있습니다. 이러한 설계는 로컬 파일 시스템에 국한된 상호 작용으로, 서버를 사용하지 않는 개념에 따라 애플리케이션을 간편하게 개발할 수 있도록 합니다.

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
