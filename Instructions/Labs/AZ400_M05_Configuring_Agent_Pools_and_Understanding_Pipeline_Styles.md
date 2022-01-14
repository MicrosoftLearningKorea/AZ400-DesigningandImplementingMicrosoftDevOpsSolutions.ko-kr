---
lab:
    title: '랩 05: 에이전트 풀 구성 및 파이프라인 스타일 이해'
    module: '모듈 5: Azure Pipelines 구성'
---

# 랩 05: 에이전트 풀 구성 및 파이프라인 스타일 이해
# 학생용 랩 매뉴얼

## 랩 개요

YAML 기반 파이프라인을 사용하면 CD/CI를 코드로 완벽하게 구현할 수 있습니다. 이 구현에서는 파이프라인 정의가 Azure DevOps 프로젝트에 포함된 코드와 같은 리포지토리에 저장됩니다. YAML 기반 파이프라인은 끌어오기 요청, 코드 검토, 기록, 분기, 템플릿 등 클래식 파이프라인에서 제공되는 폭넓은 기능을 지원합니다. 

어떤 파이프라인 스타일을 선택하든 Azure Pipelines를 사용하여 솔루션을 배포하거나 코드를 작성하려면 에이전트가 필요합니다. 작업을 한 번에 하나씩 실행하는 에이전트는 컴퓨팅 리소스를 호스트합니다. 작업은 에이전트의 호스트 머신에서 바로 실행할 수도 있고 컨테이너에서 실행할 수도 있습니다. Microsoft에서 호스트하는 에이전트(자동으로 관리됨)를 사용하여 작업을 실행하거나 직접 설정 및 관리하는 자체 호스팅 에이전트를 구현할 수 있습니다. 

이 랩에서는 클래식 파이프라인을 YAML 기반 파이프라인으로 변환한 다음, 먼저 Microsoft에서 호스트된 에이전트를 사용하여 실행한 다음 자체 호스팅 에이전트를 사용하여 동일 작업을 수행하는 프로세스를 단계별로 진행합니다.

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- YAML 기반 파이프라인 구현
- 자체 호스팅 에이전트 구현

## 랩 소요 시간

-   예상 시간: **45분**

## 지침

### 시작하기 전에

#### 랩 가상 머신에 로그인

다음 자격 증명을 사용하여 Windows 10 컴퓨터에 로그인했는지 확인합니다.
    
-   사용자 이름: **Student**
-   암호: **Pa55w.rd**

#### 설치된 애플리케이션 검토

Windows 10 데스크톱에서 작업 표시줄을 찾습니다. 작업 표시줄에는 이 랩에서 사용할 애플리케이션에 대한 아이콘이 포함되어 있습니다.
    
