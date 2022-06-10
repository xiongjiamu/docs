# 305 - 龙蜥社区软件包集成流程

> 本文档阅读对象适用于龙蜥社区软件开发人员。阅读者可以参照该文档完成新增软件包的集成流程。

## 1. Anolis OS 软件包仓库结构
OpenAnolis 龙蜥社区的主要发行版产品，包括 Anolis OS 7, 8 以及 23，都有相似的 YUM 仓库结构，这有助于用户通过 YUM 等系统工具获取操作系统内更新时，拥有较为一致的体验。
龙蜥社区欢迎广大开发者积极贡献软件包到 Anolis OS 中，集成过程需要遵循该软件包仓库结构。仓库结构大致示意图如下：

![龙蜥软件包仓库](/docs/images/305-anolis-yum-repo.png)

从来源区分，软件包仓库包含系统包和 SIG 包，往下细分又有不同的子仓库。对于用户来说，每一个子仓库在系统中的表现不同：有的仓库预装在 Anolis OS 默认仓库列表中，有的没有预装；有的默认使能，执行 `yum install` 即可执行安装，有的默认不使能，需要添加 `--enablerepo` 参数；有的仓库里的软件包跟随 ISO 交付，有的不跟随。下面是一个简单的对照例子：

仓库名|SIG 仓库|阶段|Gitee 仓库|ISO 包含|仓库默认状态
--|--|--|--|--|--
[BaseOS](http://mirrors.openanolis.cn/anolis/8/BaseOS/)|否|系统包阶段|[src-anolis-os](https://gitee.com/src-anolis-os)|是|✅ 预装 ✅ 使能
[AppStream](http://mirrors.openanolis.cn/anolis/8/AppStream/)|否|系统包阶段|[src-anolis-os](https://gitee.com/src-anolis-os)|是|✅ 预装 ✅ 使能
[PowerTools](http://mirrors.openanolis.cn/anolis/8/PowerTools/)|否|系统包阶段|[src-anolis-os](https://gitee.com/src-anolis-os)|否|✅ 预装 ✅ 使能
[Extras](http://mirrors.openanolis.cn/anolis/8/Extras/)|否|系统包阶段|[src-anolis-os](https://gitee.com/src-anolis-os)|否|✅ 预装 ✅ 使能
[Plus](http://mirrors.openanolis.cn/anolis/8/Plus/)|是|包成熟阶段|[src-anolis-sig](https://gitee.com/src-anolis-sig)|是|✅ 预装 ❌ 使能
[HighAvaliability](http://mirrors.openanolis.cn/anolis/8/HighAvailability/)|是|包成熟阶段|[src-anolis-sig](https://gitee.com/src-anolis-sig)|否|✅ 预装 ❌ 使能
[DDE](http://mirrors.openanolis.cn/anolis/8/DDE/)|是|包成熟阶段|[src-anolis-dde](https://gitee.com/src-anolis-dde)|否|✅ 预装 ❌ 使能
[Experimental](http://mirrors.openanolis.cn/anolis/8/Experimental/)|是|包孵化阶段|[src-anolis-sig](https://gitee.com/src-anolis-sig)|否|❌ 预装 ❌ 使能
[EPAO](https://mirrors.openanolis.cn/epao/)|是|包孵化阶段|[src-anolis-sig](https://gitee.com/src-anolis-sig)|否|❌ 预装 ❌使能

## 2. 软件包集成的意义
软件包默认集成到 Anolis OS 这个操作系统中，更多地意味着再分发（re-distribution）的便利。

- **对于软件包的拥有者来说**，经过软件包集成流程可以明确自己的**软件包在 Anolis OS 中的构建依赖和运行依赖分别是什么**，这意味着该软件包融入了 Anolis OS 发行版生态中，该软件的能力会作为操作系统整体能力的一部分持续存在。因此在发行版的持续演进升级过程中，必须要考虑到对该软件包的兼容性承诺；同样地，该软件包在演进升级过程中，也需要考虑对操作系统的前向兼容性。
- **对于用户来说**，软件包集成到 Anolis OS 中，可以通过发行版提供的统一的操作界面（例如 YUM 源）来对软件进行统一管理，包括安装、更新、卸载等；甚至也有可能通过操作系统提供的统一配置接口（如 systemd 服务自启动）来以对用户透明的方式获得软件的服务。
- **对于 Anolis OS 来说**，集成了某个软件包，意味着发行版获得了一个此前不具备的软件能力，以补充操作系统的软件栈大图。同样，发行版对该软件包自动负有维护和管理的能力，因此软件包的加入、变更、退休都需要正式的流程。

## 3. 我是否应该集成我的软件包到 Anolis OS 中？

对于软件包的拥有者通用的一个指导原则是，是否集成软件包到 Anolis OS 中要看**自己是否需要 Anolis OS 提供的再分发的便利**，即：

- 是否希望 Anolis OS 帮助解决二进制构建和运行的依赖，同时保持一定程度的兼容性；
- 是否希望通过 Anolis OS 提供的各类工具（YUM， systemd service 等）管理和使用自己的软件包；
- 更进一步，是否希望自己的软件成为 Anolis OS 发行版软件栈的一部分。

**如果以上的回答全部为“否”，那么可能没有必要集成自己的软件包到 Anolis OS 中**，你可以有其他的选择，比如：

- **自己维护独立的第三方 repo**。这种方式会有更好的维护上的灵活性，但是可能会和 Anolis OS 有兼容性问题，比如安装的时候和现有 Anolis OS 里的软件冲突；
- **不提供二进制分发方式，只推荐用户从源码下载到 Anolis OS 后自己构建安装**。这种方式在兼容性上不会有大问题，但是操作起来非常复杂，而且不利于整体解决方案复制到其他场景。

反之，如果回答“是”，甚至如果回答中有“我不确定”的答案，我们都建议考虑将自己软件包集成到 Anolis OS 中，当然由于需要遵循集成流程，这个过程可能会比较复杂，但是基本上是一劳永逸的付出。

## 4. 我应该分发我的软件包到哪个 YUM 仓库中？

如前文所述， Anolis OS 软件包仓库包含系统包和 SIG 包两大类。从 SIG 组产出的软件包，都应当首先集成为 SIG 包。
- SIG 包
  参照其他成熟开源社区，集成为 SIG 包后会经历孵化期和成熟期，龙蜥社区技术委员会和发布 SIG (sig-distro) 会从技术侧进行软件包集成和孵化指导。对于操作系统特别重要的软件包，还可以进一步孵化为系统包。
- 系统包
  通常来源于操作系统主动选型，包含基础系统不可分割的组件（BaseOS）、应用生态的重要组件（AppStream）、devops 依赖工具（PowerTools）及额外仓库文件（extras）等。对于可以孵化为系统包的 SIG 包，社区也提供了指导流程，满足条件的 SIG 包可以根据流程升级为系统包。
作为集成的入口，针对 SIG 包，社区提供了多种不同的集成路径和孵化路径，大致如下：

![集成步骤](/docs/images/305-intergration-ways.png)

从流程图可以看到，基于维护策略是否跟随系统更新走，会集成到不同的仓库。此处的“系统更新”特指发行版小版本的更新，如 Anolis OS 8.2 到 8.4 版本更新。

- 如果一个小版本更新了，为了专门适配此次升级，该软件包可能会迎来一次专门的升级和更新，那么应当集成到每一个小版本的 SIG 仓库中，例如 DDE(独立的 SIG repo), Experimental(非独立的 SIG repo)；
- 如果小版本更新后，该软件包并不需要专门升级，只需要定期独立维护，则可以放到 EPAO 仓库中。
集成到每一个小版本的 SIG 仓库也有不同的形式，主要分为独立的 SIG 仓库和非独立的 SIG 仓库。Anolis OS 提供了两个非独立的 SIG 仓库，Experimental 和 Plus. 其中 Experimental 仓库可以接收相对更不稳定的 SIG 包，Plus 仓库只能接收生产级可用的稳定包。

上述提到的各种集成方式详细对比如下：

<table>
    <tr>
        <td></td>
        <td><b>YUM repo</b></td>
        <td><b>EPAO</b></td>
        <td><b>非独立 SIG repo<br>Experimental & Plus</b></td>
        <td><b>独立 SIG repo</b></td>
   </tr>
    <tr>
        <td rowspan="3">对维护者</td>
  		<td>发行版大版本更新后需要采取的措施</td>
      	<td>跟随更新</td>
        <td>跟随更新</td>
        <td>跟随更新</td>
    </tr>
    <tr>
  		<td>发行版小版本更新后需要采取的措施</td>
      	<td>不主动跟随，不重新构建，<br>除非依赖关系发生冲突</td>
        <td>跟随更新</td>
        <td>跟随更新</td>
    </tr>
    <tr>
  		<td>一次推送的软件包数量</td>
      	<td>无限制</td>
        <td>SRPM <= 30 个且<br>单架构下 RPM <= 50 个</td>
        <td>SRPM > 30 个且<br>单架构下 RPM > 50 个</td>
    </tr>
    <tr>
        <td rowspan="3">对用户</td>
  		<td>ISO 中是否包含该仓库</td>
      	<td>否</td>
        <td>Experimental 不包含<br>Plus 包含</td>
        <td>不包含</td>
    </tr>
    <tr>
  		<td>系统中是否预装该仓库的 repo 文件</td>
      	<td>否</td>
        <td>Experimental 否<br>Plus 是</td>
        <td>孵化期 repo 否<br>成熟期 repo 是</td>
    </tr>
    <tr>
  		<td>用户是否需要手动开启 repo</td>
      	<td>否，添加 repo 文件自动开启</td>
        <td>是</td>
        <td>是</td>
    </tr>
</table>

## 5. 我如何提交软件包集成申请？

软件包集成申请都需要通过 SIG 来运作，如果想要引入的软件包未归属到已有 SIG 中，请浏览[社区 SIG 页面](https://openanolis.cn/sig)找到对应的 SIG 组加入；如果找不到对应的 SIG，也欢迎先联系社区（钉钉群“龙蜥OpenAnolis社区交流群”，群号 **33311793**）了解如何找到或创建一个 SIG。
当前软件包集成申请是通过社区的软件包集成项目 ([ospkg-list](https://gitee.com/anolis/ospkg-list)) 提交集成意向申请。当前我们使用 Issue 来跟踪和审核申请，后续社区可能会改为 Pull Request 模式提升审核的自动化能力。注意如果需要从 repo 中删除一个软件包，也可以使用该流程。

- 软件包新增申请模板

```
1. 软件包名：
2. 软件包分类（可以大致评估一下软件包属于系统中哪种分类，如 Development/Tools）：
3. 软件包功能描述：
4. 我当前所属的 SIG 组：
5. 软件包是否在现有仓库中存在：(如果填写为是，则不可以集成）
6. 是否要额外引入一个或多个依赖包：否/是，清单如下：
7. 是否已经制作完成 SPEC 文件：
8. 是否有需要构建的源码包：
9. 是否有明确的更新策略，是否需要跟随系统小版本定期升级：
10. 是否特别重要的软件包，申请直接跳过孵化期转系统包（一般情况请不要选择此项）：
11. 拟推送的 YUM repo 名称: Experimental, Plus, EPAO, 独立 repo
12. 其他可以帮助软件包集成的补充说明：
```

- 软件包退休申请模板

```
1. 软件包名：
2. 当前所属 RPM Tree 仓库路径：
3. 申请软件包退休的原因陈述：
```

在收到集成申请后，龙蜥发布 SIG 小组会进行评审互动，完成后将会创建对应的 RPM Tree 仓库，并告知后续构建软件包的方法，如果对于软件包集成过程不熟悉，龙蜥发布小组可以进行相关培训。培训的相关内容包含：
- 如何编写 RPM SPEC；
- 如何正确处理 RPM 依赖关系，以免破坏系统现有依赖关系；
- 如何保证系统兼容性，确保操作系统升级之后自己的软件包不受影响，或在升级软件包后不破坏系统本身的兼容性。
后续社区也会提供一系列教程和工具，提升整个过程的自动化程度，帮助 SIG 组的成员们在软件包集成和发布中顺利完成 landing。

## 6. 我的软件包如何孵化为成熟包及系统包？

软件包随着成熟度增加，SIG 包可以孵化到成熟包及转为系统包。按照是否缺省安装，是否缺省使能等要素可以区分为层级 1 到层级 5，层级编号从小到大对产品/用户影响面依次递增。其中层级 1 属于包孵化阶段，层级 2 属于包成熟阶段，层级 3 到层级 5 会同时涉及到 Mirror 源和镜像的相应操作改变，属于系统包的不同阶段。

下表会介绍不同层级转入的条件。

层级|阶段|仓库名|集成收益
--|--|--|--
1|包孵化阶段|[Experimental](http://mirrors.openanolis.cn/anolis/8/Experimental/)<br>[EPAO](https://mirrors.openanolis.cn/epao/)<br>独立 SIG repo|❌ 包含在 anolis-repos<br>❌ 安装引导可配<br>❌ 安装引导默认勾选<br>❌ （仅系统服务）开机自动启动
2|包成熟阶段|独立 SIG repo|✅ 包含在 anolis-repos<br>❌ YUM 仓库使能<br>❌ 包含在 ISO<br>❌ 安装引导可配<br>❌ 安装引导默认勾选<br>❌ （仅系统服务）开机自动启动
3|系统包阶段|[Plus](http://mirrors.openanolis.cn/anolis/8/Plus/)|✅ 包含在 anolis-repos<br>❌ YUM 仓库使能<br>✅ 包含在 ISO<br>❌ 安装引导可配<br>❌ 安装引导默认勾选<br>❌ （仅系统服务）开机自动启动
4|系统包阶段|[BaseOS](http://mirrors.openanolis.cn/anolis/8/BaseOS/)<br>[AppStream](http://mirrors.openanolis.cn/anolis/8/AppStream/)|✅ 包含在 anolis-repos<br>✅ YUM 仓库使能<br>✅ 安装引导可配<br>✅ 安装引导默认勾选<br>❌ （仅系统服务）开机自动启动
5|系统包阶段|[BaseOS](http://mirrors.openanolis.cn/anolis/8/BaseOS/)<br>[AppStream](http://mirrors.openanolis.cn/anolis/8/AppStream/)|✅ 包含在 anolis-repos<br>✅ YUM 仓库使能<br>✅ 安装引导可配<br>✅ 安装引导可配<br>✅ 安装引导默认勾选<br>✅ （仅系统服务）开机自动启动


在层级升级转入的过程中，需要提供如下材料：
- 提供典型场景测试自证明数据；
- 提供全面覆盖功能自证明测试；
- 提供兼容性相关测试自证明文档；

其中层级 1 ～ 3 在准入过程中需要通过产品发布 SIG 需求评审 ( 包含研发、测试负责人 ) ；层级 4 ～ 5 除了产品发布 SIG 需求评审，还需要通过 TC 评审。

系统包分阶段集成和层级提升和降低均通过社区 [ospkg-list](https://gitee.com/anolis/ospkg-list) 进行录入和跟踪。当前我们使用 Issue 来跟踪和审核申请，后续社区可能会改为 Pull Request 模式提升审核的自动化能力。

- 软件包层级提升（成熟度增加）申请模版：

```
1. 软件包名
2. 软件包产品线（Anolis OS 7 , Anolis OS 8 或者 Anolis OS 23）
3. 软件包当前层级
4. 软件包期望提升到的层级
5. 软件包层级准入要求对应的信息
```

- 软件包层级降低（成熟度降低）申请模版：

```
1. 软件包名
2. 软件包产品线（Anolis OS 7 , Anolis OS 8 或者 Anolis OS 23）
3. 软件包当前层级
4. 软件包期望降低到的层级
5. 软件包层级层级降低的原因
```
