---
lab:
  title: '랩 10: LaunchDarkly 및 Azure DevOps를 통한 기능 플래그 관리'
  module: 'Module 04: Design and implement a release strategy'
ms.openlocfilehash: f2b752ebb825fea24c5eb1ba74a12929e5191022
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262603"
---
# <a name="lab-10-feature-flag-management-with-launchdarkly-and-azure-devops"></a>랩 10: LaunchDarkly 및 Azure DevOps를 통한 기능 플래그 관리
# <a name="student-lab-manual"></a>학생용 랩 매뉴얼

## <a name="lab-overview"></a>랩 개요

[**LaunchDarkly**](https://launchdarkly.com/)는 기능 플래그를 서비스로 제공하는 지속적인 업데이트 플랫폼입니다. LaunchDarkly는 기능 롤아웃을 코드 배포에서 분리하고 기능 플래그를 대규모로 관리할 수 있는 기능을 제공합니다. LaunchDarkly를 Azure DevOps와 통합하면 빈번한 릴리스 관련 위험 발생 가능성을 최소화할 수 있습니다. 릴리스를 개발 프로세스와 더욱 긴밀하게 통합하려는 경우 Azure DevOps 작업 항목에 기능 플래그 롤아웃을 연결할 수 있습니다. 

이 랩에서는 LaunchDarkly를 사용하여 Azure DevOps에서 기능 플래그 관리를 최적화하는 방법을 알아봅니다. 

## <a name="objectives"></a>목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- LaunchDarkly에서 기능 플래그 만들기
- LaunchDarkly와 웹 애플리케이션 통합
- Azure DevOps 릴리스 파이프라인의 일환으로 LaunchDarkly 기능 플래그 자동 롤아웃

## <a name="lab-duration"></a>랩 소요 시간

-   예상 시간: **60분**

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>시작하기 전에

#### <a name="sign-in-to-the-lab-virtual-machine"></a>랩 가상 머신에 로그인

다음 자격 증명을 사용하여 Windows 10 가상 머신에 로그인해야 합니다.
    
-   사용자 이름: **Student**
-   암호: **Pa55w.rd**

#### <a name="review-applications-required-for-this-lab"></a>이 랩에 필요한 애플리케이션 검토

이 랩에서 사용할 애플리케이션을 확인합니다.
    
-   Microsoft Edge
-   [Visual Studio 다운로드 페이지](https://visualstudio.microsoft.com/vs)에서 제공되는 Visual Studio 2019 Community Edition. Visual Studio 2019 설치에는 **ASP.NET 및 웹 개발**, **Azure 개발**, **.NET Core 플랫폼 간 개발** 워크로드가 포함되어야 합니다. 이러한 워크로드는 랩 컴퓨터에 미리 설치되어 있습니다.

#### <a name="set-up-a-launchdarkly-trial-account"></a>LaunchDarkly 평가판 계정 설정

웹 브라우저를 사용하여 [LaunchDarkly 웹 사이트](https://launchdarkly.com/)로 이동한 다음 평가판 계정을 만듭니다. 

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 조직 설정

[조직 또는 프로젝트 컬렉션 만들기](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)에서 제공되는 지침을 따르세요.

#### <a name="prepare-an-azure-subscription"></a>Azure 구독 준비

-   기존 Azure 구독을 확인하거나 새 구독을 만듭니다.
-   Azure 구독의 Owner 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다. 자세한 내용은 [Azure Portal을 사용하여 Azure 역할 할당 목록 표시](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal) 및 [Azure Active Directory에서 관리자 역할 확인 및 할당](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)을 참조하세요.

### <a name="exercise-0-configure-the-lab-prerequisites"></a>연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Azure DevOps 데모 생성기 템플릿을 기반으로 하여 미리 구성된 Parts Unlimited 팀 프로젝트를 설정합니다. 

#### <a name="task-1-configure-the-team-project"></a>작업 1: 팀 프로젝트 구성

이 작업에서는 Azure DevOps 데모 생성기를 사용하여 **Parts Unlimited** 템플릿을 기반으로 새 프로젝트를 생성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure DevOps 데모 생성기](https://azuredevopsdemogenerator.azurewebsites.net)로 이동합니다. 이 유틸리티 사이트에서는 계정 내에서 새 Azure DevOps 프로젝트를 만드는 프로세스를 자동으로 진행할 수 있습니다. 이 프로젝트에는 랩에 필요한 작업 항목, 리포지토리 등의 콘텐츠가 미리 입력되어 있습니다. 

    > **참고**: 사이트에 대한 자세한 내용은 https://docs.microsoft.com/en-us/azure/devops/demo-gen 을 참조하세요.

1.  **로그인** 을 클릭하고 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  필요한 경우 **Azure DevOps 데모 생성기** 페이지에서 **수락** 을 클릭하여 Azure DevOps 구독에 액세스하는 데 필요한 권한 요청을 수락합니다.
1.  **새 프로젝트 만들기** 페이지의 **새 프로젝트 이름** 텍스트 상자에 **LaunchDarkly** 를 입력합니다. 그런 다음 **조직 선택** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **템플릿 선택** 을 클릭합니다.
1.  템플릿 목록의 도구 모음에서 **DevOps 랩** 을 클릭하고 **LaunchDarkly** 템플릿을 선택한 다음 **템플릿 선택** 을 클릭합니다.
1.  **새 프로젝트 만들기** 페이지로 돌아와서 누락된 확장을 설치하라는 메시지가 표시되면 **LaunchDarkly 통합 V2** 레이블 아래의 체크박스를 선택하고 **프로젝트 만들기** 를 클릭합니다.

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 이 작업은 2분 정도 걸립니다. 프로세스가 실패하면 DevOps 조직으로 이동하여 프로젝트를 삭제한 후에 다시 시도합니다.

1.  **새 프로젝트 만들기** 페이지에서 **프로젝트로 이동** 을 클릭합니다.

### <a name="exercise-1-configure-feature-flags-in-azure-devops-by-using-launchdarkly"></a>연습 1: LaunchDarkly를 사용하여 Azure DevOps에서 기능 플래그 구성

이 연습에서는 LaunchDarkly를 사용하여 Azure DevOps에서 기능 플래그를 구성합니다. 

#### <a name="task-1-create-a-feature-flag-in-launchdarkly"></a>작업 1: LaunchDarkly에서 기능 플래그 만들기

이 작업에서는 LaunchDarkly에서 기능 플래그를 만듭니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [LaunchDarkly 웹 사이트](https://app.launchdarkly.com/)로 이동하고 사용자 계정을 만듭니다. 브라우저 세션이 **Default Project** 포털로 리디렉션됩니다. 이 포털에서 기능 플래그를 만들 수 있습니다. 
1.  LaunchDarkly 포털 왼쪽의 세로 메뉴 모음에 있는 **Feature flags** 를 클릭합니다.
1. **Feature flags** 창에서 **Create FLAG** 를 클릭합니다.
1.  **Create a feature flag** 창의 **Name** 텍스트 상자에 **Member portal** 을 입력하고 **SAVE FLAG** 단추를 클릭합니다.

    > **참고**: 그러면 **Member Portal** 플래그가 작성됩니다. ASP.NET MVC 웹앱에서 **Member Portal** 기능 표시 유형을 결정하는 데 이 플래그를 사용하려는 경우를 가정해 보겠습니다. 

    > **참고**: 애플리케이션에 LaunchDarkly를 통합하려면 SDK 키가 필요합니다. 

1.  LaunchDarkly 포털 왼쪽의 세로 메뉴 모음에 있는 **계정 설정** 을 클릭한 다음 **프로젝트** 탭을 클릭합니다.

    > **참고**: **계정 설정** 창의 **프로젝트** 탭에서 미리 정의된 다음 두 환경을 확인할 수 있습니다. **프로덕션** 및 **텍스트**  이 프로젝트에는 production 환경 SDK 키를 사용하면 됩니다. 

1.  프로덕션 환경의 SDK 키 키를 적어 둡니다. 이 랩의 후반부에서 필요합니다.

#### <a name="task-2-integrate-launchdarkly-in-your-web-application"></a>작업 2: 웹 애플리케이션에 LaunchDarkly 통합

이 작업에서는 웹 애플리케이션에 LaunchDarkly를 통합합니다.

1.  랩 컴퓨터에서 **LaunchDarkly** 프로젝트가 열려 있는 Azure DevOps 포털이 표시된 웹 브라우저 창으로 전환하여 프로젝트 창 왼쪽 아래에서 **프로젝트 설정** 을 클릭합니다.
1.  **프로젝트 설정** 세로 메뉴 모음의 **Repos** 섹션에서 **리포지토리** 를 선택합니다.
1.  **모든 리포지토리** 창에서 **LaunchDarkly** 를 클릭하고 **리포지토리 설정** 창에서 **멘션 연결 커밋** 및 **멘션 작업 항목 확인 커밋** 설정이 **켜기** 로 설정되어 있는지 확인합니다.
1.  Azure DevOps 포털 맨 왼쪽의 세로 메뉴 모음에 있는 **Repos** 를 클릭하고 **파일** 창을 보고 있는지 확인합니다.

    > **참고**: 리포지토리가 비어 있는 경우 **리포지토리 가져오기** 섹션에서 **가져오기** 를 선택한 다음 **Git 리포지토리 가져오기** 창의 **복제 URL** 텍스트 상자에 `https://github.com/hsachinraj/PartsUnlimited.git`을 입력한 후 **가져오기** 를 클릭합니다.

1.  **파일** 창에서 **복제** 를 클릭합니다.
1. **리포지토리 복제** 창의 **IDE** 드롭다운 목록에서 **Visual Studio** 를 선택합니다. 메시지가 표시되면 **Microsoft Visual Studio Web Protocol Handler Selector 열기** 를 선택합니다. 그러면 Visual Studio가 자동으로 시작됩니다. Azure DevOps 구독과 연결된 Microsoft 계정으로 로그인합니다. 
1.  Visual Studio 창 내의 **Azure DevOps** 대화 상자에서 **복제** 를 클릭하고 메시지가 표시되면 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다. 
1.  Visual Studio 창 내에서 최상위 **Git** 메뉴를 클릭하고 **로컬 리포지토리**, **폴더** 를 차례로 클릭합니다. **폴더 선택** 에서 LaunchDarkly 리포지토리를 복제한 폴더로 이동하여 **폴더 선택** 을 클릭합니다.
1.  Visual Studio Code 창 내에서 최상위 **보기** 메뉴를 클릭하고 드롭다운 메뉴에서 **Git 변경 내용** 을 클릭합니다. 
1. Visual Studio 창 내의 **Git 변경 내용** 창 위쪽에 있는 **master** 드롭다운 목록에서 아래쪽 화살표를 클릭하고 드롭다운 대화 상자에서 **원격** 을 클릭합니다. 그런 다음 원격 분기 목록에서 **origin/launch-darkly** 를 선택합니다. **체크 아웃** 을 선택합니다. 

    > **참고**: 이렇게 하면 **launch-darkly** 분기가 자동으로 체크 아웃됩니다. 

1.  Visual Studio Code 창 내에서 **솔루션 탐색기** 창으로 전환한 다음 **PartsUnlimited.sln** 을 두 번 클릭하여 솔루션을 엽니다. 

    > **참고**: **LaunchDarkly** 를 .NET 애플리케이션과 통합하려면 **LaunchDarkly 클라이언트** 가 포함된 NuGet 패키지를 설치해야 합니다. 현재 프로젝트에서는 작업을 쉽게 진행할 수 있도록 해당 패키지가 이미 추가되어 있습니다.

1.  **솔루션 탐색기** 창에서 **PartsUnlimitedWebsite** 노드를 확장하고 **Dependencies** 하위 노드를 마우스 오른쪽 단추로 클릭한 후에 마우스 오른쪽 클릭 메뉴에서 **NuGet 패키지 관리** 항목을 선택합니다.
1.  **NuGet: PartsUnlimitedWebsite** 창에서 **LaunchDarkly.Client** 가 이미 설치되어 있음을 확인합니다.
1.  Visual Studio 인터페이스의 위쪽 메뉴에서 **IIS Express** 를 클릭하여 애플리케이션을 로컬에서 시작합니다. 
1.  애플리케이션이 로컬 브라우저 세션에서 정상적으로 시작되며, 웹 인터페이스 오른쪽 위에 **Member Portal** 섹션이 있는지 확인합니다.

    > **참고**: **Member Portal** 모듈은 새로운 기능이며 **LaunchDarkly 기능 플래그** 를 사용하여 이 기능을 제어하려는 경우를 가정해 보겠습니다. 이 경우 LaunchDarkly에서 플래그를 설정하면 기능이 사용자에게 표시됩니다.

1.  웹 애플리케이션 인터페이스가 표시된 웹 브라우저 창을 닫습니다.
1.  Visual Studio 창의 **솔루션 탐색기** 에서 **PartsUnlimitedWebsite\\Controllers\\HomeController.cs** 로 이동하여 해당 파일을 열고 파일 내용을 [업데이트된 HomeController.cs 코드 조각](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/HomeController.cs)에서 제공되는 코드로 바꾼 후에 변경 내용을 저장합니다. 
1.  Visual Studio 창의 **솔루션 탐색기** 에서 **PartsUnlimitedWebsite\\Controllers\\AccountController.cs** 로 이동하여 해당 파일을 열고 파일 내용을 [업데이트된 AccountController.cs 코드 조각](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/AccountController.cs)에서 제공되는 코드로 바꾼 후에 변경 내용을 저장합니다. 
1.  Visual Studio 창의 **솔루션 탐색기** 에서 **PartsUnlimitedWebsite\\Views\Shared\\_Layout.chtml** 로 이동하여 해당 파일을 열고 줄 55 `@await Html.PartialAsync("_Login")`를 아래 코드로 바꿉니다.

    ```cshtml
    @if (ViewBag.togglevalue == true)
    {
      @await Html.PartialAsync("_Login")
    }
    ```

1.  **Home Controller.cs** 파일의 내용이 표시된 탭으로 전환하여 `static LdClient client = new LdClient("__YourLaunchDarklySDKKey__");` 줄의 `__YourLaunchDarklySDKKey__` 자리 표시자를 이 연습의 이전 작업에서 복사한 LaunchDarkly 계정 SDK 키로 바꿉니다. 

    > **참고**: 그러면 환경용 SDK를 사용하여 LdClient 개체가 작성됩니다.

1.  **AccountController.cs** 파일의 내용이 표시된 탭으로 전환하여 `static LdClient client = new LdClient("__YourLaunchDarklySDKKey__");` 줄의 `__YourLaunchDarklySDKKey__` 자리 표시자를 이 연습의 이전 작업에서 복사한 LaunchDarkly 계정 SDK 키로 바꿉니다. 

    > **참고**: 위의 변경을 수행하면 정적 LaunchDarkly 클라이언트가 초기화되어 **HomeController** 가 시작됩니다. **MemberPortal** 을 확인하는 메서드는 LaunchDarkly에서 기능 플래그 토글의 상태(켜기 또는 끄기)를 확인하도록 수정됩니다. **_Layout.cshtml** 페이지에서 토글 값을 확인한 다음 플래그가 켜져 있으면 MemberPortal 링크를 렌더링합니다. 

1.  Visual Studio 창에서 **HomeController.cs** 탭으로 다시 전환한 후 이 파일의 내용을 검토합니다. 코드 줄 57~74의 내용을 특히 자세히 확인하세요.

    ```
        //LaunchDarkly start
        User user = LaunchDarkly.Client.User.WithKey("administrator@test.com");
        bool value = client.BoolVariation("member-portal", user, false);
        if (value)
        {
          ViewBag.Message = "Your application description page.";
          ViewData["togglevalue"] = value;
          return View(viewModel);
        }
        else
        {
          return View(viewModel);
        }
        // return View(viewModel);

    }
    //LaunchDarkly End
    ```

    > **참고**: 기능 플래그를 요청할 때는 user 개체를 전달해야 합니다. 그러므로 위의 코드 시작 부분에서는 user 개체가 초기화됩니다. 이 개체는 지정된 키를 소유한 사용자가 LaunchDarkly에 있는지 여부를 확인하는 데 사용됩니다. 이 샘플에서는 user 값을 하드 코드했습니다. 실제 시나리오에서는 데이터베이스 조회를 수행하는 로그인한 사용자를 확인하여 이 값을 검색할 수 있습니다.

    > **참고**: 다음으로는 **BoolVariation** 메서드를 호출하여 LaunchDarkly에서 기능 플래그 값을 확인합니다. 플래그가 true이면 **[ViewData["togglevalue"]** 도 true로 설정됩니다. 이 값은 **_Layout.chtml** 에서 Member Portal 모듈을 확인하는 데 사용됩니다. 플래그가 false이면 Member Portal 모듈이 표시되지 않습니다.

    > **참고**: 마찬가지로 **AccountController.cs** 에서도 **Login()** 메서드에 LaunchDarkly 코드를 추가했습니다. 이 메서드는 **Member portal** 아이콘을 클릭하고 나면 Login 페이지를 표시하는 데 사용됩니다. LaunchDarkly의 플래그가 false이면 Login 페이지에서 HttpNotFound 오류가 반환됩니다.

1. Visual Studio 창에서 **모두 저장** 을 클릭한 다음 **IIS Express** 를 클릭하여 애플리케이션을 로컬에서 시작합니다. **IIS Express** 옵션을 사용할 수 없는 경우 도구 모음에서 **브라우저 링크** 옵션을 선택하고 **브라우저 링크 대시보드** 를 선택합니다. 대시보드에서 **브라우저에서 보기** 를 선택합니다. 
1.  Parts Unlimited 웹 사이트가 표시된 웹 브라우저에 **Member portal** 링크가 더 이상 표시되지 않음을 확인합니다. 이 시점에서는 **MemberPortal** 플래그가 꺼져 있기 때문에 해당 링크가 표시되지 않습니다.

    > **참고**: 이제 LaunchDarkly를 사용하는 기능 플래그 제어가 구현되었습니다. 따라서 LaunchDarkly 포털에서 토글을 수동으로 사용하도록 설정할 수 있습니다. 하지만 이 랩에서는 LaunchDarkly 확장을 사용하여 Azure DevOps 릴리스 파이프라인을 통해 기능 플래그를 구성합니다. 여기서는 릴리스 프로세스의 일부분으로 기능 플래그를 포함하기 위해 이 변경 작업을 Azure DevOps 작업 항목과 연결합니다. 

1.  웹 애플리케이션 인터페이스가 표시된 웹 브라우저 창을 닫습니다.
1.  필요한 경우 Visual Studio Code 창 내에서 최상위 **보기** 메뉴를 클릭하고 드롭다운 메뉴에서 **팀 탐색기** 를 클릭합니다.
1. **팀 탐색기** 창의 **홈** 페이지에서 **작업 항목** 을 클릭합니다. 메시지가 표시되면 링크를 선택하여 새 허브를 봅니다.
1.  **팀 탐색기** 창의 **작업 항목** 페이지에서 이 분기와 관련된 단일 작업 항목을 기록합니다(작업 항목 ID 포함). 
1.  해당 작업 항목을 나타내는 항목을 두 번 클릭합니다. 그러면 다른 웹 브라우저 탭이 자동으로 열리고 Azure DevOps 포털에 작업 항목이 표시됩니다. 
1.  작업 항목 창에서 작업 항목이 자신에게 할당되어 있는지 확인합니다. 
1. Visual Studio 창으로 다시 전환하여 **Git 변경 내용** 창으로 이동한 후 **메시지 입력** 텍스트 상자에 **Integated LaunchDarkly #<work\_item\_ID>** 를 입력합니다. 여기서 **<work\_item\_ID>** 는 앞에서 확인한 ID를 나타냅니다. 그런 다음 **모두 커밋** 단추 옆에 있는 아래쪽 화살촉을 클릭하고 드롭다운 목록에서 **모두 커밋 및 푸시** 를 클릭합니다. **모두 커밋** 이 회색으로 비활성화된 경우 위쪽 화살표를 선택하여 푸시를 수행할 수 있습니다. 

#### <a name="task-3-automatically-rollout-launchdarkly-feature-flags-during-release"></a>작업 3: 릴리스 중에 LaunchDarkly 기능 플래그 자동 롤아웃

이 작업에서는 Azure DevOps 릴리스 중에 LAunchDarkly 기능 플래그가 자동으로 롤아웃되도록 구성합니다.

> **참고**: Azure DevOps Services와의 통합에 사용할 LaunchDarkly 액세스 토큰이 필요합니다. 

1.  LaunchDarkly 액세스 토큰을 검색하려면 LaunchDarkly 포털이 표시된 브라우저 창으로 다시 전환한 다음 왼쪽의 세로 메뉴에서 **Account settings** 를 클릭합니다. 그런 다음 **Account settings** 창에서 **Authorization** 탭을 클릭하고 **Access tokens** 섹션에서 **CREATE TOKEN** 을 클릭합니다.
1.  **Create an access token** 창의 **Name** 텍스트 상자에 **Member portal** 을 입력하고 **Role** 드롭다운 목록에서 **Writer** 를 선택한 다음 **SAVE TOKEN** 을 클릭합니다.
1.  **Account settings** 창의 **Authorization** 탭으로 돌아와 액세스 토큰을 복사한 다음 메모장에 붙여넣습니다. 

    > **참고**: 지금 토큰을 복사해야 합니다. 현재 페이지에서 다른 위치로 이동하면 토큰을 검색할 수 없습니다.

1.  Azure DevOps 포털이 표시된 브라우저 창으로 다시 전환하여 **LaunchDarkly** 프로젝트 창에서 **프로젝트 설정** 을 클릭합니다. 
1. **프로젝트 세부 정보** 창의 **파이프라인** 섹션에서 **서비스 연결**, **서비스 연결 만들기** 를 차례로 클릭합니다.
1.  **새 서비스 연결** 창에서 **LaunchDarkly** 를 클릭하고 **다음** 을 클릭합니다.
1.  **새 LaunchDarkly 서비스 연결** 창의 **액세스 토큰** 텍스트 상자에 이 작업 앞부분에서 검색한 액세스 토큰을 붙여넣고 **서비스 연결 이름** 에는 **az-400 m12l01 LaunchDarkly** 를 입력한 후에 **저장** 을 클릭합니다.
1.  Azure DevOps 포털 왼쪽의 세로 메뉴에서 **Boards** 를 클릭하고 **작업 항목** 창에서 이전 작업에서 자신에게 할당한 작업 항목을 클릭합니다.
1.  **LaunchDarkly를 사용하여 기능 플래그 관리 구현** 작업 항목 창 오른쪽 위의 **Launch Darkly** 탭을 선택합니다. 
1.  **LauchDarkly** 탭의 **기능 플래그 선택** 섹션에서 **Member portal** 기능 플래그를 선택합니다.
1.  Azure DevOps 포털 왼쪽의 세로 메뉴 모음에 있는 **Pipelines** 를 클릭하고 **파이프라인** 섹션에서 **릴리스** 를 선택합니다. 
1.  **파이프라인/릴리스** 창에서 **LaunchDarkly_CD** 파이프라인을 선택하고 **편집** 을 클릭합니다.
1.  **모든 파이프라인/LaunchDarkly_CD** 창에서 단계 **1단계** 의 **작업 1개, 태스크 3개** 링크를 클릭하여 파이프라인의 태스크를 확인합니다.
1.  이 단계에 다음 태스크가 포함되어 있는지 확인합니다.

    - **Azure 리소스 그룹 배포**: 이 태스크에서는 ARM 템플릿을 사용하여 PartsUnlimited 웹 사이트용 Azure App Service를 배포합니다.
    - **LaunchDarkly 롤아웃**: 이 태스크에서는 LaunchDarkly 구독에서 기능 플래그를 켭니다.
    - **Azure App Service 배포**: 이 태스크에서는 첫 번째 작업에서 만든 Azure App Service에 PartsUnlimited 웹앱을 배포합니다.

1.  태스크 목록에서 **Azure 리소스 그룹 배포** 태스크를 선택하고 **Azure 구독** 드롭다운 목록에서 아래쪽 캐럿을 클릭합니다. 그런 다음 항목 목록에서 이 랩에서 사용할 Azure 구독을 클릭하고 **권한 부여** 를 클릭하여 Azure 서비스 연결을 구성합니다. 메시지가 표시되면 Azure 구독의 Owner 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 계정을 사용하여 로그인합니다.
1.  태스크 목록에서 **LaunchDarkly 롤아웃** 태스크를 선택하고 **계정** 드롭다운 목록에서 이 작업 앞부분에서 만든 LaunchDarkly 서비스 연결을 나타내는 항목을 선택합니다.

    > **참고**: **플래그 상태** 가 **켜기** 로 설정되어 있는지 확인합니다. 이 설정은 릴리스 중의 플래그 상태를 제어합니다.

1.  태스크 목록에서 **Azure App Service 배포** 태스크를 선택하고 **Azure 구독** 드롭다운 목록에서 앞에서 만든 Azure 구독 서비스 연결을 선택한 다음 **저장** 을 클릭합니다.
1.  Azure DevOps 포털에서 Azure DevOps 페이지 오른쪽 위의 **사용자 설정** 아이콘을 클릭합니다. 그런 다음 드롭다운 메뉴에서 **개인용 액세스 토큰** 을 클릭하고 **개인용 액세스 토큰** 창에서 **+ 새 토큰** 을 클릭합니다.
1.  **새 개인용 액세스 토큰 만들기** 창에서 다음 설정을 지정한 후에 **만들기** 를 클릭합니다. 다른 값은 모두 기본값으로 유지합니다.

    | 설정 | 값 |
    | --- | --- |
    | Name | **LaunchDarkly 및 Azure DevOps 랩** |
    | 범위 | **모든 권한** |

1.  **성공** 창에서 개인용 액세스 토큰의 값을 클립보드에 붙여넣습니다.

    > **참고**: 지금 토큰을 복사해야 합니다. 이 창을 닫으면 토큰을 검색할 수 없습니다. 

1.  **성공** 창에서 **닫기** 를 클릭합니다.
1.  **모든 파이프라인/LaunchDarkly_CD** 창으로 다시 이동하여 **편집**, **변수** 탭을 차례로 클릭합니다. 
1.  변수 목록에서 **launchdarkly-pat** 변수의 값을 새로 생성한 개인용 액세스 토큰으로 설정합니다.
1.  변수 목록에서 **launchdarkly-project-name** 변수의 값을 **LaunchDarkly** 로 설정하고 **저장** 을 클릭합니다.

    > **참고**: 이것으로 릴리스 파이프라인 구성이 완료되었습니다. 

1.  Azure DevOps 포털 왼쪽의 세로 메뉴 모음에 있는 **Pipelines** 섹션에서 **파이프라인** 을 선택합니다.
1. **파이프라인** 창에서 **LaunchDarkly-CI** 빌드 파이프라인을 나타내는 항목을 클릭하고 **LaunchDarkly-CI** 창에서 **파이프라인 실행** 을 클릭합니다. 그런 다음 **파이프라인 실행** 창에서 **실행** 을 클릭합니다.

    > **참고**: 이 CI 파이프라인은 .NET Core 프로젝트를 컴파일합니다. 빌드가 정상적으로 완료되면 릴리스가 트리거되어 LaunchDarkly에서 앱이 배포되고 기능 플래그가 롤아웃됩니다.

1.  빌드 파이프라인 실행 창의 **작업** 섹션에서 **에이전트 작업 1** 항목을 클릭하고 빌드 프로세스 진행률을 모니터링합니다.
1.  빌드가 완료되면 Azure DevOps 포털 왼쪽의 세로 메뉴 모음에 있는 **Pipelines** 섹션에서 **릴리스** 를 선택합니다.
1.  **파이프라인/릴리스** 창의 **LaunchDarkly_CD** 창에서 **릴리스-1** 항목을 클릭하고 배포 프로세스 진행률을 모니터링합니다.

    > **참고**: 배포가 정상적으로 완료되면 LaunchDarkly 대시보드에서 **MemberPortal** 기능 플래그가 켜짐을 확인할 수 있습니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [**Azure Portal**](https://portal.azure.com)로 이동한 다음 이 랩에서 사용하는 Azure 구독에서 Owner 역할이 지정된 사용자 계정으로 로그인합니다.
1.  Azure Portal에서 **App Services** 리소스를 검색하여 선택하고 **App Services** 블레이드에서 Azure DevOps 릴리스 파이프라인을 통해 애플리케이션을 배포한 웹앱으로 이동합니다.
1.  웹앱 블레이드에서 **찾아보기** 를 클릭합니다. 그러면 다른 웹 브라우저 탭이 열리고 새로 배포한 웹 애플리케이션이 표시됩니다.
1.  Parts Unlimited 웹 페이지에서 **Member Portal** 기능이 사용하도록 설정되어 있는지 확인합니다.

> **참고**: LaunchDarkly에서는 다음과 같은 여러 가지 기타 기능도 제공합니다.
    
- **사용자 대상 지정**: LaunchDarkly 대상 지정 기능을 사용하면 개별 사용자나 사용자 그룹을 대상으로 기능을 켜거나 끌 수 있습니다. 이 기능을 통해 광범위한 롤아웃을 수행하기 전에 내부 테스트, 비공개 베타 또는 유용성 테스트용으로 기능을 롤아웃할 수 있습니다. 
- **사용자 지정 대상 지정 규칙**: LaunchDarkly에서는 개별 사용자를 대상으로 지정할 수 있을 뿐 아니라 사용자 지정 규칙을 생성하여 사용자 세그먼트를 대상으로 지정할 수도 있습니다. 즉, 사용자 지정 규칙을 만들어 지정한 특성을 기준으로 대상 사용자를 지정할 수 있습니다. 
- **개발 프로세스 관리용 프로젝트 및 환경**: [프로젝트](https://docs.launchdarkly.com/docs/projects)를 사용하면 LaunchDarkly 계정 하나에서 여러 소프트웨어 프로젝트를 관리할 수 있습니다. [환경](https://docs.launchdarkly.com/docs/environments)을 사용하면 로컬 개발, QA, 스테이징, 프로덕션 등을 포함한 전체 개발 수명 주기에서 기능 플래그를 관리할 수 있습니다. 

### <a name="exercise-2-remove-the-azure-lab-resources"></a>연습 2: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스는 모두 제거하세요. 사용되지 않는 리소스를 제거하면 예기치 않은 요금이 발생하지 않습니다.

#### <a name="task-1-remove-the-azure-lab-resources"></a>작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Portal을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal에서 이 랩 앞부분에서 배포한 App Service 인스턴스로 이동합니다. 
1.  App Service 인스턴스 블레이드에서 App Service 인스턴스가 포함된 리소스 그룹의 이름을 나타내는 링크를 클릭합니다.
1.  App Service 인스턴스가 포함된 리소스 그룹 블레이드에서 **리소스 그룹 삭제** 를 클릭하고 메시지가 표시되면 리소스 그룹 이름을 입력한 후에 **삭제** 를 클릭합니다.

## <a name="review"></a>검토

이 랩에서는 LaunchDarkly를 사용하여 Azure DevOps에서 기능 플래그 관리를 최적화하는 방법을 알아보았습니다. 
