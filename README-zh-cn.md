# 简单管理Python版本：pyenv

由于python的各种优点，当前学习及使用python的人越来越多, 学习python有一个不容忽视的问题就是python的版本问题! 到现在为止，python的版本有很多，但是问题在于python2与python3的区别。python3的对一些模块进行了改变，导致了python2写的代码有的不被python3兼容，从而导致程序运行报错。因此，在学习和工作中使用python的时候，最好是安装一个pyenv管理器, 多安装几个python版本进行管理, 然后再针对不同项目安装各自项目的python虚拟环境, 相互隔离, 这样便于使用和管理。

[![Join the chat at https://gitter.im/yyuu/pyenv](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/yyuu/pyenv?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[![Build Status](https://travis-ci.org/pyenv/pyenv.svg?branch=master)](https://travis-ci.org/pyenv/pyenv)

pyenv使您可以轻松地在多个版本的Python之间切换。它简单，简单，遵循UNIX的一站式工具的传统，可以很好地完成一件事。

该项目是从 [rbenv](https://github.com/rbenv/rbenv) 和[ruby-build](https://github.com/rbenv/ruby-build)派生的, 并针对Python进行了修改.

![Terminal output example](#https://github.com/pyenv/pyenv/raw/master/terminal_output.png)

### pyenv是Python版本管理系统，它能做到...
- 管理Python的解释器
- 管理Python的版本
- 管理Python的虚拟环境
- 配置当前用户的python的版本;
- 配置当前shell的python版本;
- 配置某个项目（目录及子目录）的python版本;
- 配置多个虚拟环境.
- 让您基于每个用户**更改全局Python版本**。
- 提供针对**每个项目的Python版本的支持**。
- 允许您使用环境变量**覆盖Python版本**。
- 一次搜索**多个版本Python**中的命令。这可能有助于使用[tox](https://pypi.python.org/pypi/tox)跨Python版本进行测试。


### 与pythonbrew和pythonz相比，pyenv...

- **取决于Python本身**。 pyenv是由纯shell脚本制作的。 没有Python的引导问题。
- 需要加载到你的shell中。 相反，pyenv的shim方法通过向$ PATH添加目录来工作。
- 管理virtualenv。 当然，你可以自己创建[virtualenv](https://pypi.python.org/pypi/virtualenv)，或者[pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)来自动化该过程。


----


## 目录

+ **[工作原理](#工作原理)**
  - [了解路径Shims](#了解路径)
  - [了解 Shims](#了解-shims)
  - [选择Python版本](#选择python版本)
  - [查找Python安装](#查找python安装)
  - [管理虚拟环境](#管理虚拟环境)
+ **[安装](#安装)**
  - [安装前提条件](#安装前提条件)
  - [自动安装程序](#自动安装程序)
  - [基于GitHub Checkout安装](#基于github-checkout安装)
  - [进阶设定](#进阶设定)
  - [升级](#升级)
  + [卸载Python版本](#卸载Python版本)
+ **[命令参考](#命令参考)**
+ **[发展](#发展)**
  - [版本记录](#版本记录)
  - [执照](#执照)
----

## 工作原理

介绍pyenv工作原理之前必然要先简单介绍一下PATH环境变量的相关知识。
在较高的层次上，pyenv使用注入到您的shim可执行文件来拦截Python命令`PATH`，确定您的应用程序指定了哪个Python版本，并将您的命令传递给正确的Python安装。

### 了解路径

当您运行类似`python`或`pip`的命令时，您的操作系统会搜索目录列表以查找具有该名称的可执行文件。此目录列表位于一个名为的环境变量`PATH`中，列表中的每个目录用冒号分隔：

```shll
    /usr/local/bin:/usr/bin:/bin
```

`PATH`从左到右搜索目录，因此列表开头的目录中的匹配可执行文件优先于最后的另一个目录。在这个例子中，该`/usr/local/bin`目录将首先搜索，然后`/usr/bin`，然后`/bin`。
对于系统环境变量 PATH ，里面包含了一串由冒号分隔的路径，例如 `/usr/local/bin:/usr/bin:/bin`。每当在系统中执行一个命令时，例如 `python` 或 `pip`，操作系统就会在 `PATH` 的所有路径中从左至右依次寻找对应的命令。因为是依次寻找，因此排在左边的路径具有更高的优先级。在`PATH` 最前面插入一个 `$(pyenv root)/shims` 目录，`$(pyenv root)/shims`目录里包含名称为`python`以及`pip`等可执行脚本文件；当用户执行`python`或`pip`命令时，根据查找优先级，系统会优先执行shims目录中的同名脚本。pyenv 正是通过这些脚本，来灵活地切换至我们所需的Python版本。

### 了解 Shims

pyenv的工作原理是在原有的`PATH`变量搜索目录的前面 插入一个shim目录`PATH`：
`PATH`:

    $(pyenv root)/shims:/usr/local/bin:/usr/bin:/bin

通过一个名为rehashing的进程，pyenv在该目录中维持shims程序，以匹配每个已安装的Python-`python`，`pip`等版本的每个Python命令。
shims是轻量级可执行文件，只需将命令传递给pyenv即可。 所以安装了pyenv后，当你运行`pip`时，你的操作系统会执行以下操作：

- 在`PATH`中搜索名为`pip`的可执行文件
- 在`PATH`的开头找到名为`pip`的pyenv shim
- 运行名为`pip`的shim，然后将命令传递给pyenv
综合上述：pyenv使用注入到PATH中的SHIM程序可执行文件拦截Python命令，确定应用程序指定了哪个Python版本，并将命令传递给正确的Python安装。

而PATH变量在开机或者登录用户时会自动加载，加载顺序一般为`/ect/profile` -> `/ect/profile.d/*.sh` -> `/ect/profile.d/lang.sh` -> `/ect/sys config/i18n` -> `~/.bash_profile` -> `~/.bashrc` -> `~/ect/bashrc`
### 选择Python版本

执行填充程序时，pyenv通过从以下来源按以下顺序读取来确定要使用的Python版本：

1. 在`PYENV_VERSION`环境变量（如果指定）。您可以使用该[`pyenv shell`](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#pyenv-shell)命令在当前的Shell会话中设置此环境变量。
2. `.python-version`当前目录中的特定于应用程序的文件（如果存在）。您可以`.python-version`使用[``pyenv local``](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#pyenv-local)命令修改当前目录的 文件。
3. `.python-version`通过搜索每个父目录找到第一个文件（如果有），直到到达文件系统的根目录为止。
4. 全局`$(pyenv root)/version`文件。您可以使用[``pyenv global``](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#pyenv-global) 命令修改此文件。如果不存在全局版本文件，则pyenv假定您要使用“系统” Python。（换句话说，如果您没有pyenv，则可以运行任何版本 `PATH`。）

**注意:** 注意：您可以同时激活多个版本，同时包括多个版本的Python2或Python3。这允许Python2和Python3的同时使用，并且是诸如的工具所必需的`tox`。例如，要将路径设置为首先使用`system` Python和Python3（在此示例中分别设置为2.7.9和3.4.2），但还可以使用Python 3.3.6、3.2和2.5 `PATH`，则首先会`pyenv install`缺少版本，然后设置`pyenv global system 3.3.6 3.2 2.5`。在这一点上，人们应该能够使用来找到每个文件的完整可执行路径`pyenv which`，例如`pyenv which python2.5` （应显示`$(pyenv root)/versions/2.5/bin/python2.5`）或`pyenv which python3.4`（应显示系统Python3的路径）。您还可以在`.python-version`文件中指定多个版本，以换行符或任何空格分隔。

### 查找Python安装

pyenv确定了您的应用程序指定的Python版本后，它将命令传递给相应的Python安装。
每个Python版本均安装在自己的目录下 `$(pyenv root)/versions`。
例如，您可能已安装以下版本：
- `$(pyenv root)/versions/2.7.8/`
- `$(pyenv root)/versions/3.4.2/`
- `$(pyenv root)/versions/pypy-2.4.0/`

就pyenv而言，版本名称只是其版本的目录 `$(pyenv root)/versions`。
### 管理虚拟环境
有一个名为[pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)的pyenv插件，具有各种功能，可帮助pyenv用户管理virtualenv或Anaconda创建的虚拟环境。由于`activate`这些虚拟环境的脚本依赖于`$PATH`用户交互式外壳程序的变量，因此它将拦截pyenv的shim样式命令执行钩子。如果您有计划在这些虚拟环境中玩游戏，我们也建议您安装pyenv-virtualenv。

----


# 安装

### 安装前提条件:
为了使pyenv正确安装python，您应该 [**安装Python构建依赖项**](https://github.com/pyenv/pyenv/wiki#suggested-build-environment)构建依赖项。

### 自动安装程序
访问我的另一个项目：https://github.com/pyenv/pyenv-installer

### 基于GitHub Checkout安装
这将使您使用最新版本的pyenv，并轻松进行派生并向上游进行任何更改。

1. 安装git，curl和bash
```shll
       $ sudo apt-get install git curl bash
```

2. **在您要安装pyenv的地方签出**。 建议路径为`$HOME/.pyenv`（但您可以将其安装在其他位置）。

```shll
        git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```
（这一步是可选的）尝试编译动态bash扩展以加快pyenv的速度。不要担心它是否会失败；pyenv仍然可以正常工作：
```shll
        cd ~/.pyenv && src/configure && make -C src
```
3. **添加环境变量**。`PYENV_ROOT` 指向 pyenv 检出的根目录，并向 `PATH` 添加 `PATH‘添加‘PYENV_ROOT/bin` 以提供访问 `pyenv` 这条命令的路径

   - 如果你使用 bash shll，就依次执行如下命令：
     ~~~ bash
     echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
     echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
     ~~~

   - 如果你使用 **Ubuntu Desktop**就依次执行如下命令：
     ~~~ bash
     echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
     echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
     ~~~

   -  如果你使用 **Zsh**就依次执行如下命令：
     ~~~ zsh
     echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
     echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
     ~~~

   - 如果你使用 **Fish shell**就依次执行如下命令：
     ~~~ fish
     set -Ux PYENV_ROOT $HOME/.pyenv
     set -Ux fish_user_paths $PYENV_ROOT/bin $fish_user_paths
     ~~~

   - **代理注意**：如果您使用代理，出口http_proxy和HTTPS_PROXY
   - echo 命令的含义是：将引号中内容写入某文件中

4. **环境变量的设置 添加pyenv init到您的shll**以启用填充和自动补全功能。请确保`eval "$(pyenv init -)"`将其放置在shll程序配置文件的末尾，因为它会`PATH`在初始化期间进行操作。

   - 如果你使用 **Ubuntu Desktop**就执行如下命令：
     ~~~ bash
     echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
     ~~~

   - 如果你使用 **Ubuntu Desktop** 和 **Fedora**就执行如下命令：
     ~~~ bash
     echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc
     ~~~

   - 如果你使用 **Zsh**就执行如下命令:
     ~~~ zsh
     echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc
     ~~~

   - 如果你使用 **Fish shell**就执行如下命令:
     ~~~ fish
     echo -e '\n\n# pyenv init\nif command -v pyenv 1>/dev/null 2>&1\n  pyenv init - | source\nend' >> ~/.config/fish/config.fish
     ~~~
**一般警告**：在某些系统中，`BASH_ENV`变量被配置为指向`.bashrc`。在这样的系统上，您应该几乎应该将上述内容`eval "$(pyenv init -)"`放入 `.bash_profile`而**不是**放入`.bashrc`。否则因为在 pyenv 初始化期间会操作 path 环境变量，导致不可预测的行为，例如`pyenv`进入无限循环。有关详情，请参见 [#264](https://github.com/pyenv/pyenv/issues/264)。
    
5. **重新启动shll程序** 使路径更改生效。 您现在可以开始使用pyenv。
    ```sh
    exec "$SHELL"
    ```

6. **将Python版本安装到`$(pyenv root)/versions`中**。 例如，要下载并安装Python 2.7.8，请运行：
    ```sh
    pyenv install 2.7.8
    ```
   **注意**：如果需要通过configure选项进行构建，请使用```CONFIGURE_OPTS``` 环境变量。

   **注意**：如果要使用代理下载，请使用`http_proxy`和`https_proxy`环境变量。

   **注意**：如果您在安装python版本时遇到问题，请访问有关[常见构建问题](https://github.com/pyenv/pyenv/wiki/Common-build-problems)的Wiki页面
   
pyenv套件下插件：
- pyenv-doctor
- pyenv-installer
- pyenv-update
- pyenv-virtualenv
- pyenv-which-ext
.此时，你已经完成了 pyenv 的安装了，你使用可以它的全部命令了，但是我建议你先别急着用，一口气装完 pyenv 的一个插件，那就是 pyenv-virtualenv


### 升级

如果您使用**自动安装程序**安装了pyenv，请使用以下命令进行升级：
```sh
brew upgrade pyenv
```

如果您已按照上述说明安装了pyenv，则可以随时使用git升级安装。要升级到pyenv的最新开发版本，请使用`git pull`：

```sh
cd $(pyenv root)
git pull
```
要升级到特定版本的pyenv，请查看相应的标签：

```sh
cd $(pyenv root)
git fetch
git tag
git checkout v0.1.0
```

### 卸载pyenv

pyenv的简单性使其易于临时禁用它或从系统中卸载。
1. 要**禁用**pyenv管理您的Python版本，只需`pyenv init` 从Shell启动配置中删除该行即可 。这将从`PATH`中删除`python`  shims目录，并且像pythonpyenv之前一样，将来的调用 将执行系统Python版本。
pyenv 仍然可以在命令行上访问，但是您的Python应用程序不会受到版本切换的影响。

2. 要完全**卸载**pyenv，请执行步骤（1），然后删除其根目录。这将**删除**`` $(pyenv root)/versions/ ``目录下安装的**所有Python版本**：
    ```sh
    rm -rf $(pyenv root)
    ```
   如果您已经使用软件包管理器安装了pyenv，请在最后一步执行pyenv软件包删除。例如，对于Homebrew：
        `brew uninstall pyenv`

### 进阶设定

除非您必须知道shell配置文件中的每一行都在做什么，否则请跳过本节。
`pyenv init` 是唯一将额外命令加载到您的shell中的命令。来自rvm，有些人可能反对这个想法。下面是`pyenv init`实际执行：
   1. **设置您的shims路径**。这是pyenv正常运行的唯一要求。您可以通过手动添加`$(pyenv root)/shims`到自己的 `$PATH`。

   2. **安装自动完成功能**。这是完全可选的，但非常有用。采用`$(pyenv root)/completions/pyenv.bash`设置。 `$(pyenv root)/completions/pyenv.zsh`用户也有一个。

   3. **刷新shims**。您不时需要重建填充文件。在init上执行此操作可确保所有内容都是最新的。您始终可以`pyenv rehash` 手动运行。
   4. **安装sh调度程序**。该位也是可选的，但允许pyenv和插件更改当前shell中的变量，从而使命令成为`pyenv shell`可能。shll调度程序不会做任何疯狂的操作，例如覆盖`cd`或破解您的shell提示，但是如果由于某种原因您需要`pyenv`成为真正的脚本而不是shell函数，则可以安全地跳过它。


要了解自己的内幕，请运行 `pyenv init -`

如果您不想使用`pyenv init`和shims，仍然可以从pyenv为您安装Python版本的功能中受益。只需运行 `pyenv install` ，您将在中找到安装的版本 `$(pyenv root)/versions`，可以根据需要手动执行或符号链接。


### 卸载Python版本

随着时间的流逝，您将在`$(pyenv root)/versions`目录中累积Python版本 。

要删除旧的Python版本，请`pyenv uninstall`执行命令以自动执行删除过程。

或者，只需`rm -rf`删除要删除的版本的目录。您可以使用以下 `pyenv prefix`命令找到特定Python版本的目录 `pyenv prefix 2.6.8`。


----


## 命令参考

参见 [COMMANDS.md](COMMANDS.md).


----

## 环境变量
您可以通过以下设置影响pyenv的运行方式：

| 姓名 | 默认 | 描述 |
| --- | --- | --- |
| `PYENV_VERSION` |  | 指定要使用的Python版本。  
另见[`pyenv shell`](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#pyenv-shell) |
| `PYENV_ROOT` | `~/.pyenv` | 定义Python版本和垫片所在的目录。  
另见`pyenv root` |
| `PYENV_DEBUG` |  | 输出调试信息。  
也作为：`pyenv --debug <subcommand>` |
| `PYENV_HOOK_PATH` | [_参见维基_](https://github.com/pyenv/pyenv/wiki/Authoring-plugins#pyenv-hooks) | 用冒号分隔的路径列表搜索pyenv挂钩。 |
| `PYENV_DIR` | `$PWD` | 目录开始搜索`.python-version`文件。 |
| `PYTHON_BUILD_ARIA2_OPTS` |  | 用于将其他参数传递给[`aria2`](https://aria2.github.io/)。  
如果`aria2c`二进制文件在PATH上可用，则pyenv使用`aria2c`而不是`curl`或`wget`下载Python源代码。如果网络连接不稳定，则可以使用此变量来指示`aria2`加快下载速度。  
在大多数情况下，您只需要将其`-x 10 -k 1M`用作`PYTHON_BUILD_ARIA2_OPTS`环境变量的值 |


## 发展

pyenv源代码[托管在GitHub上](https://github.com/pyenv/pyenv)。即使您不是Shell黑客，它也干净，模块化且易于理解。

使用[Bats](https://github.com/bats-core/bats-core)测试:

    bats test
    bats/test/<file>.bats

请随时在 [问题跟踪](https://github.com/pyenv/pyenv/issues)上提交请求请求和文件错误。


  [pyenv-virtualenv]: https://github.com/pyenv/pyenv-virtualenv#readme
  [hooks]: https://github.com/pyenv/pyenv/wiki/Authoring-plugins#pyenv-hooks

### 版本记录

参见 [CHANGELOG.md](CHANGELOG.md).

### 执照

[MIT麻省理工学院执照](LICENSE)


### 如有纠正译文，请联系QQ1006399173
