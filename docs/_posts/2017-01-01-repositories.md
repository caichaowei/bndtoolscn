# Bndtools 仓库

使用和配置仓库

目录

- 1 [索引仓库（Indexed Repositories）](#1)
  - 1.1 [固定索引仓库（Fixed Index Repositories）](#11)
  - 1.2 [本地索引仓库（Local Indexed Repository）](#12)
- 2 [文件仓库](#2)
- 3 [Aether (Maven)仓库](#3)
  - 3.1 [使用限制](#31)
- 4 [Maven仓库(老式风格)](#4)
  - 4.1 [Maven本地仓库](#41)
  - 4.2 [Maven远程仓库](#42)

Bndtools使用 **仓库（repositories）** 来提供在构建时（Build-time）和运行时（Rumtime）所需的依赖。一个仓库本质上是一个由Bundle以及各种索引组成的集合。
它可能被存放在各种位置上：在工作空间中；在本地文件系统的某个位置；或者在一个远程服务器上。

Bndtools按如下方式来使用仓库：

- 根据Bundle符号名（Bundle Symbolic Name ,BSN）和版本号查找Build Path上的Bundle；
- 解析运行需求列表（Run Requirements list）；
- 根据BSN和版本号在运行Bundle列表（Run Bundles list）中查找Bundle。

仓库被实现成了一个bnd插件，可以通过编辑工作空间配置文件（Bndtools菜单项 » Open main bnd config）的Plugins部分来配置其参数。

![](https://caichaowei.github.io/bndtoolscn/images/repositories/repositories01.png)

由于仓库被实现成了插件，因此理论上可以通过开发新的插件来支持几乎所有类型的仓库；不过使用已有的仓库插件会更为方便。
Bnd和Bndtools目前已经支持了如下仓库类型：

<span id="1"/>

## 1 索引仓库（Indexed Repositories）

Bndtools支持很多基于索引文件的仓库，索引文件中列出了仓库中的内容，并且列出了每种资源的情况和依赖需求。如下索引格式是可用的：

- OBR，命名来自OSGi Bundle Repository 伪标准。这个格式已经被废弃了，但是有些OSGi生态系统仍在使用它。
- R5, 命名来自OSGi Release 5，它引入了真正的OSGi仓库规范。此种索引格式在OSGi Release 5 Enterprise 规范的132.5章节部分进行了说明。
- 其它各种格式可通过增加IRepositoryContentProvider插件来获得支持。

使用索引仓库的好处在于可以在bndrun编辑器中进行自动解析（Resolution）。这里有两种基本类型的索引仓库：

<span id="11"/>

### 1.1 固定索引仓库（Fixed Index Repositories）

此种仓库可以使用一个存放在任意位置的索引文件，只要通过URL的形式来提供定位即可。
例如索引可以放在本地文件系统中，通过file: URL来进行定位；也可以存放在一个远程HTTP(s)服务器上。
真实资源的位置——例如JAR文件——通过索引文件中的URL来指定，因此它们可以是在本地或远程服务器上。
在使用远程索引和远程资源的情况下，会使用本地缓存来避免重复下载并支持离线构建。

固定索引仓库（Fixed Index repository）无法通过bnd或Bndtools来修改。

此类仓库支持如下属性：

|**名称**|**描述**|**是否必须（Required）？**|
|---|---|---|
|name|仓库名称|No|
|locations|以逗号分隔的索引URL列表。注意: 如果其中包括多项，请使用单引号将值引起来。|No。默认值: 空|
|cache|远程资源的本地缓存目录|No。 默认值: ${user.home}/.bnd/cache/|

无需指定索引格式——只要其格式是被支持的格式之一，则会由插件自动检测。索引文件可以选择使用gzip进行压缩。

<span id="12"/>

### 1.2 本地索引仓库（Local Indexed Repository）

此种仓库在一个本地文件夹中存放Bundle。仓库可以通过bnd/Bndtools进行编辑，索引文件会在Bundle部署到其中时自动重新生成。

此类仓库支持如下属性：

|**名称**|**描述**|**是否必须（Required）？**|
|---|---|---|
|name|仓库名称|No|
|local|本地文件夹|Yes|
|type|要生成的索引类型（格式）参考下面的Note1|No。 默认值: R5|
|pretty|是否以格式美化方式生成索引。参考下面的Note2|No。Default: false|
|readonly|仓库是否只读，例如禁止通过Bndtools进行编辑|No。 默认值: false|
|mode|解析（Resolution）模式: build, runtime或any。控制此仓库使用的解析模式。|No。默认值: any|
|locations|以逗号分隔的索引URL列表。注意: 如果其中包括多项，请使用单引号将值引起来。|No。默认值: empty|
|cache|远程资源的本地缓存目录|No。 默认值: ${local}/.cache|

Note 1: 产生的索引默认使用R5格式。如果需要其它类型的格式，则请指出格式名列表，其中以"|" (管道)字符分隔。例如要同时生成R5和OBR格式的索引，则使用：**type=R5|OBR.**

Note 2: 默认产生的R5格式的索引不会带有空格和缩进字符，而且会使用gzip压缩，默认的索引文件名为**index.xml.gz**。启用格式美化会进行缩进，并且将内容解压缩到**index.xml**文件。此设置对OBR格式的索引没有影响，因为它们本身就是缩进并且未压缩的格式，并且其名称为**repository.xml**。

<span id="2"/>

## 2 文件仓库

这种类型的仓库基于简单的文件系统目录结构。它可以通过Bndtools进行编辑。注意：它不支持索引，因此此种类型的仓库无法用于解析运行需求（Run Requirements）。

此类仓库支持如下属性：

|**名称**|**描述**|**是否必须（Required）？**|
|---|---|---|
|name|仓库名称|No。|
|location|本地文件夹|Yes。|
|readonly|仓库是否只读，例如禁止通过Bndtools进行编辑|No。 默认值: false|

<span id="3"/>

## 3 Aether (Maven)仓库

这种类型的仓库使用 [Eclipse Aether](http://www.eclipse.org/aether/) 来与Maven风格的仓库配合使用。
包括公共的Maven中央仓库，也包括由仓库管理产品托管的仓库，例如由Sonatype公司提供的[Nexus](http://www.sonatype.org/nexus)或JFrog公司提供的[Artifactory](http://www.jfrog.com/home/v_artifactory_opensource_overview)。

这种仓库也可以支持由远程仓库管理器提供的索引。这需要在对应的管理器上启用并且正确配置了索引功能。例如：Nexus OBR插件即可提供此功能。在启用了索引功能的情况下，此种仓库类型可以用于解析运行需求（Run Requirements）。

此类仓库支持如下属性：

|**名称**|**描述**|**是否必须（Required）？**|
|---|---|---|
|name|仓库名称|No。默认值: "AetherRepository"|
|url|远程仓库的主URL|Yes。|
|username|仓库管理器提供的用户名|No，但如果某些特定的操作（例如部署操作）需要管理器验证授权则可能会失败。|
|password|仓库管理器提供的密码|No.|
|indexUrl|由仓库管理器产生的索引文件的URL。|No。默认值：主URL+ -obr/.meta/obr.xml.|
|cache|远程资源的本地缓存文件夹|No。默认值： ${user.home}/.bnd/cache/|

<span id="31"/>

### 3.1使用限制

由Aether（以及Maven）提供的功能受到很大的限制。如果索引不可用则会有以下限制：

- 不能浏览：无法列出或浏览仓库的内容。在Bndtools的Repositories视图中查看仓库时，该仓库看起来会是空的。尽管仍然可以使用仓库中的组件，但是你必须知道精确的Group ID和Artifact ID，并且通过**\<groupId\>:\<artifactId>**格式来指定。
- 无法解析：仓库无法自动解析运行需求（Run Requirement）。

<span id="4"/>

## 4 Maven仓库（老式风格）

### 注意：在本节中介绍的仓库类型已经被废弃了。请使用Aether 仓库类型来替代之。下面的文档内容仅供参考。

<span id="41"/>

### 4.1 Maven本地仓库

此种类型的仓库用于访问Maven本地仓库中的Bundle，一般位于$HOME/.m2/repository。注意此插件直接访问Maven的仓库，并未通过Mavne来进行构建。

|**名称**|**描述**|**是否必须（Required）？**|
|---|---|---|
|name|仓库名称|No。|
|root|Maven仓库所在的位置|No。默认值：$HOME/.m2/repository|

注意如果你使用了支持Maven的Bundle插件，则也可以使用OBR仓库类型，因为Bundle插件会在**maven install**命令执行时生成一个OBR索引文件。例如：

    aQute.bnd.deployer.repository.FixedIndexedRepo;\
    locations='file:${user.home}/.m2/repository/repository.xml';\
    name='Maven Repo'



<span id="42"/>

### 4.2 Mavne远程仓库

此种类型的仓库用于访问Maven远程仓库中的Bundle，包括Maven中央仓库或其它Nexus仓库。注意：此种仓库类型是不可浏览的；如果你尝试通过Bndtool的Repositories视图来查看仓库的内容，该仓库看起来会是空的。
不过如果你知道Group ID和Artifact ID，仍然可以通过**-buildpath**或**-runbundles**来引用仓库中的JAR包。
例如如果要引用group ID为org.osgi，同时artifact ID为osgi_R4_core的JAR包，可以使用下面的语法：

    -buildpath: org.osgi+osgi_R4_core

|**名称**|**描述**|**是否必须（Required）？**|
|---|---|---|
|repositories|以逗号分隔的Maven仓库URL列表。注意：如果其中包含多项则使用单引号将此参数值引起来。|No。默认值：空|