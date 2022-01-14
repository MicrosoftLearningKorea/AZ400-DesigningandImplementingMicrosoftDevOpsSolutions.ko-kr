---
lab:
    title: '랩 14a: Ansible과 Azure'
    module: '모듈 14: Azure에서 사용 가능한 타사 IaC(코드 제공 인프라) 도구'
---

# 랩 14a: Ansible과 Azure
# 학생용 랩 매뉴얼

## 랩 개요

이 랩에서는 Ansible을 사용하여 Azure 리소스를 배포, 구성 및 관리합니다. 

선언형 구성 관리 소프트웨어인 Ansible은 관리 컴퓨터에 적용되는 적절한 구성의 설명(플레이북 형식)을 확인한 다음 해당 구성을 자동으로 적용하고 지속적으로 유지 관리함으로써 구성의 불일치 발생 가능성을 방지합니다. YAML을 사용하여 플레이북의 서식을 지정합니다.

Puppet, Chef 등 흔히 사용되는 기타 구성 관리 도구와는 달리 Ansible에는 에이전트가 없으므로 관리 컴퓨터에 소프트웨어를 설치하지 않아도 됩니다. Ansible은 SSH를 사용해 Linux 서버를, PowerShell 원격을 사용해 Windows 서버와 클라이언트를 관리합니다.

