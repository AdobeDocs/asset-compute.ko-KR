---
title: ' [!DNL Asset Compute Service]에 대한 개발'
description: ' [!DNL Asset Compute Service]을(를) 사용하여 사용자 지정 응용 프로그램을 만듭니다.'
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 63f83ff33ac6cd090fac4f6db18000155f464643
workflow-type: tm+mt
source-wordcount: '1489'
ht-degree: 0%

---

# 사용자 정의 애플리케이션 개발 {#develop}

사용자 정의 응용 프로그램 개발을 시작하기 전에:

* [필수 구성 요소](/help/using/understand-extensibility.md#prerequisites-and-provisioning)가 모두 충족되었는지 확인하십시오.
* [필요한 소프트웨어 도구](/help/using/setup-environment.md#create-dev-environment)를 설치하십시오.
* 사용자 지정 응용 프로그램을 만들 준비가 되었는지 확인하려면 [환경 설정](setup-environment.md)을 참조하십시오.

## 사용자 정의 애플리케이션 만들기 {#create-custom-application}

로컬에 [Adobe aio-cli](https://github.com/adobe/aio-cli)를 설치해야 합니다.

1. 사용자 지정 응용 프로그램을 만들려면 [App Builder 프로젝트를 만듭니다](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#4-bootstrapping-new-app-using-the-cli). 이렇게 하려면 터미널에서 `aio app init <app-name>`을(를) 실행하십시오.

   아직 로그인하지 않은 경우, 이 명령은 Adobe ID으로 [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis)에 로그인하도록 요구하는 브라우저 메시지를 표시합니다. CLI에서 로그인하는 방법에 대한 자세한 내용은 [여기](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#3-signing-in-from-cli)를 참조하십시오.

   Adobe은 먼저 로그인하는 것을 권장합니다. 문제가 있는 경우 [지침에 따라 ](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#42-developer-is-not-logged-in-as-enterprise-organization-user)에 로그인하지 않고 앱을 만듭니다.

1. 로그인 후 CLI의 안내에 따라 응용 프로그램에 사용할 `Organization`, `Project` 및 `Workspace`을(를) 선택하십시오. [환경을 설정](setup-environment.md)할 때 만든 프로젝트 및 작업 영역을 선택하십시오. `Which extension point(s) do you wish to implement ?` 메시지가 표시되면 `DX Asset Compute Worker`을(를) 선택해야 합니다.

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. `Which Adobe I/O App features do you want to enable for this project?` 메시지가 표시되면 `Actions`을(를) 선택합니다. 웹 자산에서 다른 인증 및 권한 부여 검사를 사용하므로 `Web Assets` 옵션을 선택 해제해야 합니다.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. `Which type of sample actions do you want to create?` 메시지가 표시되면 `Adobe Asset Compute Worker`을(를) 선택해야 합니다.

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 나머지 프롬프트에 따라 Visual Studio Code(또는 즐겨 사용하는 코드 편집기)에서 새 응용 프로그램을 엽니다. 여기에는 사용자 지정 응용 프로그램에 대한 스캐폴딩과 샘플 코드가 포함되어 있습니다.

   App Builder 앱의 [기본 구성 요소](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#5-anatomy-of-an-app-builder-application)에 대해 읽어 보십시오.

   템플릿 응용 프로그램은 Adobe의 [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk)을(를) 사용하여 응용 프로그램 표현물을 업로드, 다운로드 및 오케스트레이션하므로 개발자는 사용자 지정 응용 프로그램 논리만 구현하면 됩니다. `actions/<worker-name>` 폴더 내에서 `index.js` 파일은 사용자 지정 응용 프로그램 코드를 추가할 위치입니다.

사용자 지정 응용 프로그램에 대한 예제와 아이디어는 [사용자 지정 응용 프로그램 예제](#try-sample)를 참조하십시오.

### 자격 증명 추가 {#add-credentials}

애플리케이션을 생성할 때 로그인하면 대부분의 App Builder 자격 증명이 ENV 파일에 수집됩니다. 그러나 개발자 도구를 사용하려면 추가 자격 증명이 필요합니다.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 개발자 도구 저장소 자격 증명 {#developer-tool-credentials}

개발자가 [!DNL Asset Compute service]을(를) 사용하여 사용자 지정 앱을 평가하는 도구를 사용하려면 클라우드 저장소 컨테이너를 사용해야 합니다. 이 컨테이너는 테스트 파일을 저장하고 앱에서 생성된 렌디션의 수신 및 프레젠테이션에 필수적입니다.

>[!NOTE]
>
>이 컨테이너는 [!DNL Adobe Experience Manager]&#x200B;(으)로 [!DNL Cloud Service]의 클라우드 저장소와 별개입니다. Asset Compute 개발자 도구를 사용한 개발 및 테스트에만 적용됩니다.

[지원되는 클라우드 저장소 컨테이너](https://github.com/adobe/asset-compute-devtool#prerequisites)에 액세스할 수 있어야 합니다. 이 컨테이너는 필요할 때마다 다양한 개발자가 다른 프로젝트에 일괄적으로 사용합니다.

#### ENV 파일에 자격 증명 추가 {#add-credentials-env-file}

`.env` 파일에 개발 도구에 대한 후속 자격 증명을 삽입합니다. 파일은 App Builder 프로젝트의 루트에 있습니다.
<!--
1. Add the absolute path to the private key file created while adding services to your App Builder Project:

    ```conf
    ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
    ```

   >[!NOTE]
   >
   >JWT is deprecated and Private Key is not available for download. While we are working on updating the testing tools, note that custom workers created using OAuth can be deployed but devtools would not work.
-->
1. Adobe Developer Console에서 파일을 다운로드합니다. 프로젝트의 루트로 이동하고 오른쪽 상단 모서리에서 &quot;모두 다운로드&quot;를 클릭합니다. 파일 이름으로 `<namespace>-<workspace>.json`을(를) 사용하여 파일이 다운로드됩니다. 다음 중 하나를 수행하십시오.

   * 파일 이름을 `console.json`(으)로 변경하고 프로젝트의 루트로 이동합니다.
   * 선택적으로 Adobe Developer Console 통합 JSON 파일에 대한 절대 경로를 추가할 수 있습니다. 이 파일은 프로젝트 작업 영역에서 다운로드한 것과 동일한 [`console.json`](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#42-developer-is-not-logged-in-as-enterprise-organization-user) 파일입니다.

     ```conf
     ASSET_COMPUTE_INTEGRATION_FILE_PATH=
     ```

1. S3 또는 Azure 스토리지 자격 증명을 추가합니다. 하나의 클라우드 스토리지 솔루션에만 액세스하면 됩니다.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>`config.json` 파일에 자격 증명이 포함되어 있습니다. 프로젝트 내에서 JSON 파일을 `.gitignore` 파일에 추가하여 공유하지 못하도록 합니다. `.env` 및 `.aio` 파일에도 동일하게 적용됩니다.

## 애플리케이션 실행 {#run-custom-application}

Asset Compute 개발자 도구를 사용하여 응용 프로그램을 실행하기 전에 [자격 증명](#developer-tool-credentials)을 올바르게 구성하십시오.

응용 프로그램을 개발자 도구에서 실행하려면 `aio app run` 명령을 사용합니다. 작업을 Adobe [!DNL I/O Runtime]에 배포하고 로컬 컴퓨터에서 개발 도구를 시작합니다. 이 도구는 개발 중에 애플리케이션 요청을 테스트하는 데 사용됩니다. 다음은 렌디션 요청의 예입니다.

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>`--local` 명령과 함께 `run` 플래그를 사용하지 마십시오. [!DNL Asset Compute] 사용자 지정 응용 프로그램 및 Asset Compute 개발자 도구에서는 작동하지 않습니다. 사용자 지정 응용 프로그램은 개발자의 로컬 컴퓨터에서 실행 중인 작업에 액세스할 수 없는 [!DNL Asset Compute] 서비스에 의해 호출됩니다.

응용 프로그램을 테스트하고 디버깅하는 방법은 [여기](test-custom-application.md)를 참조하세요. 사용자 지정 응용 프로그램 개발을 마치면 [사용자 지정 응용 프로그램을 배포](deploy-custom-application.md)합니다.

## Adobe에서 제공하는 샘플 애플리케이션을 사용해 보십시오. {#try-sample}

다음은 사용자 정의 응용 프로그램의 예입니다.

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [worker-animal-pictures](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 템플릿 사용자 정의 애플리케이션 {#template-custom-application}

[worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)은(는) 템플릿 응용 프로그램입니다. 소스 파일을 복사하기만 하면 렌디션이 생성됩니다. 이 응용 프로그램의 콘텐츠는 aio 앱 만들기에서 `Adobe Asset Compute`을(를) 선택할 때 받은 템플릿입니다.

응용 프로그램 파일 [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)은(는) [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview)을(를) 사용하여 소스 파일을 다운로드하고, 각 렌디션 처리를 조정하고, 결과 렌디션을 클라우드 저장소에 다시 업로드합니다.

응용 프로그램 코드 내에 정의된 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required)은(는) 모든 응용 프로그램 처리 논리를 수행할 위치입니다. `worker-basic`의 렌디션 콜백은 단순히 소스 파일 내용을 렌디션 파일에 복사합니다.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 외부 API 호출 {#call-external-api}

애플리케이션 코드에서는 애플리케이션 처리에 도움이 되도록 외부 API를 호출할 수 있습니다. 다음은 외부 API를 호출하는 애플리케이션 파일의 예입니다.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

예를 들어 [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46)은(는) [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 라이브러리를 사용하여 Wikimedia에서 정적 URL로 가져오기를 요청합니다.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 사용자 지정 매개 변수 전달 {#pass-custom-parameters}

렌디션 개체를 통해 사용자 정의 매개 변수를 전달할 수 있습니다. [`rendition` 지침](https://github.com/adobe/asset-compute-sdk#rendition)에서 응용 프로그램 내에서 참조할 수 있습니다. 렌디션 객체의 예는 다음과 같습니다.

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

사용자 지정 매개 변수에 액세스하는 응용 프로그램 파일의 예는 다음과 같습니다.

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

`example-worker-animal-pictures`이(가) 사용자 지정 매개 변수 [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39)을(를) 전달하여 Wikimedia에서 가져올 파일을 결정합니다.

## 인증 및 권한 부여 지원 {#authentication-authorization-support}

기본적으로 Asset Compute 사용자 지정 응용 프로그램에는 App Builder 프로젝트에 대한 권한 부여 및 인증 검사가 제공됩니다. `require-adobe-auth`에서 `true` 주석을 `manifest.yml`(으)로 설정하여 사용할 수 있습니다.

### 다른 Adobe API 액세스 {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

설정에서 만든 [!DNL Asset Compute] 콘솔 작업 영역에 API 서비스를 추가합니다. 이러한 서비스는 [!DNL Asset Compute Service]에서 생성한 JWT 액세스 토큰의 일부입니다. 응용 프로그램 작업 `params` 개체 내에서 토큰 및 기타 자격 증명에 액세스할 수 있습니다.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 서드파티 시스템에 대한 자격 증명 전달 {#pass-credentials-for-tp}

다른 외부 서비스에 대한 자격 증명을 처리하려면 해당 자격 증명을 작업에 대한 기본 매개 변수로 전달합니다. 전송 중 자동으로 암호화됩니다. 자세한 내용은 [Adobe I/O Runtime 개발자 안내서에서 작업 만들기](https://developer.adobe.com/app-builder/docs/guides/runtime_guides/creating-actions#)를 참조하십시오. 그런 다음 배포 중에 환경 변수를 사용하여 설정하십시오. 이러한 매개 변수는 작업 내의 `params` 개체에서 액세스할 수 있습니다.

`inputs`의 `manifest.yml` 내에서 기본 매개 변수를 설정합니다.

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

`$VAR` 식이 이름이 `VAR`인 환경 변수에서 값을 읽습니다.

개발하는 동안 로컬 `.env` 파일에서 값을 할당할 수 있습니다. 그 이유는 `aio`이(가) `.env` 파일에서 환경 변수를 초기화 셸에서 설정한 변수와 함께 자동으로 가져오기 때문입니다. 이 예제에서 `.env` 파일은 다음과 같습니다.

```CONF
#...
SECRET_KEY=secret-value
```

프로덕션 배포의 경우 CI 시스템에서 환경 변수를 설정할 수 있습니다(예: GitHub 작업에서 암호 사용). 마지막으로, 다음과 같이 애플리케이션 내의 기본 매개 변수에 액세스합니다.

```javascript
const key = params.secretKey;
```

## 애플리케이션 크기 조정 {#sizing-workers}

응용 프로그램은 Adobe [!DNL I/O Runtime]의 컨테이너에서 [을(를) 통해 구성할 수 있는 ](https://developer.adobe.com/app-builder/docs/guides/runtime_guides/system-settings#)limits`manifest.yml`을(를) 사용하여 실행됩니다.

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Asset Compute 애플리케이션에서 수행하는 광범위한 처리로 인해 최적의 성능(바이너리 자산을 처리할 수 있을 만큼 큼)과 효율성(사용하지 않는 컨테이너 메모리로 인해 리소스 낭비 방지)을 위해 이러한 제한을 조정해야 합니다.

런타임의 작업에 대한 기본 시간 제한은 1분이지만 `timeout` 제한(밀리초)을 설정하여 늘릴 수 있습니다. 더 큰 파일을 처리해야 하는 경우에는 이 시간을 늘립니다. 소스를 다운로드하고 파일을 처리하며 렌디션을 업로드하는 데 소요되는 총 시간을 고려합니다. 작업이 시간 초과된 경우, 즉 지정된 시간 제한 전에 활성화를 반환하지 않으면 런타임에서 컨테이너를 폐기하고 재사용하지 않습니다.

기본적으로 Asset Compute 애플리케이션은 네트워크 및 디스크 입력 또는 출력에 바인딩되는 경향이 있습니다. 소스 파일을 먼저 다운로드해야 합니다. 처리는 리소스를 많이 사용하므로 결과 렌디션이 다시 업로드되는 경우가 많습니다.

`memorySize` 매개 변수를 사용하여 작업 컨테이너에 할당된 메모리를 MB로 지정할 수 있습니다. 현재 이 매개 변수는 CPU에서 컨테이너에 액세스하는 양을 정의하며, 가장 중요한 것은 런타임 사용 비용의 주요 요소입니다(컨테이너가 클수록 더 많은 비용). 처리에서 더 많은 메모리나 CPU이 필요한 경우 여기에 더 큰 값을 사용하십시오. 컨테이너가 클수록 전체 처리량이 감소하므로 리소스를 낭비하지 마십시오.

또한 `concurrency` 설정을 사용하여 컨테이너 내에서 작업 동시성을 제어할 수 있습니다. 이 설정은 단일 컨테이너(동일한 작업)가 받는 동시 활성화 수입니다. 이 모델에서 작업 컨테이너는 해당 제한까지 여러 동시 요청을 수신하는 Node.js 서버와 같습니다. 런타임의 기본값 `memorySize`이(가) 200MB로 설정되어 더 작은 App Builder 작업에 적합합니다. Asset Compute 애플리케이션의 경우, 로컬 처리 및 디스크 사용량이 많기 때문에 이 기본값이 과도할 수 있습니다. 일부 응용 프로그램은 구현에 따라 동시 활동에서 잘 작동하지 않을 수도 있습니다. Asset Compute SDK을 사용하면 파일을 다른 고유 폴더에 작성하여 활성화를 구분할 수 있습니다.

응용 프로그램을 테스트하여 `concurrency` 및 `memorySize`에 대한 최적의 숫자를 찾습니다. 컨테이너가 크면 = 메모리 제한이 클수록 더 많은 동시성을 허용할 수 있지만 더 낮은 트래픽에 낭비될 수도 있습니다.
