---
title: ' [!DNL Asset Compute Service] 사용자 지정 응용 프로그램 테스트 및 디버그'
description: ' [!DNL Asset Compute Service] 사용자 지정 응용 프로그램을 테스트하고 디버그합니다.'
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '775'
ht-degree: 0%

---

# 사용자 지정 응용 프로그램 테스트 및 디버그 {#test-debug-custom-worker}

## 사용자 정의 응용 프로그램에 대한 단위 테스트 실행 {#test-custom-worker}

컴퓨터에 [Docker Desktop](https://www.docker.com/get-started)을 설치합니다. 사용자 지정 작업자를 테스트하려면 응용 프로그램의 루트에서 다음 명령을 실행합니다.

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

이 명령은 아래 설명된 대로 프로젝트의 Asset compute 응용 프로그램 작업에 대한 사용자 지정 단위 테스트 프레임워크를 실행합니다. `package.json` 파일의 구성을 통해 연결됩니다. Jest와 같은 JavaScript 단위 테스트를 수행할 수도 있습니다. `aio app test`이(가) 둘 다 실행됩니다.

[aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) 플러그인은 빌드/테스트 시스템에 설치할 필요가 없도록 사용자 지정 응용 프로그램 앱에 개발 종속성으로 포함되어 있습니다.

### 애플리케이션 단위 테스트 프레임워크 {#unit-test-framework}

asset compute 애플리케이션 단위 테스트 프레임워크를 사용하면 코드를 작성하지 않고도 애플리케이션을 테스트할 수 있습니다. 응용 프로그램의 소스 대 렌디션 파일 원칙에 의존합니다. 테스트 소스 파일, 선택적 매개 변수, 예상 표현물 및 사용자 정의 유효성 검사 스크립트를 사용하여 테스트 사례를 정의하기 위해 특정 파일 및 폴더 구조를 설정해야 합니다. 기본적으로 렌디션은 바이트 동일성과 비교됩니다. 또한 외부 HTTP 서비스는 간단한 JSON 파일을 사용하여 쉽게 조롱할 수 있습니다.

### 테스트 추가 {#add-tests}

프로젝트의 루트 수준에서 `test` 폴더 내에 테스트가 필요합니다. 각 응용 프로그램에 대한 테스트 사례는 경로 `test/asset-compute/<worker-name>`에 있어야 하며 각 테스트 사례에 대해 하나의 폴더가 있어야 합니다.

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

일부 예제는 [사용자 지정 응용 프로그램 예제](https://github.com/adobe/asset-compute-example-workers/)를 참조하십시오. 아래는 자세한 참조입니다.

### 출력 테스트 {#test-output}

Adobe Developer App Builder 앱 루트의 `build` 디렉터리에는 사용자 지정 응용 프로그램의 자세한 테스트 결과와 로그가 들어 있습니다. 이러한 세부 정보는 `aio app test` 명령의 출력에도 표시됩니다.

### 모의 외부 서비스 {#mock-external-services}

HOST_NAME을 모방하려는 특정 호스트로 사용하여 테스트 시나리오에 대한 `mock-<HOST_NAME>.json` 파일을 만들어 동작 내에서 외부 서비스 호출을 시뮬레이션할 수 있습니다. 사용 사례의 예는 S3에 대해 별도의 호출을 수행하는 애플리케이션입니다. 새 테스트 구조는 다음과 같습니다.

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

모의 파일은 JSON 형식의 http 응답입니다. 자세한 내용은 [이 설명서](https://www.mock-server.com/mock_server/creating_expectations.html)를 참조하세요. 모의할 호스트 이름이 여러 개 있는 경우 `mock-<mocked-host>.json`개의 파일을 여러 개 정의하십시오. 다음은 이름이 `mock-google.com.json`인 `google.com`의 샘플 모의 파일입니다.

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

예제 `worker-animal-pictures`은(는) 상호 작용하는 Wikimedia 서비스에 대한 [모의 파일](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json)을(를) 포함합니다.

#### 여러 테스트 사례에서 파일 공유 {#share-files-across-test-cases}

Adobe 여러 테스트에서 `file.*`, `params.json` 또는 `validate` 스크립트를 공유하는 경우 상대 심볼릭 링크를 사용하는 것이 좋습니다. Git에서 지원됩니다. 다른 이름이 있을 수 있으므로 공유 파일에 고유한 이름을 지정해야 합니다. 아래 예에서 테스트는 몇 개의 공유 파일과 자체 파일을 혼합하여 일치시키고 있습니다.

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### 테스트 예상 오류 {#test-unexpected-errors}

오류 테스트 사례에는 필요한 `rendition.*` 파일이 포함되지 않아야 하며 `params.json` 파일 내에 필요한 `errorReason`을(를) 정의해야 합니다.

>[!NOTE]
>
>테스트 사례에 필요한 `rendition.*` 파일이 포함되어 있지 않고 `params.json` 파일 내에 필요한 `errorReason`을(를) 정의하지 않으면 `errorReason`의 오류 사례로 간주됩니다.

오류 테스트 사례 구조:

```json
<error_test_case>/
    file.jpg
    params.json
```

오류 이유가 있는 매개 변수 파일:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

[Asset compute 오류 이유](https://github.com/adobe/asset-compute-commons#error-reasons)에 대한 전체 목록 및 설명을 참조하세요.

## 사용자 지정 응용 프로그램 디버깅 {#debug-custom-worker}

다음 단계에서는 Visual Studio 코드를 사용하여 사용자 지정 응용 프로그램을 디버깅하는 방법을 보여 줍니다. 이를 통해 라이브 로그, 히트 중단점 및 코드를 단계별로 볼 수 있을 뿐만 아니라 활성화 시마다 로컬 코드 변경 사항을 실시간으로 다시 로드할 수 있습니다.

`aio`은(는) 이러한 단계를 대부분 자동화합니다. [Adobe Developer App Builder 설명서](https://developer.adobe.com/app-builder/docs/getting_started/first_app)의 응용 프로그램 디버깅 섹션으로 이동합니다. 현재 아래 단계에는 해결 방법이 포함되어 있습니다.

1. GitHub에서 최신 [wskdebug](https://github.com/apache/openwhisk-wskdebug) 및 선택적 [ngrok](https://www.npmjs.com/package/ngrok)을(를) 설치합니다.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. JSON 파일에서 사용자 설정에 추가합니다. 이전 Visual Studio Code Debugger를 계속 사용합니다. 새 WSKDEBUG `"debug.javascript.usePreview": false`에 [일부 문제](https://github.com/apache/openwhisk-wskdebug/issues/74)가 있습니다.
1. `aio app run`을(를) 통해 열려 있는 앱의 인스턴스를 모두 닫습니다.
1. `aio app deploy`을(를) 사용하여 최신 코드를 배포합니다.
1. `aio asset-compute devtool`을(를) 사용하여 Asset compute Devtool만 실행합니다. 열어 두십시오.
1. Visual Studio 코드 편집기에서 다음 디버그 구성을 `launch.json`에 추가합니다.

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   `aio app deploy`의 출력에서 `ACTION NAME`을(를) 가져옵니다.

1. 실행/디버그 구성에서 `wskdebug worker`을(를) 선택하고 재생 아이콘을 누릅니다. **[!UICONTROL Debug Console]** 창에 **[!UICONTROL 활성화 준비]**&#x200B;가 표시될 때까지 시작될 때까지 기다립니다.

1. Devtool에서 **[!UICONTROL 실행]**&#x200B;을 클릭합니다. Visual Studio 코드 편집기에서 실행 중인 작업을 볼 수 있으며 로그가 표시되기 시작합니다.

1. 코드에서 중단점을 설정합니다. 그런 다음 다시 실행하면 맞을 것입니다.

모든 코드 변경 사항은 실시간으로 로드되며 다음 활성화가 발생하는 즉시 유효합니다.

>[!NOTE]
>
>사용자 지정 응용 프로그램의 각 요청에 대해 두 개의 활성화가 존재합니다. 첫 번째 요청은 SDK 코드에서 자신을 비동기적으로 호출하는 웹 작업입니다. 두 번째 활성화는 코드를 히트하는 활성화입니다.