Azure Resource Manager를 통해 액세스 가능한 Azure 리소스 등 운영 체제 이외의 리소스와 상호 작용할 수 있도록 Ansible은 모듈이라는 확장을 지원합니다. Ansible은 Python으로 작성되었으므로 모듈도 Python 라이브러리로 구현됩니다. 그리고 Azure 리소스를 관리하기 위해 [GitHub에서 호스트되는 모듈](https://github.com/ansible-collections/azure)을 사용합니다.

Ansible을 사용하려면 관리되는 리소스가 지정된 호스트 인벤토리에 지정되어 있어야 합니다. Ansible은 Azure 등의 특정 시스템용 동적 인벤토리를 지원하므로 런타임 시 호스트 인벤토리가 동적으로 생성됩니다.

이 랩에서는 대략적으로 다음과 같은 단계를 진행합니다.

- Azure VM에서 Ansible 설치 및 구성
- Ansible 구성 및 샘플 플레이북 파일 다운로드
- Azure AD에서 관리 ID 만들기 및 구성
- Ansible에 사용할 Azure AD 자격 증명 및 SSH 구성
- Ansible 플레이북을 사용하여 Azure VM 배포
- Ansible 플레이북을 사용하여 Azure VM 구성

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Azure VM에서 Ansible 설치 및 구성
- Ansible 구성 및 샘플 플레이북 파일 다운로드
- Azure Active Directory 관리 ID 만들기 및 구성
- Ansible에 사용할 Azure AD 자격 증명 및 SSH 구성
- Ansible 플레이북을 사용하여 Azure VM 배포
- Ansible 플레이북을 사용하여 Azure VM 구성

## 랩 소요 시간

-   예상 시간: **90분**

## 지침

### 시작하기 전에

#### 랩 가상 머신에 로그인

다음 자격 증명을 사용하여 Windows 10 컴퓨터에 로그인했는지 확인합니다.
    
-   사용자 이름: **Student**
-   암호: **Pa55w.rd**

#### 이 랩에 필요한 애플리케이션 검토

이 랩에서 사용할 애플리케이션을 확인합니다.
    
-   Microsoft Edge

#### Azure 구독 준비

-   기존 Azure 구독을 확인하거나 새 구독을 만듭니다.
-   Azure 구독의 Owner 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다. 자세한 내용은 [Azure Portal을 사용하여 Azure 역할 할당 목록 표시](https://docs.microsoft.com/ko-kr/azure/role-based-access-control/role-assignments-list-portal) 및 [Azure Active Directory에서 관리자 역할 확인 및 할당](https://docs.microsoft.com/ko-kr/azure/active-directory/roles/manage-roles-portal#view-my-roles)을 참조하세요.

### 연습 1: Ansible을 사용하여 Azure VM 배포, 구성 및 관리

이 연습에서는 Ansible을 사용하여 Azure VM을 배포, 구성 및 관리합니다.

#### 작업 1: Ansible 제어 노드로 사용할 Azure VM 프로비전

이 작업에서는 Azure CLI를 사용하여 Azure VM을 배포한 다음 Ansible 환경을 관리하도록 Ansible 제어 노드로 구성합니다.

>**참고**: Ansible 제어 노드로 구성된 Azure VM을 사용하여 이 랩의 이전 작업에서 수행한 작업을 포함한 Ansible 관리 작업을 수행합니다. 

1.  Azure Portal의 도구 모음에서 검색 텍스트 상자 바로 오른쪽에 있는 **Cloud Shell** 아이콘을 클릭합니다. 

    >**참고**: [https://shell.azure.com](https://shell.azure.com)로 이동하여 Cloud Shell에 바로 액세스할 수도 있습니다.

1.  **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작할 때 **탑재된 스토리지가 없음** 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이 랩에서 배포하는 리소스를 호스트할 Azure 지역 이름을 지정합니다. 여기서 `<Azure_region>` 자리 표시자는 리소스를 배포할 Azure 지역 이름으로 바꿉니다. 이름에는 공백이 없어야 합니다(예: `westeurope`).

    ```bash
    LOCATION=<Azure_region>
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이 랩에서 배포하는 Azure VM을 호스트할 리소스 그룹을 만듭니다.

    ```bash
    RG1NAME=az400m14l03rg
    az group create --name $RG1NAME --location $LOCATION
    RG2NAME=az400m14l03arg
    az group create --name $RG2NAME --location $LOCATION
    ```

1.  다음 명령을 실행하여 Ubuntu를 실행하는 Azure VM을 이전 단계에서 만든 리소스 그룹에 배포합니다.

    ```bash
    VM1NAME=az400m1403vm1
    az vm create \
    --resource-group $RG1NAME \
    --name $VM1NAME \
    --image UbuntuLTS \
    --authentication-type password \
    --admin-username azureuser \
    --admin-password Pa55w.rd1234
    ```

    >**참고**: 배포가 완료될 때까지 기다린 후 다음 작업 진행합니다. 완료되려면 약 2분이 소요됩니다.

    >**참고**: 프로비전이 완료되면 JSON 기반 출력에 포함된 **"publicIpAddress"** 속성의 값을 확인합니다. 

1.  다음 명령을 실행하여 새로 배포한 Azure VM을 SSH을 사용하여 연결합니다.

    ```bash
    PIP=$(az vm show --show-details --resource-group $RG1NAME --name $VM1NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  계속 진행할지를 확인하라는 메시지가 표시되면 **yes**를 입력하고 **Enter** 키를 누릅니다. 암호를 입력하라는 메시지가 표시되면 **Pa55w.rd1234**를 입력합니다.


#### 작업 2: Azure VM에서 Ansible 설치 및 구성

이 작업에서는 이전 작업에서 배포한 Azure VM에서 Ansible을 설치하고 구성합니다.

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 최신 버전 및 패키지 세부 정보가 포함되도록 Advanced Packaging Tool(apt) 패키지 목록을 업데이트합니다.

    ```bash
    sudo apt-get update
    ```

1.  다음 명령을 실행하여 Ansible 및 필요한 Ansible 모듈을 설치합니다. **명령은 한 줄씩** 개별적으로 실행해야 하며 작업을 확인하라는 메시지가 표시되면 항상 **y**를 입력하고 **Enter** 키를 누릅니다.

    ```bash
    sudo apt install python3-pip
    sudo -H pip3 install --upgrade pip
    sudo -H pip3 install ansible[azure]
    sudo apt-add-repository --yes --update ppa:ansible/ansible
    sudo apt install ansible
    sudo ansible-galaxy collection install azure.azcollection
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    sudo pip3 install -r requirements-azure.txt
    rm requirements-azure.txt
    ```

    >**참고**: 경고는 무시합니다. 오류가 발생하면 명령을 다시 실행합니다.

1.  다음 명령을 실행하여 Ansible 플레이북이 배포 전에 DNS 이름을 확인할 수 있도록 dnspython 패키지를 설치합니다.

    ```bash
    sudo -H pip3 install dnspython
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 **jq** JSON 구문 분석 도구를 설치합니다. 메시지가 표시되면 **y**를 입력하고 **Enter** 키를 누릅니다.

    ```bash
    sudo apt install jq
    ```

1.  다음 명령을 실행하여 Azure CLI를 설치합니다.

    ```bash
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

#### 작업 3: Ansible 구성 및 샘플 플레이북 파일 다운로드

이 작업에서는 GitHub Ansible 구성 리포지토리에서 샘플 랩 파일과 함께 몇 가지 파일을 다운로드합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 **git**이 설치되어 있는지 확인합니다.

    ```bash
    sudo apt install git
    ```

1.  다음 명령을 실행하여 GitHub에서 PartsUnlimitedMRP 리포지토리를 복제합니다.

    ```bash
    git clone https://github.com/Microsoft/PartsUnlimitedMRP.git
    ```

    >**참고**: 이 리포지토리에는 다양한 리소스를 만드는 데 사용할 수 있는 플레이북이 포함되어 있습니다. 이 랩에서는 해당 플레이북 중 몇 가지를 사용합니다.


#### 작업 4: Azure Active Directory 관리 ID 만들기 및 구성

이 작업에서는 Azure 리소스에 액세스하려면 진행해야 하는 Ansible 비대화형 인증을 원활하게 진행하기 위해 Azure AD 관리 ID를 생성합니다. 그리고 이전 작업에서 만든 리소스 그룹의 Contributor 역할을 해당 관리 ID에 할당합니다.

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Azure 구독과 연결된 Azure AD 테넌트에 로그인합니다.

    ```bash
    az login
    ```

    >**참고**: 명령이 실패하면 Azure CLI 설치를 다시 실행합니다.

1.  이전 명령의 출력으로 표시된 코드를 확인하고 랩 컴퓨터로 전환합니다. 그런 다음 랩 컴퓨터의 브라우저 창에서 다른 탭을 열고 Azure Portal이 표시되면 [Microsoft 디바이스 로그인 페이지](https://microsoft.com/devicelogin)로 이동합니다. 메시지가 표시되면 코드를 입력하고 **다음**을 선택합니다.

1.  다시 메시지가 표시되면 이 랩에서 사용하는 자격 증명을 사용하여 로그인한 다음 브라우저 탭을 닫습니다.

1.  Cloud Shell 창의 Bash 세션으로 다시 전환합니다. Ansible 제어 노드로 구성된 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 시스템에서 할당한 관리 ID를 생성합니다.


    ```bash
    RG1NAME=az400m14l03rg
    VM1NAME=az400m1403vm1
    az vm identity assign --resource-group $RG1NAME --name $VM1NAME
    ```

1.  다음 명령을 실행하여 구독 값을 확인합니다.

    ```bash
    SUBSCRIPTIONID=$(az account show --query id --output tsv)
    ```

1.  다음 명령을 실행하여 기본 제공 Azure 역할 기반 액세스 제어 Contributor 역할의 **ID** 속성 값을 검색합니다. 

    ```bash
    CONTRIBUTORID=$(az role definition list --name "Contributor" --query "[].id" --output tsv)
    ```

1.  다음 명령을 실행하여 이 랩의 앞부분에서 만든 리소스 그룹에 Contributor 역할을 할당합니다. 

    ```bash
    MIID=$(az resource list --name $VM1NAME --query [*].identity.principalId --out tsv)

    RG2NAME=az400m14l03arg
    az role assignment create --assignee "$MIID" \
    --role "$CONTRIBUTORID" \
    --scope /subscriptions/$SUBSCRIPTIONID/resourceGroups/$RG2NAME
    ```

#### 작업 5: Ansible에서 사용할 SSH 구성

이 작업에서는 Ansible에서 사용할 SSH를 구성합니다.

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 키 쌍을 생성합니다. 메시지가 표시되면 **Enter** 키를 3번 눌러 파일 위치의 기본값을 수락하고 암호는 설정하지 않습니다.

    ```bash
    ssh-keygen -t rsa
    ```

1.  다음 명령을 실행하여 개인 키를 호스트하는 **.ssh** 폴더에서 읽기, 쓰기 및 실행 권한을 부여합니다.

    ```bash
    chmod 755 ~/.ssh
    ```

1.  다음 명령을 실행하여 **authorized_keys** 파일을 만들고 읽기 및 쓰기 권한을 설정합니다.

    ```bash
    touch ~/.ssh/authorized_keys
    chmod 644 ~/.ssh/authorized_keys
    ```

    >**참고**: 이 파일에 포함되어 있는 키를 제공하면 암호를 입력하지 않아도 액세스가 허용됩니다.

1.  다음 명령을 실행하여 **authorized_keys** 파일에 암호를 추가합니다.

    ```bash
    ssh-copy-id azureuser@127.0.0.1
    ```

1. 메시지가 표시되면 **yes**를 입력하고 이 랩의 앞부분에서 세 번째 Azure VM을 배포할 때 지정했던 **azureuser** 사용자 계정의 암호 **Pa55w.rd1234** 를 입력합니다. 
1.  다음 명령을 실행하여 암호를 입력하라는 메시지가 표시되지 않았는지 확인합니다.

    ```bash
    ssh 127.0.0.1
    ```

1.  **exit**를 입력하고 **Enter** 키를 눌러 방금 설정한 루프백 연결을 종료합니다. 

>**참고**: Ansible 환경을 설정할 때는 암호 없는 SSH 인증 설정 단계를 반드시 수행해야 합니다. 


#### 작업 6: Ansible 플레이북을 사용하여 웹 서버 Azure VM 만들기

이 작업에서는 Ansible 플레이북을 사용하여 웹 서버를 호스트하는 Azure VM을 만듭니다. 

>**참고**: 제어 Azure VM에서 Ansible이 작동되어 실행되고 있으므로 관리되는 Azure VM을 만들고 구성하는 데 사용할 첫 번째 플레이북을 배포할 수 있습니다. 샘플 플레이북을 배포하기 전에 플레이북의 내용에 포함되어 있는 공용 SSH 키를 이전 작업에서 생성한 키로 바꿔야 합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 노드로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 이전 작업에서 생성한 로컬에 저장된 공개 키를 확인합니다.

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```

1.  출력 문자열 끝부분의 사용자 이름을 포함하여 전체 출력을 적어 둡니다. 
1.  다음을 실행하여 코드 텍스트 편집기에서 **new_vm_web.yml** 파일을 엽니다.

    ```bash
    code ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml
    ```

1.  코드 편집기에서 필요한 경우 `dnsname: '{{ vmname }}.westeurope.cloudapp.azure.com'` 항목의 지역 이름을 배포 대상 Azure 지역 이름으로 변경합니다. 

    >**참고**: 이 지역이 **az400m14l03rg** 리소스 그룹에서 만든 Azure 지역과 일치하는지 확인합니다.

1.  코드 편집기에서 `vm_size` 항목의 값을 `Standard_A0`에서 `Standard_DS1_v2`로 변경합니다.
1.  코드 편집기에서 파일 끝부분의 `key_data` 항목에 포함된 SSH 문자열을 찾은 다음 기존 키 값을 삭제하고 이 작업의 앞부분에서 적어 둔 키 값으로 바꿉니다. 

    >**참고**: 파일에 포함된 `admin_username` 항목의 값이 Ansible 제어 시스템을 호스트하는 Azure VM에 로그인하는 데 사용한 사용자 이름(**azureuser**)과 일치하는지 확인합니다. `ssh_public_keys` 섹션의 `path` 항목에도 같은 사용자 이름을 사용해야 합니다.

1.  코드 편집기 인터페이스 내에서 오른쪽 상단의 **...** 를 클릭하고 **저장**을 선택합니다.

    >**참고**: 다음으로 랩을 시작할 때 만든 리소스 그룹에 Azure VM을 배포할 것입니다. 배포에는 다음 값을 사용합니다. 

    | 설정 | 값 |
    | --- | --- |
    | 리소스 그룹 | **az400m14l03arg** |
    | 가상 네트워크 | **az400m1403aVNET** |
    | 서브넷 | **az400m1403aSubnet** |

    >**참고**: 변수는 플레이북 내에서 정의할 수도 있고 런타임에 입력할 수도 있습니다. 런타임에 변수를 입력하는 경우 `ansible-playbook` 명령을 호출할 때 `--extra-vars` 옵션을 포함하면 됩니다. VM 이름은 소문자와 숫자만 포함하여 15자 이하로 입력합니다. 하이픈, 밑줄 기호, 대문자는 사용하지 마세요. 그리고 해당 Azure VM과 연결된 공용 IP 주소의 스토리지 계정 및 DNS 이름을 생성할 때도 같은 이름이 사용되므로, 전역적으로 고유한 이름을 지정해야 합니다. 

1.  다음 명령을 실행하여 Ansible 플레이북을 사용하여 Azure VM에 배포할 가상 네트워크 및 해당 서브넷을 만듭니다.

    ```bash
    RG1NAME=az400m14l03arg
    LOCATION=$(az group show --resource-group $RG1NAME --query location --output tsv)
    RG2NAME=az400m14l03arg
    VNETNAME=az400m1403aVNET
    SUBNETNAME=az400m1403aSubnet
    az network vnet create \
    --name $VNETNAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --address-prefixes 192.168.0.0/16 \
    --subnet-name $SUBNETNAME \
    --subnet-prefix 192.168.1.0/24
    ```

1.  다음 명령을 실행하여 Azure VM을 프로비전하는 샘플 Ansible 플레이북을 배포합니다. **`<VM_name>`은 선택한 고유 VM 이름으로 바꿔야 합니다**.

    ```bash
    sudo ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml --extra-vars "vmname=<VM_name> resgrp=az400m14l03arg vnet=az400m1403aVNET subnet=az400m1403aSubnet"
    ```

    >**참고**: setting ip_ 구성에 대한 사용 중단 경고는 무시합니다.

    >**참고**: 기존 또는 잘못된 VM 이름을 입력하면 다음 오류가 발생할 수 있습니다.

    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "이름이 storageaccountname인 스토리지 계정은 이미 사용되었습니다. - Reason.already_exists"}`. 이 문제를 해결하려면 Azure VM에 다른 이름을 사용하십시오. 사용한 이름은 전역적으로 고유하지 않습니다.
    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "your-vm-name을(를) 만들거나 업데이트하는 동안 오류가 발생했습니다. Azure 오류: InvalidDomainNameLabel\nMessage: VM의 도메인 이름 레이블이 잘못되었습니다. 다음 정규식을 따라야 합니다. ^[a-z][a-z0-9-]{1,61}[a-z0-9]$.”}`. 이 문제를 해결하려면 Azure VM에 필요한 명명 규칙을 따르는 다른 이름을 사용하십시오. 

    >**참고**: 배포가 완료될 때까지 기다립니다. 완료되려면 약 3분이 소요됩니다. 

1.  다음을 실행하여 새 파일 **myazure_rm.yml** 을 만든 다음 코드 텍스트 편집기에서 엽니다.

    ```bash
    code ./myazure_rm.yml
    ```

1.  코드 편집기 인터페이스에 다음 내용을 붙여넣습니다.

    ```bash
    plugin: azure_rm
    include_vm_resource_groups:
    - az400m14l03arg
    auth_source: msi

    keyed_groups:
    - prefix: tag
      key: tags
    ```

1.  코드 편집기 인터페이스 내에서 오른쪽 상단의 **...** 를 클릭하고 **저장**을 선택합니다.
1.  Cloud Shell 창에서 Bash 세션을 다시 표시합니다. 그런 다음 Ansible 제어 노드로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 ping 테스트를 수행합니다. 이 테스트에서는 새로 배포된 Azure VM이 동적 인벤토리 파일에 포함되어 있는지를 확인합니다.

    ```bash
    sudo ansible --user azureuser --private-key=/home/azureuser/.ssh/id_rsa all -m ping -i ./myazure_rm.yml
    ```

1.  연결을 계속할지 묻는 메시지가 표시되면 **yes**를 입력하고 **Enter** 키를 누릅니다.

    >**참고**: 출력은 다음과 유사합니다.

    ```bash
    az400m1403vm2_5444 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }
    ```

    >**참고**: 명령을 처음 실행할 때는 **yes**를 입력하고 **Enter** 키를 눌러 대상 VM을 신뢰할 수 있음을 승인해야 합니다. 


#### 작업 7: Ansible 플레이북을 사용하여 Azure VM 구성

이 작업에서는 다른 Ansible 플레이북을 실행하여 새로 배포한 Azure VM을 구성합니다. 여기서는 소프트웨어 패키지 httpd를 설치하고 GitHub 리포지토리에서 HTML 페이지를 다운로드하는 플레이북을 사용합니다. 이 작업을 완료하면 정상 작동하는 웹 서버가 완성됩니다.

>**참고**: 여기서는 **~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml** 샘플 플레이북을 사용합니다. 그리고 **vmname** 변수를 사용하여 플레이북의 hosts 매개 변수를 수정합니다. 이 매개 변수는 동적 인벤토리 스크립트에서 반환된 호스트 중 플레이북이 대상으로 사용할 호스트를 정의합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 노드로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 새로 배포한 Azure VM의 공용 IP 주소를 확인합니다. **`<VM_name>` 자리 표시자는 새로 프로비전한 Azure VM에 할당한 이름으로 바꿔야 합니다.**

    ```bash
    RGNAME='az400m14l03arg'
    VMNAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RGNAME --name $VMNAME --query publicIps --output tsv)
    ```

1.  다음 명령을 실행하여 새로 배포한 Azure VM에서 현재 웹 서비스를 실행하고 있지 않음을 확인합니다. 여기서 `<IP_address>` 자리 표시자는 이전 작업에서 프로비전한 Azure VM의 네트워크 어댑터에 할당된 공용 IP 주소를 나타냅니다.

    ```bash
    curl http://$PIP
    ```

    >**참고**: 응답이 `curl: (7) Failed to connect to 52.186.157.26 port 80: Connection refused` 형식인지 확인합니다.

1.  다음 명령을 실행하여 Ansible 플레이북을 사용해 HTTP 서비스를 설치합니다. 여기서 `<VM_name>` 자리 표시자는 이전 작업에서 프로비전한 VM의 이름을 나타냅니다. 

    ```bash
    sudo ansible-playbook --user azureuser --private-key=/home/azureuser/.ssh/id_rsa -i ./myazure_rm.yml ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>*"
    ```

    >**참고**: Azure VM 이름 다음에 후행 별표(**\***)를 포함해야 합니다.

    >**참고**: 설치가 완료될 때까지 기다립니다. 이 작업에는 1분 미만이 소요됩니다. 

1.  설치가 완료되면 다음 명령을 실행하여 새로 배포한 Azure VM에서 현재 웹 서비스를 실행하고 있음을 확인합니다. 여기서 `<IP_address>` 자리 표시자는 이전 작업에서 프로비전한 Azure VM의 네트워크 어댑터에 할당된 공용 IP 주소를 나타냅니다.

    ```bash
    curl http://$PIP
    ```

    >**참고**: 출력에는 다음 내용이 포함되어 있어야 합니다. 

    ```html
     <!DOCTYPE html>
     <html lang="en">
         <head>
             <meta charset="utf-8">
             <title>Hello World</title>
         </head>
         <body>
             <h1>Hello World</h1>
             <p>
                 <br>This is a test page
                 <br>This is a test page
                 <br>This is a test page
             </p>
         </body>
     </html>
    ```

### 연습 2: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.

## 복습

이 랩에서는 Ansible을 사용하여 Azure 리소스를 배포, 구성 및 관리하는 방법을 배웠습니다. 

