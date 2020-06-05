---
layout: post
title: 管理 Microstation CE 版本的环境配置
category: MS环境
tagline: by 明不知昔
tags: 
  - Microstation
published: true
mermaid: true
---

本文将解释如何构建和管理 MicroStation CONNECT Edition（以下简称 MS ） 的环境配置

<!--more-->



## 概述

Microstation 通过配置变量（configuration variables）来引导软件配置向正确的位置。

**配置变量** 被组织成以 _USTN_ 前缀开始的 **框架配置变量（Framework Configuration Variables）**和以 MS_ 为前缀开始的 **操作配置变量（Operational Configuration Variables)**。通常，框架配置变量用于设置基本路径，而操作配置变量用于指导 MS 内的程序流程。一些框架配置变量由MicroStation安装文件夹决定。其他框架配置变量默认设置为相对于安装文件夹的位置，但是可以在用户提供或编辑的配置文件中更改。

MS 的 配置文件处理可以看作是一个简单的程序，其中一部分由系统配置文件提供，不应该由用户修改，另一部分由用户可修改的配置文件提供。所有配置文件都是简单的文本文件，可以使用任何文本编辑器检查，修改。

系统配置文件位于 [C:\Program Files\Bentley\MicroStation CONNECT Edition\MicroStation\config](C:\Program Files\Bentley\MicroStation CONNECT Edition\MicroStation\config) 文件夹,同时用户可以修改的配置文件位于安装目录或其他指定的目录。系统配置文件在适当的时候将用户可修改的配置文件包含到配置文件处理流中。

### MS 配置

#### MS 默认配置目录

MS 默认配置目录为 [ C:\Program Files\Bentley\MicroStation CONNECT Edition\MicroStation\Default\ ](C:\Program Files\Bentley\MicroStation CONNECT Edition\MicroStation\Default)

#### MS 配置目录结构
``` mermaid
graph LR;
0{配置结构}-->1[Configuration];
    1-->11(Workspaces);
        11-->111(Example);
            111-->1111(Standards);
                1111-->11111(Cell);
                1111-->11112(DgnLib);
                    11112-->111121(GUI);
            111-->1112(WorkSets);
                1112-->11121(Building);
                    11121-->111211(Data);
                    11121-->111212(DGN);
                    11121-->111213(DWG);
                    11121-->111214(Out);
                    11121-->111215(Standars);
                        111215-->1112151(Cell);
                        111215-->1112152(DgnLib);
                        111215-->1112153(Seed);
                        111215-->1112154(Etc...);
                1112-->11122(Civil);
                    11122-->111221(Data);
                    11122-->111222(Etc...);
                1112-->11123(General);
                    11123-->111231(Data);
                    11123-->111232(Etc...);
                1112-->11124(Etc...);
0-->2[Organization];
    2-->21(Cell);
    2-->22(Data);
    2-->23(Dgn);
    2-->24(DgnLib);
        24-->241(ClashDetection);
        24-->242(DrawComp);
        24-->243(Etc...);
    2-->25(Macros);
    2-->26(Mareials);
    	26-->261(Bump);
    	26-->262(Pattern);
    2-->27(MdlApps);
    	27-->271(InteInt)
    2-->28(Pltcfg);
    2-->29(Seed);
    2-->210(Spc);
    2-->211(Symb);
    2-->212(Tables);
    	212-->2121(Dwg);
    	212-->2122(PEN)
    2-->213(VBA);
```
<!--
上述为mermaid图，可能因为网站问题，不能显示，可见下面的文字表示：

- 配置结构
  - Configuration
    - Workspaces
    	- Example            
    	- Standards
    		- Cell
    		- DgnLib
    			- GUI
    - WorkSets
    - Building
    		- Data
    		- DGN
    		- DWG
    		- Out
    		- Standars
    			- Cell
    			- DgnLib
    			- Seed
    			- Etc...
    	- Civil
    		- Data
    		- Etc...
    	- General
    		- Data
    		- Etc...
    - Etc...
  - Organization
    - Cell

    - Data

    - Dgn

    - DgnLib

    - ClashDetection
      - DrawComp
      - Etc...

    - Macros

    - Mareials

    - Bump

    - Pattern

    - MdlApps

      - InteInt

    - Pltcfg

    - Seed

    - Spc

    - Symb

    - Tables
      - Dwg
      - PEN

    - VBA
-->
	
#### 关于Configuration目录

此处至少有两个文件夹和一个文件，如果是基于 MS 平台的其它产品，还可能会有更多的文件夹及文件。

\Organization 文件夹是组织 CAD 标准所在的位置。可以通过在配置文件中使用 _USTN_ORGANIZATION（ORD是_USTN_CUSTOM_CONFIGURATION）指向不同的位置。该配置在 ConfigurationSetup.cfg中。

```
_USTN_CUSTOM_CONFIGURATION=C:\MyOrganization\
或者：_USTN_CUSTOM_CONFIGURATION=你的服务器配置地址\
```

MicroStation CONNECT Edition 中的 \WorkSpaces 文件夹是工作集、设计文件、标准文件和配置文件的容器。.cfg 文件是一个配置文件，可以将MS 的环境配置重定向到服务器或其他位置上。
	
#### 关于Organization目录

组织文件夹是存储自己的组织、单元格、dgnlib和其他与CAD标准相关的数据的地方。这些文件夹是空的，通过配置 standard .cfg 文件告诉MicroStation在何处查找种子文件等特定项。


### 配置变量定义及语法

### 配置文件加载顺序

配置文件处理从配置文件 mslocal.cfg 开始。它是启动 MicroStation 时打开的第一个文件，是一个简短的文件。mslocal.cfg 包含 msdir.cfg，msdir.cfg 是在安装时生成的另一个小配置文件，它标识 MicroStation 安装文件夹，然后包含 msconfig.cfg.cfg 文件包含处理事件的大部分配置文件。

永远不要修改 msconfig.cfg 本身(或 MicroStation 程序文件夹中的任何其他配置文件)，也不要在这个位置添加自己的配置文件。msconfig.cfg 中有许多已经定义好的，包含用户可修改的配置文件。在这些用户可修改的配置文件中，可以修改配置变量，以提供所有必要的灵活性，以满足组织对项目数据位置和 CAD 标准的需求。

当然，如果已经对配置文件理解得很透彻，则任何配置文件都可以修改。

msconfig.cfg 配置文件首先设置 _USTN_BENTLEYROOT 配置变量和一些框架配置变量，这些配置变量指向安装程序数据的目录。这些是程序操作所必需的，且不为任何用户数据或文件定义的位置。然后还包含MicroStation附带的系统和应用程序配置文件。

msconfig.cfg 文件被完全重新组织，行内文档解释了它的作用。msconfig.cfg 文件没有定义任何MS_xxx(操作型)配置变量。

下面是配置文件的加载顺序：

```
mslocal.cfg-->msdir.cfg-->msconfig.cfg
```

在 msconfig.cfg 文件中，会按照顺序先后加载配置配置文件，配置文件的顺序按照出现在 msconfig.cfg 中定义的先后顺序，注意不要修改其顺序，修改后可能导致系统出现意外的错误。加载顺序如下：

```
all system cfg files
all appl cfg files
workspacesetup.cfg files
all organization cfg files
personal.ucf file
workspace<name>.cfg file
all workspace cfg files
workset<name>.cfg files
all workset cfg files
role.cfg file
database cfg file
```





> ###### 参考资料

[1] . [Managing Configurations in MicroStation CONNECT Edition](learn.bentley.com)
[2]. [help](c:\\)