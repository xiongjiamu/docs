# 107 - 社区开发基础设施

社区开发基础设施包含一系列社区研发、测试的支撑系统，目前正在快速发展中。

基础设施 SIG：https://openanolis.cn/sig/SIG-Infra

## 社区官网

社区官网是社区信息交流的门户，是完全自研建立的，包含首页、SIG、活动、动态、博客、下载、支持、后台管理等完备的信息互动交流以及管理模块；社区用户可以通过官网查看龙蜥的一系列公告、会议、活动、文档以及其它各类基础设施等。

官网链接：https://openanolis.cn/

## 代码托管平台

龙蜥社区采用国内知名代码托管平台Gitee的企业版本进行代码管理，社区为了更好地服务社区开发者制定了代码目录管理规范，以下目录下对外界开源代码库，任何免密人员都是可以看到并进行clone、fork、提交PR等基本的操作。

平台链接：https://gitee.com/openanolis

代码托管目录规范：
1. source tree：[anolis](https://gitee.com/anolis)
	- 存放 Anolis OS 各类项目源代码。
2. rpm tree：[src-anolis-os](https://gitee.com/src-anolis-os)
	- Anolis OS 发行版默认 RPM 打包代码（通常应当包含一个 spec 文件）。
3. module：https://gitee.com/src-anolis-module
	- Anolis OS module 包描述文件的存放组织。
4. Plus 包 rpm tree：[src-anolis-plus](https://gitee.com/src-anolis-plus)
	- 存放 rpm tree plus 包。
5. SIG 包 rpm tree：[src-anolis-sig](https://gitee.com/src-anolis-sig)
	- Anolis OS SIG RPM 打包代码（通常应当包含一个 spec 文件）。
6. DDE 包：[src-anolis-dde](https://gitee.com/src-anolis-dde)
	- Anolis OS rpm tree repo for anolis dde sig。
7. Epao 包：[src-anolis-epao](https://gitee.com/src-anolis-epao)
	- Extra Packages for Anolis OS -- RPM Tree。
8. Anolis Education: [anolis-education](https://gitee.com/anolis-education)
	- 龙蜥高校合作，产教融合，服务于基础软硬件人才培养，开源知识和文化普及。

## Issue 管理平台

龙蜥社区有完善的需求缺陷管理系统，社区开发者可以通过它管理需求、缺陷；一般产品级的需求或缺陷使用 bugzilla 进行管理，项目级别的需求或缺陷使用 gitee issue 进行管理。

Bugzilla链接：https://bugzilla.openanolis.cn/
Gitee链接：https://gitee.com/openanolis

## 邮件列表系统

邮件列表系统是社区开发者基于邮件进行交流的主要工具。

系统链接：https://lists.openanolis.cn/postorius/

## 开发者服务平台

龙蜥社区开发者服务平台致力于为社区的所有参与者及用户提供一个开放、高效的研发服务平台，主要包含构建服务、测试服务、资源服务、镜像服务、安全服务等等。

### 1. 龙蜥实验室

龙蜥实验室为社区用户提供了一个预装龙蜥OS的在线机器资源服务；用户可以通过web页面及机器人等形式自动创建和管理机器资源，
为社区用户体验 Anolis OS 以及使用 Anolis OS 进行开发测试提供了极大的便利性，满足社区用户对于机器资源的各类需求。
目前龙蜥实验室正在开发

系统链接：https://lab.openanolis.cn/#/apply/home
使用指南：https://www.yuque.com/anolis-docs/community/peng85


### 2. T-One（Testing in One）

T-One（Testing in One）是一站式的自动化质量协作平台；打通了测试计划、测试准备、测试执行、测试分析、测试报告、覆盖率检测、智能Bisect，环境服务等流程的闭环，为社区研发提供一站式质量服务。

平台采用分布式业务架构设计，分master/slave端，对外统一叫Testfarm: 
1. master端：对外称为Testfarm，负责测试监控触发，整体测试的数据对外展示。
2. slave端：对外称为T-One, 负责测试执行，支持社区、外部用户测试；外部用户也可以独立部署。


T-One 链接：https://tone.openanolis.cn/
Testfarm 链接：https://testfarm.openanolis.cn/
用户文档：https://tone.openanolis.cn/help_doc/1
T-One SIG：https://openanolis.cn/sig/t-one

欢迎加入 T-One SIG 进行交流。

### 3. ABS（Anolis Build Service）

ABS（Anolis Build Service）是龙蜥社区的构建服务，为社区开发者、社区版本发行提供构建支持，包括软件包、镜像、以及社区 CICD 流程的构建支撑。

系统链接：https://abs.openanolis.cn/all_project