-   Microsoft Edge
-   [Visual Studio Code](https://code.visualstudio.com/). 이 랩의 필수 구성 요소로 설치됩니다.

#### Azure DevOps 조직 설정

이 랩에 사용할 수 있는 Azure DevOps 조직이 아직 없으면 [조직 또는 프로젝트 컬렉션 만들기](https://docs.microsoft.com/ko-kr/azure/devops/organizations/accounts/create-organization?view=azure-devops)에서 제공되는 지침에 따라 조직을 만듭니다.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Azure DevOps 데모 생성기 템플릿을 기반으로 하여 미리 구성된 Parts Unlimited 팀 프로젝트를 설정합니다.

#### 작업 1: 팀 프로젝트 구성

이 작업에서는 Azure DevOps 데모 생성기를 사용하여 **PartsUnlimited** 템플릿을 기반으로 새 프로젝트를 생성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure DevOps 데모 생성기](https://azuredevopsdemogenerator.azurewebsites.net)로 이동합니다. 이 유틸리티 사이트에서는 계정 내에서 새 Azure DevOps 프로젝트를 만드는 프로세스를 자동으로 진행할 수 있습니다. 이 프로젝트에는 랩에 필요한 작업 항목, 리포지토리 등의 콘텐츠가 미리 입력되어 있습니다. 

    > **참고**: 이 사이트에 대한 자세한 내용은 https://docs.microsoft.com/ko-kr/azure/devops/demo-gen을 참조하세요.

1.  **로그인**을 클릭하고 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  필요한 경우 **Azure DevOps 데모 생성기** 페이지에서 **수락**을 클릭하여 Azure DevOps 구독에 액세스하는 데 필요한 권한 요청을 수락합니다.
1.  **새 프로젝트 만들기** 페이지의 **새 프로젝트 이름** 텍스트 상자에 **에이전트 풀 구성 및 파이프라인 스타일 이해**를 입력합니다. 그런 다음 **조직 선택** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **템플릿 선택**을 클릭합니다.
1.  **템플릿 선택** 페이지에서 **PartsUnlimited** 템플릿을 클릭한 다음 **템플릿 선택**을 클릭합니다.
1.  **프로젝트 만들기**를 클릭합니다.

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 약 2분이 소요됩니다. 프로세스가 실패하면 DevOps 조직으로 이동하여 프로젝트를 삭제한 후에 다시 시도합니다.

1.  **새 프로젝트 만들기** 페이지에서 **프로젝트로 이동**을 클릭합니다.

### 연습 1: YAML 기반 Azure DevOps 파이프라인 작성

이 연습에서는 클래식 Azure DevOps 파이프라인을 YAML 기반 파이프라인으로 변환합니다. 

#### 작업 1: Azure DevOps YAML 파이프라인 만들기

이 작업에서는 템플릿 기반 Azure DevOps YAML 파이프라인을 만듭니다.

1.  **에이전트 풀 구성 및 파이프라인 스타일 이해** 프로젝트가 열린 Azure DevOps 포털이 표시되어 있는 웹 브라우저 왼쪽의 세로 탐색 창에서 **Pipelines**를 클릭합니다. 
1.  **파이프라인** 창의 **최근** 탭에서 **새 파이프라인**을 클릭합니다.
1.  **코드 위치** 창에서 **Azure Repos Git**를 클릭합니다. 
1.  **리포지토리 선택** 창에서 **PartsUnlimited**를 클릭합니다.
1.  **파이프라인 구성** 창에서 **시작 파이프라인**을 클릭합니다.
1.  **파이프라인 YAML 검토** 페이지에서 샘플 파이프라인을 검토하고 **저장 및 실행** 단추 옆의 아래쪽 캐럿 기호를 클릭한 다음 **저장**을 클릭하고 **저장** 창에서 **저장**을 클릭합니다.

    > **참고**: 이렇게 하면 프로젝트 코드를 호스트하는 리포지토리의 루트 디렉터리에 **azure-pipelines.yml** 파일이 생성됩니다.

    > **참고**: 여기서는 샘플 YAML 파이프라인 내용을 클래식 편집기로 생성한 파이프라인의 코드로 바꾼 다음, 클래식 파이프라인과 YAML 파이프라인의 차이를 반영하여 해당 코드를 수정할 것입니다.

#### 작업 2: 클래식 파이프라인을 YAML 파이프라인으로 변환

이 작업에서는 클래식 파이프라인을 YAML 파이프라인으로 변환합니다.

1.  **에이전트 풀 구성 및 파이프라인 스타일 이해** 프로젝트가 열린 Azure DevOps 포털이 표시되어 있는 웹 브라우저 왼쪽의 세로 탐색 창에서 **Pipelines**를 클릭합니다. 
1.  **파이프라인** 패널의 **최근** 탭에서 **PartsUnlimitedE2E** 항목이 포함된 항목의 오른쪽 모서리 위에 마우스를 놓으면 **자세히** 메뉴를 나타내는 세로 줄임표가 표시됩니다. 이 줄임표 기호를 클릭하고 드롭다운 메뉴에서 **편집**을 클릭합니다. 그러면 랩을 시작할 때 생성한 프로젝트에 포함되어 있는 빌드 파이프라인이 표시됩니다. 
1.  **PartsUnlimitedE2E** 편집 패널의 **작업** 탭에서 **트리거**를 클릭하고 **PartsUnlimited** 창 오른쪽에서 **연속 통합 사용** 체크박스의 선택을 취소합니다. 그런 다음 **저장 및 큐에 넣기** 단추 옆에 있는 아래쪽 캐럿을 클릭하고 드롭다운 메뉴에서 **저장**을 클릭한 후에 **빌드 파이프라인 저장**에서 **저장**을 클릭합니다.

    > **참고**: 이렇게 하면 이 랩에서 리포지토리 변경으로 인해 자동 빌드가 의도치 않게 실행되는 상황을 방지할 수 있습니다.

    > **참고**: 기존 파이프라인의 내용을 새 파이프라인에 복사한 후에 기존 파이프라인을 삭제해도 됩니다.

1.  Azure DevOps 포털 왼쪽의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **파이프라인**을 클릭합니다. 
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimitedE2E** 항목을 클릭합니다. 
1.  **PartsUnlimitedE2E** 창의 **실행** 탭 오른쪽 위에서 세로 줄임표(세로 점 3개) 기호를 클릭한 다음 드롭다운 메뉴에서 **YAML로 내보내기**를 클릭합니다. 이렇게 하면 로컬 **다운로드** 폴더에 **PartsUnlimitedE2E.yml** 파일이 자동으로 다운로드됩니다.

    > **참고**: **YAML로 내보내기** 기능은 이전에 Azure DevOps 포털 내 파이프라인 편집기 창에서 제공되었던 **YAML 보기** 옵션 대신 제공됩니다. 이전 옵션 사용 시에는 YAML 내용을 한 번에 한 작업씩만 확인할 수 있었습니다. 새 기능은 기존의 클래식 파이프라인 인프라와 YAML 파이프라인 인프라(YAML 구문 분석 라이브러리 포함)를 모두 활용하므로 두 인프라 간의 더욱 정확한 변환이 가능합니다. 이 기능은 다음과 같은 파이프라인 구성 요소를 지원합니다.
    > - 단일 작업/여러 작업
    > - 체크 아웃 옵션
    > - 실행 계획 병렬 처리
    > - 시간 제한 및 이름 메타데이터
    > - 수요
    > - 일정 및 기타 트리거
    > - 풀 선택(기본값과 다른 작업 포함)
    > - 모든 작업과 모든 입력(기본 입력 최적화 포함)
    > - 작업 및 단계 조건
    > - 작업 그룹 언롤

    > **참고**: 새 기능에서 지원되지 않는 구성 요소는 변수 및 표준 시간대 변환뿐입니다. YAML에서 정의된 변수는 Azure DevOps 포털 사용자 인터페이스에 설정된 변수를 재정의합니다. **YAML로 내보내기** 기능 사용 시 클래식 파이프라인 변수가 검색되면 새로 생성되는 YAML 파이프라인 정의 내의 주석에 해당 변수가 명시적으로 포함됩니다. 마찬가지로 YAML의 cron 일정은 UTC로 표시되므로 조직 표준 시간대를 사용하는 클래식 일정 역시 주석에 포함됩니다.

    > **참고**: 이 기능에 대한 자세한 내용은 ["YAML 보기" 대신 제공되는 기능](https://devblogs.microsoft.com/devops/replacing-view-yaml/)을 참조하세요.

1.  랩 컴퓨터에서 Visual Studio Code를 시작한 다음 Visual Studio Code를 사용하여 **PartsUnlimitedE2E.yml** 파일을 엽니다. 이 파일의 내용은 다음과 같습니다.

    ```yaml
    name: $(date:yyyyMMdd)$(rev:.r)
    jobs:
    - job: Phase_1
      displayName: Phase 1
      cancelTimeoutInMinutes: 1
      pool:
        vmImage: vs2017-win2016
      steps:
      - checkout: self
      - task: NuGetInstaller@0
        name: NuGetInstaller_1
        displayName: NuGet restore
        inputs:
          solution: '**\*.sln'
      - task: VSBuild@1
        name: VSBuild_2
        displayName: Build solution
        inputs:
          vsVersion: 15.0
          msbuildArgs: /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.stagingDirectory)" /p:IncludeServerNameInBuildInfo=True /p:GenerateBuildInfoConfigFile=true /p:BuildSymbolStorePath="$(SymbolPath)" /p:ReferencePath="C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\Extensions\Microsoft\Pex"
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: VSTest@1
        name: VSTest_3
        displayName: Test Assemblies
        inputs:
          testAssembly: '**\$(BuildConfiguration)\*test*.dll;-:**\obj\**'
          codeCoverageEnabled: true
          vsTestVersion: latest
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: CopyFiles@2
        name: CopyFiles1
        displayName: Copy Files
        inputs:
          SourceFolder: $(build.sourcesdirectory)
          Contents: '**/*.json'
          TargetFolder: $(build.artifactstagingdirectory)
      - task: PublishBuildArtifacts@1
        name: PublishBuildArtifacts_5
        displayName: Publish Artifact
        inputs:
          PathtoPublish: $(build.artifactstagingdirectory)
          TargetPath: '\\my\share\$(Build.DefinitionName)\$(Build.BuildNumber)'
    ...
    ```

1.  Visual Studio Code 창 최상위 메뉴에서 **선택**을 클릭하고 드롭다운 메뉴에서 **모두 선택**을 클릭합니다.
1.  Visual Studio Code 창 최상위 메뉴에서 **편집**을 클릭하고 드롭다운 메뉴에서 **복사**를 클릭합니다.
1.  Azure DevOps 포털이 표시된 브라우저 창으로 전환하여 왼쪽의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **파이프라인**을 클릭합니다. 
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimited**를 선택하고 **PartsUnlimited** 창에서 **편집**을 선택합니다.
1.  **PartsUnlimited** 편집 창에서 파이프라인의 기존 YAML 내용을 선택한 후 클립보드의 내용으로 바꿉니다.
1.  CI 트리거를 추가하고, 이 줄들을 파일의 맨 위에 붙여넣습니다.
    ```yaml
    trigger:
    - master
    ```
1.  **PartsUnlimited** 편집 창 오른쪽 위의 **저장**을 클릭하고 **저장** 창에서 **저장**을 클릭합니다. 그러면 이 파이프라인을 기반으로 하는 빌드가 자동으로 트리거됩니다. 
1.  Azure DevOps 포털 왼쪽의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **파이프라인**을 클릭합니다.
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimited** 항목을 클릭하고 **PartsUnlimited** 창의 **실행** 탭에서 최신 실행을 선택합니다. 그런 다음 해당 실행의 **요약** 창에서 아래쪽으로 스크롤하여 **작업** 섹션에서 **1단계**를 클릭하고 작업이 정상적으로 완료될 때까지 모니터링합니다. 

### 연습 2: Azure DevOps 에이전트 풀 관리

이 연습에서는 자체 호스팅 Azure DevOps 에이전트를 구현합니다.

#### 작업 1: Azure DevOps 자체 호스팅 에이전트 구성

이 작업에서는 LOD VM을 Azure DevOps 자체 호스팅 에이전트로 구성한 다음 빌드 파이프라인을 실행하는 데 사용합니다.

1.  Lab VM(Lab Virtual Machine) 또는 개인 컴퓨터를 내에서 웹 브라우저를 시작하고 [Azure DevOps 포털](https://dev.azure.com)로 이동한 다음 Azure DevOps 조직과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  Azure DevOps 포털에서 Azure DevOps 페이지 오른쪽 위의 **사용자 설정** 아이콘을 클릭하면 미리 보기 기능이 켜져 있는지 여부에 따라 메뉴에서 **보안** 또는 **개인용 액세스 토큰** 항목이 표시됩니다. **보안**이 표시되면 이를 클릭한 다음 **개인용 액세스 토큰**을 선택합니다. **개인용 액세스 토큰** 창에서 **+ 새 토큰**을 클릭합니다.
1.  **새 개인용 액세스 토큰 만들기** 창에서 **모든 범위 표시** 링크를 클릭하고 다음 설정을 지정한 후에 **만들기**를 클릭합니다. 다른 값은 모두 기본값으로 유지합니다.

    | 설정 | 값 |
    | --- | --- |
    | 이름 | **에이전트 풀 구성 및 파이프라인 스타일 이해 랩** |
    | 범위(사용자 정의됨) | **에이전트 풀**(필요하면 아래에 더 많은 범위 옵션 표시)|
    | 사용 권한 | **읽기 및 관리** |

1.  **성공** 창에서 개인용 액세스 토큰의 값을 클립보드에 붙여넣습니다.

    > **참고**: 지금 토큰을 복사해야 합니다. 이 창을 닫으면 토큰을 검색할 수 없습니다. 

1.  **성공** 창에서 **닫기**를 클릭합니다.
1.  Azure DevOps 포털에서 **개인용 액세스 토큰** 창 왼쪽 위의 **Azure DevOps** 기호를 클릭한 다음 왼쪽 아래의 **조직 설정** 레이블을 클릭합니다.
1.  **개요** 창 왼쪽의 세로 메뉴에 있는 **Pipelines** 섹션에서 **에이전트 풀**을 클릭합니다.
1.  **에이전트 풀** 창의 오른쪽 위에서 **풀 추가**를 클릭합니다. 
1.  **에이전트 풀 추가** 창의 **풀 유형** 드롭다운 목록에서 **자체 호스팅**을 선택하고 **이름** 텍스트 상자에 **az400m05l05a-pool** 을 입력한 다음 **만들기**를 클릭합니다.
1.  **에이전트 풀** 창으로 돌아와서 새로 만든 **az400m05l05a-pool** 을 나타내는 항목을 클릭합니다. 
1.  **az400m05l05a-pool** 창의 **작업** 탭에서 **에이전트** 머리글을 클릭합니다.
1.  **az400m05l05a-pool** 창의 **에이전트** 탭에서 **새 에이전트** 단추를 클릭합니다.
1.  **에이전트 가져오기** 창에서 **Windows** 및 **x64** 탭이 선택되어 있는지 확인하고 **다운로드**를 클릭하여 에이전트 이진 파일이 포함된 zip 보관 파일을 사용자 프로필 내의 로컬 **다운로드** 폴더에 다운로드합니다.

    > **참고**: 이 시점에서 현재 시스템 설정으로 인해 파일을 다운로드할 수 없다는 오류 메시지가 표시되면 Internet Explorer 창 오른쪽 위에 있는 **설정** 메뉴 머리글을 나타내는 톱니바퀴 기호를 클릭합니다. 그런 다음 드롭다운 메뉴에서 **인터넷 옵션**을 선택하고 **인터넷 옵션** 대화 상자에서 **고급**을 클릭합니다. 그리고 나서 **고급** 탭의 **원래대로**를 클릭하고 **Internet Explorer 기본 설정 복원** 대화 상자에서 **다시 설정**을 다시 클릭한 후에 **닫기**를 클릭하고 다운로드를 다시 시도해 봅니다. 

1.  관리자 권한으로 Windows PowerShell을 시작하고, **관리자: Windows PowerShell** 콘솔에서 다음 명령을 실행하여 **C:\\agent** 디렉터리를 만든 후에 다운로드한 보관 파일의 압축을 풀어 해당 내용을 이 디렉터리에 저장합니다. 

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

1.  동일한 **관리자: Windows PowerShell** 콘솔에서 다음 명령을 실행하여 에이전트를 구성합니다.

    ```powershell
    .\config.cmd
    ```

1.  메시지가 표시되면 다음 설정의 값을 지정합니다.

    | 설정 | 값 |
    | ------- | ------ |
    | 서버 URL 입력 | **https://dev.azure.com/`<organization_name>`** 형식의 Azure DevOps 조직 URL. 여기서 `<organization_name>`은 Azure DevOps 조직의 이름을 나타냅니다. |
    | 인증 유형 입력(PAT의 경우 Enter 키 누르기) | **Enter** |
    | 개인용 액세스 토큰 입력 | 이 작업의 앞부분에서 적어 둔 액세스 토큰 |
    | 에이전트 풀 입력(기본값을 사용하려면 Enter 키 누르기) | **az400m05l05a-pool** |
    | 에이전트 이름 입력 | **az400m05-vm0** |
    | 작업 폴더 입력(_work를 사용하려면 Enter 키 누르기) | **Enter** |
    | 각 단계의 작업에서 압축 풀기 수행 여부 입력 (N을 입력하려면 Enter 키 누르기) | **Enter** |
    | 에이전트를 서비스로 실행할지 여부 입력 (Y/N)(N을 입력하려면 Enter 키 누르기) | **Y** |
    | 서비스에 사용할 사용자 계정 입력(NT AUTHORITY\NETWORK SERVICE에 대해 Enter 키 누르기) | **Enter** |
    | 구성을 마친 후에 서비스가 즉시 시작되는 것을 방지할 것인지 여부를 입력합니다. (Y/N)(N을 입력하려면 Enter 키 누르기) | **Enter** |

    > **참고**: 자체 호스팅 에이전트는 서비스로 실행할 수도 있고 대화형 프로세스로 실행할 수도 있습니다. 처음에는 대화형 모드를 사용하는 것이 좋습니다. 이렇게 하면 에이전트 기능을 쉽게 확인할 수 있기 때문입니다. 프로덕션 환경에서는 자동 로그온을 사용하도록 설정한 대화형 프로세스나 서비스로 에이전트를 실행해야 합니다. 이 두 모드에서는 실행 상태가 영구 보존되며, 운영 체제를 다시 시작할 때 에이전트가 자동으로 다시 시작되기 때문입니다.

    > **참고**: 에이전트가 **작업 수신 중** 상태를 보고하는지 확인합니다.

1.  Azure DevOps 포털이 표시된 브라우저 창으로 전환하여 **에이전트 가져오기** 창을 닫습니다.
1.  **az400m05l05a-pool** 창의 **에이전트** 탭으로 돌아와서 새로 구성한 에이전트가 **온라인** 상태로 목록에 표시되는지 확인합니다.
1.  Azure DevOps 포털이 표시된 웹 브라우저 창의 왼쪽 위에서 **Azure DevOps** 레이블을 클릭합니다.
1.  프로젝트 목록이 표시된 브라우저 창에서 **에이전트 풀 구성 및 파이프라인 스타일 이해** 프로젝트를 나타내는 타일을 클릭합니다. 
1.  **에이전트 풀 구성 및 파이프라인 스타일 이해** 창 왼쪽의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **파이프라인**을 클릭합니다. 
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimited**를 선택하고 **PartsUnlimited** 창에서 **편집**을 선택합니다.
1.  **PartsUnlimited** 편집 창의 기존 YAML 기반 파이프라인에서 대상 에이전트 풀을 지정하는 줄 **7** `vmImage: vs2017-win2016`을 새로 만든 자체 호스팅 에이전트 풀을 지정하는 다음 내용으로 바꿉니다.

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```
1. `Task: NugetInstaller@0`에 대해 **설정(작업 위에 회색으로 표시되는 링크)** 을 클릭하고 **고급 > NuGet 버전 > 4.0.0** 을 수정하고 **추가**를 클릭합니다. 
1.  **PartsUnlimited** 편집 창 오른쪽 위의 **저장**을 클릭하고 **저장** 창에서 **저장**을 다시 클릭합니다. 그러면 이 파이프라인을 기반으로 하는 빌드가 자동으로 트리거됩니다. 
1.  Azure DevOps 포털 왼쪽의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **파이프라인**을 클릭합니다.
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimited** 항목을 클릭하고 **PartsUnlimited** 창의 **실행** 탭에서 최신 실행을 선택합니다. 그런 다음 해당 실행의 **요약** 창에서 아래쪽으로 스크롤하여 **작업** 섹션에서 **1단계**를 클릭하고 작업이 정상적으로 완료될 때까지 모니터링합니다. 

#### 복습

이 랩에서는 클래식 파이프라인을 YAML 기반 파이프라인으로 변환하는 방법과 자체 호스팅 에이전트를 구현 및 사용하는 방법을 배웠습니다.
