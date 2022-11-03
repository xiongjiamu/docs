# 305 SPEC 模版和 checklist

## 1 背景
本文档规定龙蜥社区 RPM Tree 组织规范和 SPEC File 写作规范。
适用范围：

1. Anolis OS 自研的软件包；
1. Anolis 23 及以后的发行版所有的软件包；

总体原则：

1. 简洁、可阅读、易维护
1. 统一龙蜥标签

**修订记录**

| **时间** | **版本** | **作者** | **备注** |
| --- | --- | --- | --- |
| 2022.2.10 | v1.0 | [@林生](https://gitee.com/forrest_ly) | 初始版本 |
| 2022.3.31 | v1.1 | [@伊和](https://gitee.com/yueeranna) | 添加规范 |
| 2022.4.20 | v1.2 | [@伊和](https://gitee.com/yueeranna) | 添加epoch版本号说明 |
| 2022.8.3 | v1.3 | [@橘悦](https://gitee.com/happy_orange)| 新增模版和 checklist |

## 2 SPEC File 写作规范
### 2.1 spec 基础模版

spec 基础模版，可以使用 rpmdev-newspec 命令生成。
```
%define anolis_release 1
#Global macro/variable 定义

Name:	     package                    
Version:     1.0.0                      
Release:     %{anolis_release}{?dist}   
Summary:     Library providing xxx     
License:     LBPLv2+ and MIT          
URL:         https://github.com/package 
Source0:     https://github.com/package/archive/%{name}-%{version}.tar.gz
Source1:     temple.conf

Patch0:      bugfix-xxx-yyy.patch

BuildRequires:  cmake
BuildRequires:  gcc
BuildRequires:  gcc-c++

Requires:       glibc

%description
Library providing xxx and xxx.

%package devel
Summary:        Development files for %{name}
Requires:       %{name} = %{version}-%{release}

%description devel
The %{name}-devel package contains development files for %{name}.

%package -n python3-%{name}
%{?python_provide:%python_provide python3-%{name}}
Summary:        Python 3 bindings for the %{name} library.
BuildRequires:  python3-devel python3-setuptools
Requires:       %{name} = %{version}-%{release}

%description -n python3-%{name}
python 3 bindings for the %{name} library.

%package doc
Summary:        Documentation files for %{name}
Requires:       %{name} = %{version}-%{release}
BuildArch:      noarch

%description    doc
The %{name}-doc package contains documentation files for %{name}.


%prep
%autosetup -n %{name}-%{version} -p1


%build
%configure
%make_build


%install
%make_install
# 下面两行按需添加，当前仅为样例
mkdir -p %{buildroot}/%{prefix}/%{name}
install -m 0644 -p %{SOURCE1} %{buildroot}/%{prefix}/%{name}

%check
%make test 

%files
%license COPYTRING
%{_bindir}/%{name}
%{_libdir}/%{name}.so.*

%files devel
%doc example
%{_libdir}/%{name}.so
%{_libdir}/pkgconfig/%{name}.pc
%{_includedir}/%{name}/

%files -n  python3-%{name}
%{python3_sitearch}/%{name}/

%files doc
%doc README.md AUTHORS ChangLog NEWS TODO


%changelog
- Wed Aug 03 2022 happy_orange <songnannan@linux.alibaba.com> - 1.0.0-1
- Init package from upstream
```

### 2.2 纯 python 类 spec 模版

由于 python 类软件比较多，额外对外提供 python 类软件的 spec 模版。
```
%define anolis_release 1
%global pname package_real_name
%global debug_package %{nil}

Name:           python-%{pname}
Version:        0.1.0
Release:        %{anolis_release}%{?dist}
Summary:        xxxx package for Python

License:        MIT
URL:            https://sourceforge.net/projects/%{pname}
Source0:        https://sourceforge.net/%{pname}/code/%{pname}-%{version}.tar.gz

# python package only need to build in noarch.
BuildArch:      noarch

%description
xxxx package for Python

%package -n     python3-%{pname}
Summary:        YAML 1.2 loader/dumper package for Python
BuildRequires:  python3-devel
BuildRequires:  python3-setuptools
# For tests
BuildRequires:  python3-pytest
Requires:       python3-setuptools
%{?python_provide:%python_provide python3-%{pypi_name}}

%description -n python3-%{pname}
xxxx package for Python

%package -n     python3-%{pname}-doc
Summary:        doc files for python3-%{pname}
Requires:       python3-%{pname} = %{verison}-%{release}

%description -n python3-%{pname}-doc
doc files for python3-%{pname}


%prep
%autosetup -n %{pname}-code-%{commit} -p1
rm -rf %{pypi_name}.egg-info


%build
%py3_build


%install
%py3_install


%check
%pytest


%files -n python3-%{pname}
%license LICENSE
%{python3_sitelib}/%{pypi_name}
%{python3_sitelib}/%{pypi_name}-%{version}-*.pth
%{python3_sitelib}/%{pypi_name}-%{version}-*.egg-info

%files -n python3-%{pname}-doc
%doc README.rst

%changelog
* Mon Jul 25 2022 happy_orange <songnannan@linux.alibaba.com> - 0.1.0-1
- Init pacakge from upstream
```

### 2.3 rpm 增加 abi 和 api 信息

在 anolis 23 版本中额外支持在 rpm 中分别对 so 文件和 bin 文件增加 abi 和 api 信息，下面描述如何进行使用。

- 使用规则：
  - 在 %install 阶段使用 `%generate_compatibility_deps` 生成 abi/api 文件
  - abi/api 文件默认都存在放在 `%{abidir}` 中
  - abi 文件需要和 so 文件一起打包
  - api 文件需要和 bin 文件一起打包
  - 需要通过`%dir %{abidir}`的方式增加路径定义，否则会造成卸载残留
  - 需要根据文件列表和依赖关系共同决定`%dir %{abidir}` 的定义位置，放置在依赖树中最底层的包内，常见情况如下：
    - 如果只有单个 rpm ，则将路径定义增加到此 rpm 中；
    - 如果有多个 rpm，但是 abi/api 文件仅存在一个 rpm 内，则将路径定义增加到此 rpm 内；
    - 如果多个子包中都包含 abi/api 文件，并且这些子包之间是独立的，没有依赖关系，如果有主包，且都依赖主包，则将路径定义增加到主包内；
    - 如果多个子包中都包含 abi/api 文件，并且这些子包之间是独立的，没有依赖关系，如果没有主包，则新定一个主包，并将路径定义增加到主包内；
    - 如果多个子包中都包含 abi/api 文件，并且这些子包不是独立的，则需要找出这条依赖线中最底层的 rpm，比如 common 或者 libs，然后将路径定义增加到这个底层 rpm 中；
- 使用样例
  - 在 `%install` 阶段生成 abi/api 文件，注意需要将脚本调用放在最后
    ```
    %install
    %make_install docdir=%{_pkgdocdir}
    # remove unpackaged files from the buildroot
    rm -f $RPM_BUILD_ROOT%{_libdir}/*.la
    %generate_compatibility_deps
    ```
  - 打包 abi/api 文件
    ```
    %files
    %dir %{abidir}
    %{_libdir}/libasound.so.*
    %{abidir}/libasound.so.dump
    %{_bindir}/aserver
    %{abidir}/aserver-option.list
    ```
  - 验证 abi/api 的有效性
    ```
    # rpm -qpl alsa-lib-1.2.7.2-2.an23.x86_64.rpm | grep dump
    /usr/lib/compatibility/alsa-lib/libasound.so.dump
    /usr/lib/compatibility/alsa-lib/aserver-option.list
    ```
  - 验证 abi/api 的 provides
    ```
    # rpm -qp alsa-lib-1.2.7.2-2.an23.x86_64.rpm --provides | grep abi
    abi(alsa-lib) = 1.2.7.2

    # rpm -qp alsa-lib-1.2.7.2-2.an23.x86_64.rpm --provides | grep api
    api(alsa-lib) = 1.2.7.2
    ```



## 3 字段介绍和标准

| **spec header** | 增加 anolis_release 的定义，从 1 开始递增 |  |
| --- | --- | --- |
| **字段** | **定义** | **是否可选** |
| Name | 包的基本名称，应与 SPEC 文件名匹配。 | 必选 |
| Epoch | 超级版本号。<br/>1. 用于修正版本号，比如：0.20.0 版本 更改成 21.1 <br/>2. 初始化时， epoch 为 1，每次修正版本可以递增  <br/>3. 不可过度使用  <br/>4. 其他引用 %{version}-%{relase} 需要修改成：%{epoch}:%{version}-%{relase} |可选|
| Version | 软件的上游版本号。正式发布的 release/tag 版本号。 |必选|
| Release | 此版本软件的发布次数。<br/>1. 初始值为： %{anolis_release}%{?dist} <br/>2. 每次构建递增 anolis_release <br/>3. 构建软件新version 时 anolis_release 需要重置为 1 |必选|
| Summary | 一个简短的、单行的软件包摘要。 |必选|
| License | 被打包的软件的许可证，所有许可证的并集。 |必选|
| URL | 该软件的上游项目网站。 |必选|
| Source0 | 上游源代码压缩存档的路径。这应该指向存档的可访问且可靠的存储，例如，上游页面而不是打包程序的本地存储。<br/>如果需要，可以添加更多 SourceX 指令，每次递增编号，例如：Source1、Source2、Source3 等。 |必选|
| Patch0 | 对源码进行的修改以补丁的形式。<br/>1. 可以添加更多 PatchX 指令，每次递增编号，例如：Patch1、Patch2、Patch3 等。<br/>2. 自研补丁序号从100、1000、10000等编号开始。 |可选|
| BuildArch | 声明该软件的构建体系结构。<br/>1. koji 构建时默认为：x86_64  和 aarch64<br/>2. 本地构建时会自动继承构建它的机器的体系结构<br/>3. 如果不依赖体系结构，可以声明：BuildArch: noarch<br/>4. 如果仅涉及一个架构，则需要将对应的架构声明：BuildArch：x86_64 或 BuildArch：aarch64 |可选|
| ExcludeArch | 声明该软件不需要的架构体系。<br/> 1. 默认不需要<br/>  2. 指定不进行编译的架构，举例：ExcludeArch: x86_64  | 可选 | 
| ExclusiveArch | 声明该软件需要的架构体系。<br/> 1. 默认不需要<br/> 2. 指定进行编译的架构，举例：ExclusiveArch:  x86_64 | 可选 |
| BuildRequires | 声明该软件构建所需要的全部软件包列表。<br/>1. 有多个条目 BuildRequires每个条目在 SPEC 文件中各占一行<br/>2. 每个条目内不同软件使用空格隔开<br/>3. 直接声明依赖软件的 package name，不要包含： %{_isa}、/usr/bin/xx、pkg-config(xx)、/usr/lib64/xx.so 等 |必选|
| **spec body**      |              |
| **字段**      | **定义**                                                     | **是否可选** |
| %description  | RPM 中打包的软件的完整描述。该描述可以跨越多行并且可以分成段落。 | 必选         |
| %prep         | 用于准备软件包构建所需要的源码。<br/>1. 路径信息：将 tar.gz 从 ～/rpmbuild/SOURCES/ 目录下解压到 ～/rpmbuild/BUILD/ 下<br/>2. 建议使用 %autosetup -n %{name}-%{version}  -p1，可以自动按照补丁定义顺序将补丁以 -p1 形式打入<br/>3. 允许在此处拷贝 source 文件，例如：cp %{SOURCE1} ./<br/>4. 也允许去执行一些 shell 脚本 | 必选         |
| %build        | 将软件包的源码进行编译阶段。<br/>1. 路径信息：在 ～/rpmbuild/BUILD/%{name}-%{version}/ 下<br/>2. 执行构建可以根据源码语言去选择构建方式<br/>3. 在构建过程中是个 chroot 环境，不允许联网 download 等动作<br/>4. 不允许随意修改 flags 等 | 必选         |
| %install      | 将软件包编译生成的文件复制到安装目录，进行预安装动作，即模拟所有 package 安装后的环境。<br/>1. 路径信息： ～/rpmbuild/BUILDROOT/%{name}-%{version}/ <br/>2. 复制时，文件从  ～/rpmbuild/BUILD/%{name}-%{version}/  拷贝到  ～/rpmbuild/BUILDROOT/%{name}-%{version}/ <br/>3. 复制文件时，需要保留文件的时间戳，采用 cp -p 或 install -p <br/>4. 操作允许直接将 SourceX 文件拷贝到安装目录允许再该阶段直接生成新文件 | 必选         |
| %check        | 用于测试软件的命令或一系列命令。<br/>这通常包括诸如单元测试之类的东西，阶段能开则开，如果不能开启，声明不能开启的原因。 | 可选         |
| %files        | 定义每个 package 的文件列表，并生成对应的 .rpm。<br/>1. 文件布局必须布局遵循 [**FHS**](https://yuque.antfin-inc.com/bobac/pm1qpi/xz4m02) ，个别情况额外声明<br/>2. 正确设置文件权限：目录 0755，文件 0644 root root，除非出于安全考虑需要使用特定的用户或组<br/>3. 所有安装在  ～/rpmbuild/BUILDROOT/%{name}-%{version}/ 下的文件必须全部有唯一归属<br/>4. 不允许包含具有攻击性、歧义性、宗教性、色情性、受限制使用的文件等<br/>5. %doc 后面可以直接跟  ～/rpmbuild/BUILD/%{name}-%{version}/ 下的说明类文档：README、ChangeLog等<br/>6. %license 后面可以跟～/rpmbuild/BUILD/%{name}-%{version}/ 下的版权文件：copying、license 等 | 必选         |
| %changelog    | Version不同或Release构建之间的包发生的更改的记录。<br/>1. changlog 的格式正确，包括：日期、提交者个人信息、版本信息、描述信息等<br/>2. Message 简洁易懂，清晰明确 | 必选         |


## 4 check list
下面给出 spec 的 check list，可以自行检查。

| **分类** | **要求程度** | **详细内容** | **自检结果** |
| --- | --- | --- | --- |
| 宏定义 | must | 使用宏变量代替硬编码的目录名称 |  |
|  | must | 不要出现其他 OS 发行版的宏，比如：fedora、rhel、openEuler、opensuse等 |  |
|  | should | spec 中的宏统一使用同一种格式 |  |
|  | should  | 可以采用 %bcond_with 和 %bcond_without 来作为开关控制特性状态 |  |
|  | should | 宏定义时，%global 优先于 %define |  |
|  | should | 不使用 %if 0 作为判断条件 |  |
|  | should | 关于 python/perl/rust/golang/ruby/meson 等宏的使用，遵循各个 rpm-macros 包中的定义。 |  |
| 基本信息 | must | spec 的第一行增加 anolis_release 的定义：%define anolis_release 1  |  |
|  | must | Name 和 package 命名匹配  |  |
|  | must | Epoch 用来修正版本号，但不得过度使用  |  |
|  | must | Version 为正式发布的 release/tag 版本，如果采用 rc 版本，则 version 需要定义为 xx~rc1，post 版本需要定义为：xx-post1 |  |
|  | must | Release 使用 %{anolis_release}%{?dist}  |  |
|  | must | License 与实际的许可证相匹配，不存在缺失或者扩大  |  |
|  | should | 如果存在多个 license，建议给多个 license 加以注释  |  |
|  | must | Url 真实可用，属于上游源真实地址  |  |
|  | must | Source0 地址支持 curl 或者 wget 方式进行直接下载  |  |
|  | must | Source0 对应的源码包的 md5 值与开源社区的 md5 值相同 |  |
| 依赖阶段 | must | Buildrequires 中声明所有的第一层构建依赖，且全部使用 package name 进行声明，并要求构建依赖以 rpm 形式存在相同 OS 版本构建源里，不允许其他方式引入 |  |
|  | must | Requires 中声明所有的第一层运行依赖，且全部使用 package name 进行声明，并要求运行依赖以 rpm 形式存在相同 OS 版本的 yum 源里，不允许其他方式引入 （禁止安装后执行 pip install 等动作） |  |
|  | must | 声明 devel 或其他子包对于主包或者 libs 的依赖时，需要增加版本限制: Requires: %{name} = %{epoch}:%{verison}-%{release}  |  |
|  | should | 如果对依赖软件的版本有限制，则限制到软件包的 %{version} 即可，不需限制到 %{release} ，例：BuildRequires: glibc >= 2.32 ||
|  | should | 声明 buildrequires 和 requires 时，不要添加 %{?_isa}  |  |
|  | should | 使用 provides 对外提供功能时，和以前版本保持一致，以前有加版本号，则需要一直加下去，如果以前没有加版本号，则不要增加  |  |
|  | should | 使用 obsoletes 时，需要增加对应版本号  |  |
| 补丁文件 | must | Patch 和 Souce 采用序号区分自研和开源三方 |  |
|  | should | Patch 序号连续，并保持一种规范  |  |
|  | should | 每个 patch 或 source 内有详细的修改原因和原链接地址  |  |
| 架构阶段 | must | 至少在一种架构上构建成功，如果存在仅在部分架构上构建，可以使用 BuildArch（白名单）或者 ExcludeArch（黑名单） |  |
|  | must | Anolis OS 23 不支持 32 位，不支持multilib，请去除对其他架构的支持 |  |
|  | should | 合理使用 ifarch 和 ifnarch 的区别 |  |
| 子包划分 | must | 有定义 doc 子包，且定义正确（依赖、架构） |  |
|  | must | python 类软件仅生成 python3 子包，不再支持 python2 |  |
| 准备阶段 | should | 在 %prep 阶段采用 %autospec 进行自动解压并打补丁，保持 spec 简洁，如果存在与架构相关的补丁时，可以不采用 %autospec  |  |
| 构建阶段|must|在 %build 阶段不允许随意修改 flags，如果要修改，需要在 spec 中备注原因||
|  | must | 在 %build 阶段不允许随意禁用 PIE  |  |
|  | should | 在 %build 阶段尽量采用多线程进行编译，提高构建速度  |  |
|  | should | 在 %build 和 %install 阶段使用宏变量要保持一致，比如：%{buildroot} 或者 $RPM_BUILD_ROOT  |  |
| 安装阶段 | must | 在 %install 阶段复制文件时，需要保留文件的时间戳，比如 cp -p 或者 install -p  |  |
| 脚本阶段| must | 如果存在动态库，则必须在 %post 和 %postun 阶段调用 %ldconfig  |  ||
|  | must | 如果存在 service 文件时，需要在 %post、%postun 和 %preun 中对服务作出对应动作（比如：%post %systemd_post xxx.service） |  |
| 打包阶段 | must | %files 阶段文件系统布局遵循 FHS，个别特殊情况可以加以说明  |  |
|  | must | %files 阶段正确设置文件权限：目录 0755，文件 0644 root root，除非出于安全考虑需要使用特定的用户或组  |  |
|  | must | %files 里不允许有重复文件，一个文件仅能有一个归属  |  |
|  | must | %files 里不允许包含具有攻击性、歧义性、宗教性、色情性、受限制使用的文件等  |  |
|  | must | %doc 存放 readme、changelog、authors 等文件  |  |
|  | must | %license 存放 copying、license 等许可证文件  |  |
|  | must | 可以使用 %find_lang 宏来处理 locale 文件，进行自动打包相关文件。例：%find_lang %{name}  |  |
|  | must | 如果存在需要保存配置的 conf 文件，需要声明 %config(noreplace) xxx.conf  |  |
|  | must | 如果存在头文件，则需要放置在 devel 包内  |  |
|  | must | 如果有动态库，则包含后缀的库文件放在主包或者 libs 包，不包含后缀的库文件要放在 devel 中  |  |
|  | must | 如果有静态库，则静态库需要放置在 static 包内  |  |
|  | must | 公共的目录名称不可以通过 %dir 被重复定义，比如：%{_libdir}、%{_bindir}  |  |
| changlog 阶段 | must | changlog 的格式正确，包括：日期、提交者个人信息、版本信息、描述信息等  |  |
|  | should | changlog 的描述信息简洁易懂 |  |

