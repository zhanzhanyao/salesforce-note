# Salesforce DX (Development Experience)
Salesforce 推出的一个新的开发和部署模式，旨在提供更好的开发者体验。  
## Salesforce DX
Salesforce DX（下面简称SFDX）有两个主要优点：
- SFDX 符合快速迭代的开发模式
- SFDX 提供了统一的命令行工具在代码库和服务器之间进行通信和同步

![SDFX](image/SDFX.png)

### 1. sandbox & scretch
sandbox的缺陷
- 在多个功能同时进行开发的情况下，需要同时拥有多个开发的沙盒，即上图的“Developer Sandbox”。这些沙盒没法同时拥有所有已经开发的功能。  
- 随着项目的发展，越来越多的沙盒会因为没有特定的功能而“过时”，这就需要创建新的沙盒或刷新已有的沙盒。而这些操作需要耗费大量的时间

为了改进这种缺陷，SFDX 提供了一种新的概念：Scratch org。  
- Scratch org 是一种可以快速搭建的“沙盒”，只包含了特定的功能。开发者可以通过 SFDX 中的设置快速创建 Scratch org，在上面开发新的功能，然后将所做的更改直接提取出来放到代码库中进行整合。  
- 通过这样一种流程，之前笨重的“开发沙盒”就被轻盈的 Scratch org 所取代。

### 2. Salesforce命令行工具  
Salesforce 随着 SFDX 一起推出了命令行工具（下面简称 CLI）。  
在此之前，开发者一般使用官方的 Force.com IDE 或其他第三方插件（比如 MavensMate）来进行本地代码和服务器之间的通信。  
CLI 作为整合这些功能的工具而出现。它可以和各种代码编辑器结合使用，包含了在开发中需要的各种操作和命令，让开发者可以更方便的将本地修改上传到各种 Scratch org 中或者将 Scratch org 中的修改同步到本地代码库。

### 3. SFDX项目

#### 建立SFDX项目

        创建项目
        sfdx force:project:create -n ProjectName
        项目目录结构
        ProjectName
            config
                project-scratch-def.json    # 建立 Scratch org 的基本配置信息
            force-app    # 用来存放代码
                main
                    default
                        aure
            sfdx-project.json    # 包含该项目的基本信息

#### 建立Scratch org

        进入SFDX项目的目录
        输入以下命令建立Scratch org:
        sfdx force:org:create -s -f config/project-scratch-def.json -a ExampleScratchOrg
        打开刚建立的org
        sfdx force:org:open -u ExampleScratchOrg

#### 使用Scratch org
在 Scratch org 中，开发者（或管理员）可以和使用真正的 Salesforce 环境一样开发或配置功能。  
于此同时，Salesforce CLI 工具也提供了一系列的命令来进行相关的操作，比如建立 Apex 类、建立 Lightning 组件等。
#### 代码库与org同步
SFDX 的一个重要功能就是将代码库与 Scratch org 同步：
- 将更改从org同步到本地：开发者可以登录 Scratch org 进行开发和配置。当完成开发后，可以在 SFDX 中使用命令pull来将更改同步到本地的代码库。

      sfdx force:source:pull
- 将更改从本地代码库同步到org：在本地代码库中，也可以新建一些组件，比如 Apex 类或 Lightning 组件等，然后通过命令“sfdx force:source:push”将其同步到 Scratch org 中。

      sfdx force:source:push




