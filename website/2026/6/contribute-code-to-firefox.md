# 为 Firefox 贡献代码

## 前言

> [Firefox](https://www.firefox.com/en-US/)（火狐浏览器）是来自非营利组织 Mozilla 的快速、可靠且私密的网络浏览器。
>
> 所有 Mozilla 软件都是开源（open source）的自由软件（free software）。这意味着它不仅可以免费下载，而且您可以访问源代码，并可在一定限制下修改和重新分发软件。

本文介绍了将 bug 修复提交给上游的大致流程。

注：本文假设你掌握 Git 的基本使用，同时，本文以 Linux 用户为例，使用其他系统的用户可能需要对文中的命令稍作修改。

## 找到 Bug

- 要提交 bug 修复，首先得找到 bug。

你可以在 Mozilla 的 [Bugzilla](https://bugzilla.mozilla.org/describecomponents.cgi?product=Firefox) 上找到你想修复的 bug；你也可以在 Bugzilla 上[报告你发现的 bug](https://bugzilla.mozilla.org/enter_bug.cgi?)。同时，你也可以阅读 Mozilla 的报告 bug [指南](https://support.mozilla.org/zh-CN/kb/contributors-guide-writing-good-bug)。

- 找到 bug 后，便要定位问题。

你可以将代码 clone 到本地，运用 VS Code 等进行阅读。你也可以借助 GitHub 的搜索功能（GitHub 仓库：[https://github.com/mozilla-firefox/firefox](https://github.com/mozilla-firefox/firefox)）或 Mozilla 官方的 [Searchfox](https://searchfox.org/)。考虑到 Firefox 的仓库比较大，使用云端的功能通常比较方便。但在编写修复时，建议将 Firefox clone 到本地。

注：为了提交修复，你需要在 [Bugzilla](https://bugzilla.mozilla.org/) 上注册用户并启用双因素认证，接着再注册 [Phabricator](https://phabricator.services.mozilla.com/)（需使用真名）。

## 安装依赖

为了编译 Firefox，你需要在你的电脑上安装 Git 和 Python 3.9 或更高版本。

### Windows 依赖项

在 Windows 上，你还需要额外下载一个 [MozillaBuild Package](https://ftp.mozilla.org/pub/mozilla/libraries/win32/MozillaBuildSetup-Latest.exe)，并安装在 `C:\mozilla-build\`。

## 初始化源代码

### 官方方法

官方推荐的流程是使用 [bootstrap.py](https://raw.githubusercontent.com/mozilla-firefox/firefox/refs/heads/main/python/mozboot/bin/bootstrap.py)。

```bash
curl -LO https://raw.githubusercontent.com/mozilla-firefox/firefox/refs/heads/main/python/mozboot/bin/bootstrap.py

python3 bootstrap.py
```

该脚本将会检查 Python 版本以及选择使用哪种版本控制系统，默认情况下使用 Git 拉取代码（但也可以指定使用 Mercurial）。脚本将交互式引导你 clone 代码。如果你使用 Windows，脚本还会设置 Windows Defender，将项目目录添加到排除列表。

随后，该脚本将会使用项目目录里的 mach（Mozilla 的命令行工具框架）进行环境配置，并让你选择构建类型。

### 手动初始化

手动初始化的优点是你可以使用浅克隆来避免下载 Firefox 的接近一百万次 commit 历史（目前 97 万 7 千次）。

```bash
git clone --depth 1 git@github.com:mozilla-firefox/firefox.git
```

随后，在项目中使用 `./mach bootstrap` 进行初始化。

~~不过，当我试图抛弃 Python 虚拟环境，使用 Arch Linux 上的系统库进行环境配置与编译时，我狠狠地坠机了——我编译了半天，最终还是失败了，mach 的编译脚本对我来说还是有些过于复杂了。~~

#### 镜像加速

mach 会使用 Python 的虚拟环境进行配置，但从官方源下载依赖不够快，因此你可以手动指定 pip 的索引源。

```bash
# 创建虚拟环境
python -m venv .venv

# 启用虚拟环境
source .venv/bin/activate

# 设置清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

## 编译

使用 `./mach build && ./mach run` 进行编译和运行。

~~可惜我坠机了没编译成功。~~

### [Artifact Builds](https://firefox-source-docs.mozilla.org/contributing/build/artifact_builds.html)

> 桌面版和安卓版的 Firefox 支持一种称为 artifact mode 的快速构建模式。生成的构建称为 artifact builds。Artifact mode 下载预构建的 C++ 组件，而不是在本地构建。用带宽换取时间。

#### 前置条件

> 使用 Git 时需要使用 [git-cinnabar](https://github.com/glandium/git-cinnabar) 帮助克隆 mozilla-central。

#### 使用方法

将以下配置加到 [mozconfig](https://firefox-source-docs.mozilla.org/setup/configuring_build_options.html) 文件中（可以是 `./mozconfig` 或 `./.mozconfig`）：

```bash
# 自动下载并使用编译好的 C++ 组件
ac_add_options --enable-artifact-builds

# 设置构件下载目录
mk_add_options MOZ_OBJDIR=./objdir-frontend
```

如果你需要 debug 版本的构件（目前支持 Windows、Linux 和 macOS），请使用：

```bash
# 使用 debug 版构件
ac_add_options --enable-debug

# 自动下载并使用编译好的 C++ 组件
ac_add_options --enable-artifact-builds

# 下载 debug 信息，以便堆栈跟踪引用文件和代码行号，而不是库和十六进制地址
ac_add_options --enable-artifact-build-symbols

# 设置构件下载目录
mk_add_options MOZ_OBJDIR=./objdir-frontend-debug-artifact
```

~~我也不知道为啥我用不了，或许是我坠机了吧~~

## 在 Matrix 上与他人沟通

你可以在 [Matrix](https://chat.mozilla.org/) 的介绍频道上和别人打招呼，或进行讨论、求助。

## 编写补丁

在本地编写好修复后，用 `git commit` 提交更改。

提交信息应如下所示：

```text
Bug xxxx - Short description of your change. r?reviewer

Optionally, a longer description of the change.
```

> **请确保包含错误编号和至少一位审阅者（或审阅者小组），格式如下。**
>
> 例如 `"Bug 123456 - Null-check presentation shell so we don't crash when a button removes itself during its own onclick handler. r=person"`
>
> 要找到[审阅者或审阅组](https://firefox-source-docs.mozilla.org/contributing/reviews.html)，最简单的方法是在相关文件上运行 `git log 你更改的文件`，然后查看谁通常负责审阅实际变更（即不包括重新格式化、变量重命名等）。
>
> 要在仓库中可视化您的补丁，请运行 `git show`。
>

如果你使用了浅克隆，建议去 GitHub 上找到相关文件的 log。

别忘了检测代码风格违规项。

```bash
./mach lint "你更改的文件或文件夹"
```

## 提交补丁

> 要提交补丁以供审核，我们使用一个名为 [moz-phab](https://pypi.org/project/MozPhab/) 的工具。

你可以用 `./mach install-moz-phab` 或 `pip install MozPhab` 安装 moz-phab。

> 一旦您想提交您的补丁（确保您使用正确的提交信息），请运行 `moz-phab`。

跟着 moz-phab 的指引即可。

## 更新补丁

> 审阅者很少会接受补丁的第一个版本。此外，由于代码审查机器人可能会提出一些改进建议，你的补丁可能需要进行修改。

修改后，使用 `git commit --amend` 来提交更改，并再次运行 `moz-phab` 提交补丁。

> 不要使用 `git commit --amend -m`。
>
> Phabricator 在创建修订时编辑提交信息，添加一行特殊行 `Differential Revision: <url>` 来跟踪修订。
>
> 使用 `--amend -m` 时，该行会丢失，导致重新提交时创建新的修订版本，这并非预期结果。

### 注意事项

有时，上游会有新的变更，或撤销某些变更，这会导致 CI 编译不了。

使用以下命令拉取更改。

```bash
git remote update
git rebase origin/main
```

遇到冲突时请解决合并冲突，并保证你的提交信息不变。

别忘了检测代码风格违规项。

```bash
./mach lint "你更改的文件或文件夹"
```

## 奇妙的小方法

如果你的修改仅涉及纯前端/脚本资源等文件，又不想编译一遍 Firefox，你可以通过修改已安装的 Firefox 文件来验证功能。

以 Arch Linux 下的 Firefox 为例，打包后的文件存储在 `/usr/lib/firefox/browser/omni.ja` 和 `/usr/lib/firefox/omni.ja` 里。

通过 unzip 可以查看和解压 ja 文件：

```bash
$ unzip -l /usr/lib/firefox/browser/omni.ja | head -n 10

Archive:  /usr/lib/firefox/browser/omni.ja
warning [/usr/lib/firefox/browser/omni.ja]:  50354572 extra bytes at beginning or within zipfile
  (attempting to process anyway)
error [/usr/lib/firefox/browser/omni.ja]:  reported length of central directory is
  -50354572 bytes too long (Atari STZip zipfile?  J.H.Holm ZIPSPLIT 1.1
  zipfile?).  Compensating...
  Length      Date    Time    Name
---------  ---------- -----   ----
    96737  2010-01-01 00:00   defaults/preferences/firefox.js
     2301  2010-01-01 00:00   defaults/preferences/firefox-branding.js
     3290  2010-01-01 00:00   defaults/preferences/debugger.js
       72  2010-01-01 00:00   chrome.manifest
     1583  2010-01-01 00:00   chrome/chrome.manifest
    11815  2010-01-01 00:00   components/components.manifest
    60799  2010-01-01 00:00   modules/BrowserGlue.sys.mjs
```

```bash
unzip -d "解压到的目录" /usr/lib/firefox/browser/omni.ja
```

应用完修改后，使用 zip 打包：

```bash
cd "解压到的目录"

zip -r9 "生成文件的位置" *
```

备份、替换原本的文件即可。

最后运行 `firefox -purgecaches` 删除缓存并打开浏览器，随后进行功能验证即可。

（据传 `rm -rf ~/.cache/mozilla/firefox/*/startupCache/` 也可以清除缓存）

## 最后

> 一旦更改被接受并且您已修复审阅者发现的任何剩余问题，审阅者应合并补丁。
>
> 如果补丁几天后仍未合并到 “autoland”（集成分支），请联系审阅者或在 #introduction 频道联系 @Aryx 或 @Sylvestre。

## 参考

- [Building Firefox On Linux](https://firefox-source-docs.mozilla.org/setup/linux_build.html)
- [Configuring Build Options](https://firefox-source-docs.mozilla.org/setup/configuring_build_options.html)
- [Contributing to Mozilla projects](https://firefox-source-docs.mozilla.org/contributing/contributing_to_mozilla.html)
- [Firefox Contributors' Quick Reference](https://firefox-source-docs.mozilla.org/contributing/contribution_quickref.html)
- [Getting reviews](https://firefox-source-docs.mozilla.org/contributing/reviews.html)
- [Mozilla Phabricator User Guide](https://moz-conduit.readthedocs.io/en/latest/phabricator-user.html)
- [Understanding Artifact Builds](https://firefox-source-docs.mozilla.org/contributing/build/artifact_builds.html)
- [Working with stack of patches Quick Reference](https://firefox-source-docs.mozilla.org/contributing/stack_quickref.html)
