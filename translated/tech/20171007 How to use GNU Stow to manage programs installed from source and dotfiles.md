如何使用 GNU Stow 来管理从源代码和 dotfiles 安装的程序
=====
dotfiles(**.**开头的文件在 *nix 下默认为隐藏文件)
### 目的

使用 GNU Stow 轻松管理从源代码和 dotfiles 安装的程序

### 要求

* root 权限


### 难度

简单

### 约定

* **#** \- 要求直接以 root 用户身份或使用 `sudo` 命令以 root 权限执行给定的命令
* **$** \- 给定的命令将作为普通的非特权用户来执行

### 介绍

有时候我们必须从源代码安装程序，因为它们也许不能通过标准渠道获得，或者我们可能需要特定版本的软件。 GNU Stow 是一个非常不错的 `symlinks factory` 程序，它可以帮助我们保持文件的整洁，易于维护。

### 获得 stow

你的 Linux 发行版本很可能包含 `stow`，例如在 Fedora，你安装它只需要：
```
# dnf install stow
```

在 Ubuntu/Debian 中，安装 stow 需要执行：
```
# apt install stow
```

在某些 Linux 发行版中，stow 在标准库中是不可用的，但是可以通过一些额外的软件源（例如 Rhel 和 CentOS7 中的epel ）轻松获得，或者，作为最后的手段，你可以从源代码编译它。只需要很少的依赖关系。

### 从源代码编译

最新的可用 stow 版本是 `2.2.2`。源码包可以在这里下载：`https://ftp.gnu.org/gnu/stow/`。

一旦你下载了源码包，你就必须解压它。切换到你下载软件包的目录，然后运行：
```
$ tar -xvpzf stow-2.2.2.tar.gz
```

解压源文件后，切换到 stow-2.2.2 目录中，然后编译该程序，只需运行:
```
$ ./configure
$ make

```

最后，安装软件包：
```
# make install
```

默认情况下，软件包将安装在 `/usr/local/`  目录中，但是我们可以改变它，通过配置脚本的 `--prefix` 选项指定目录，或者在运行 `make install` 时添加 `prefix="/your/dir"`。

此时，如果所有工作都按预期工作，我们应该已经在系统上安装了 `stow`。

### stow 是如何工作的？

stow 背后主要的概念在程序手册中有很好的解释：
```
Stow 使用的方法是将每个软件包安装到自己的树中，然后使用符号链接使它看起来像文件一样安装在普通树中

```

为了更好地理解这个软件的运作，我们来分析一下它的关键概念：

#### stow 文件目录

stow 目录是包含所有 `stow 包` 的根目录，每个包都有自己的子目录。典型的 stow 目录是 `/usr/local/stow`：在其中，每个子目录代表一个 `package`。

#### stow 包

如上所述，stow 目录包含多个 "包"，每个包都位于自己单独的子目录中，通常以程序本身命名。包不过是与特定软件相关的文件和目录列表，作为实体进行管理。

#### stow 目标目录

stow 目标目录解释起来是一个非常简单的概念。它是包文件必须安装的目录。默认情况下，stow 目标目录被认为是从目录调用 stow 的目录。这种行为可以通过使用 `-t` 选项（ --target 的简写）轻松改变，这使我们可以指定一个替代目录。

### 一个实际的例子

我相信一个好的例子胜过 1000 句话，所以让我来展示 stow 如何工作。假设我们想编译并安装 `libx264`，首先我们克隆包含其源代码的仓库：
```
$ git clone git://git.videolan.org/x264.git
```

运行该命令几秒钟后，将创建 "x264" 目录，并且它将包含准备编译的源代码。我们切换到 "x264" 目录中并运行 `configure` 脚本，将 `--prefix` 指定为 /usr/local/stow/libx264 目录。
```
$ cd x264 && ./configure --prefix=/usr/local/stow/libx264
```

然后我们构建该程序并安装它：
```
$ make
# make install
```

x264 目录应该在 stow 目录内创建：它包含所有通常直接安装在系统中的东西。 现在，我们所要做的就是调用 stow。 我们必须从 stow 目录内运行这个命令，通过使用 `-d` 选项来手动指定 stow 目录的路径（默认为当前目录），或者通过如前所述用 `-t` 指定目标。我们还应该提供要作为参数存储的包的名称。 在这种情况下，我们从 stow 目录运行程序，所以我们需要输入的内容是：
```
# stow libx264
```

libx264 软件包中包含的所有文件和目录现在已经在调用 stow 的父目录 (/usr/local) 中进行了符号链接，因此，例如在 `/usr/local/ stow/x264/bin` 中包含的 libx264 二进制文件现在在 `/usr/local/bin` 中符号链接，`/usr/local/stow/x264/etc` 中的文件现在符号链接在 `/usr/local/etc` 中等等。通过这种方式，系统将显示文件已正常安装，并且我们可以容易地跟踪我们编译和安装的每个程序。要恢复该操作，我们只需使用 `-D` 选项：
```
# stow -d libx264
```

完成了！符号链接不再存在：我们只是“卸载”了一个 stow 包，使我们的系统保持在一个干净且一致的状态。 在这一点上，我们应该清楚为什么 stow 还用于管理 dotfiles。 通常的做法是在 git 仓库中包含用户特定的所有配置文件，以便轻松管理它们并使它们在任何地方都可用，然后使用 stow 将它们放在适当位置，如放在用户主目录中。

Stow 还会阻止你错误地覆盖文件：如果目标文件已经存在并且没有指向 Stow 目录中的包时，它将拒绝创建符号链接。 这种情况在 Stow 术语中称为冲突。

就是这样！有关选项的完整列表，请参阅 stow 帮助页，并且不要忘记在评论中告诉我们你对此的看法。

--------------------------------------------------------------------------------

via: https://linuxconfig.org/how-to-use-gnu-stow-to-manage-programs-installed-from-source-and-dotfiles

作者：[Egidio Docile][a]
译者：[MjSeven](https://github.com/MjSeven)
校对：[校对者ID](https://github.com/ 校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]:https://linuxconfig.org
