---
title: 아키텍처 [!DNL Asset Compute Service]
description: 방법 [!DNL Asset Compute Service] API, 애플리케이션 및 SDK가 함께 작동하여 클라우드 기반 에셋 처리 서비스를 제공합니다.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '478'
ht-degree: 0%

---

# 아키텍처 [!DNL Asset Compute Service] {#overview}

다음 [!DNL Asset Compute Service] 은 서버를 사용하지 않는 Adobe 위에 구축됩니다. [!DNL `I/O Runtime`] 플랫폼. 자산에 대한 Adobe Sensei 컨텐츠 서비스 지원을 제공합니다. 호출하는 클라이언트(전용) [!DNL Experience Manager] as a [!DNL Cloud Service] 가 지원됨)에는 에셋에 대해 검색한 Adobe Sensei 생성 정보가 제공됩니다. 반환된 정보는 JSON 형식입니다.

[!DNL Asset Compute Service] 은 을 기반으로 사용자 정의 애플리케이션을 만들어 확장 가능합니다. [!DNL Adobe Developer App Builder]. 이러한 맞춤형 애플리케이션은 다음과 같습니다 [!DNL Project Adobe Developer App Builder] headless 앱 및 사용자 지정 전환 도구를 추가하거나 외부 API를 호출하여 이미지 작업을 수행하는 등의 작업을 수행합니다.

[!DNL Project Adobe Developer App Builder] 는 Adobe에서 사용자 지정 웹 애플리케이션을 빌드하고 배포하는 프레임워크입니다 [!DNL `I/O Runtime`]. 사용자 정의 응용 프로그램을 만들기 위해 개발자는 [!DNL React Spectrum] (Adobe의 UI 툴킷), 마이크로서비스 만들기, 사용자 정의 이벤트 만들기 및 API 오케스트레이션. 다음을 참조하십시오 [Adobe Developer App Builder 설명서](https://developer.adobe.com/app-builder/docs/overview).

아키텍처의 기반이 되는 기반에는 다음이 포함됩니다.

* 주어진 작업에 필요한 것을 포함하는 응용 프로그램 전용 모듈화를 통해 응용 프로그램을 서로 분리하여 단순하게 유지할 수 있습니다.

* 서버리스 개념 [!DNL Adobe I/O] 런타임은 비동기적이고 확장성이 뛰어나고 격리된 작업 기반 처리와 같은 다양한 이점을 제공하며 이는 자산 처리에 적합합니다.

* 이진 클라우드 스토리지는 사전 서명된 URL 참조를 사용하여 스토리지에 대한 전체 액세스 권한 없이도 에셋 파일 및 렌디션을 개별적으로 저장하고 액세스하는 데 필요한 기능을 제공합니다. 전송 가속화, CDN 캐싱 및 클라우드 스토리지와 함께 컴퓨팅 애플리케이션 공동 배치를 통해 대기 시간이 짧은 최적의 콘텐츠 액세스를 제공합니다. AWS 및 Azure 클라우드 모두 지원됩니다.

![asset compute 서비스 아키텍처](assets/architecture-diagram.png)

*그림: 아키텍처 [!DNL Asset Compute Service] 및 통합 방법 [!DNL Experience Manager], 저장 및 처리 애플리케이션.*

아키텍처는 다음 부분으로 구성됩니다.

* **API 및 오케스트레이션 계층** 는 소스 에셋을 여러 표현물로 변환하도록 서비스에 지시하는 요청(JSON 형식)을 수신합니다. 이 요청은 비동기적으로 수행되며 작업 ID인 활성화 ID를 사용하여 반환됩니다. 지침은 순전히 선언적이며, 모든 표준 처리 작업(예: 썸네일 생성, 텍스트 추출)에 대해 소비자는 원하는 결과만 지정하고 특정 렌디션을 처리하는 애플리케이션은 지정하지 않습니다. 인증, 분석, 속도 제한과 같은 일반 API 기능은 서비스 앞의 Adobe API 게이트웨이를 사용하여 처리되며 [!DNL Adobe I/O] 런타임. 응용 프로그램 라우팅은 오케스트레이션 계층에 의해 동적으로 수행됩니다. 클라이언트는 고유한 매개 변수 세트가 있는 특정 변환에 대한 사용자 정의 응용 프로그램을 정의합니다. 응용 프로그램 실행은 Adobe에서 별도의 서버리스 기능이므로 완전히 병렬화할 수 있습니다 [!DNL `I/O Runtime`].

* **자산을 처리하는 애플리케이션** 특정 유형의 파일 형식 또는 타겟 표현물을 전문으로 합니다. 개념적으로 응용 프로그램은 UNIX® 파이프 개념과 같습니다. 입력 파일은 하나 이상의 출력 파일로 변환됩니다.

* **A [공통 애플리케이션 라이브러리](https://github.com/adobe/asset-compute-sdk)** 는 일반적인 작업을 처리합니다. 예를 들어 소스 파일 다운로드, 렌디션 업로드, 오류 보고, 이벤트 전송 및 모니터링이 있습니다. 이러한 설계는 로컬 파일 시스템에 국한된 상호 작용으로, 서버를 사용하지 않는 개념에 따라 애플리케이션을 간편하게 개발할 수 있도록 합니다.

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
