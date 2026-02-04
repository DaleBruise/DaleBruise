# VScode C++ 环境配置
## VScode

很多人的 C++ 都是从 VisualStudio 开始的，但是使用 VisualStudio 就代表着你的 C++ 开发无时无刻不在被微软平台绑定。并且 VS 很笨重，很多的开发库你也用不到，那么有没有轻量化一点的编辑器，但是又不像 Vim、Emacs 那样学习成本那么高呢？

有的，就是耳熟能详的 VScode。VScode 就是一个学习成本较低的轻量化的编辑器，其只要下载对应的扩展包就可以运作相应的功能。这篇文章就是来分享我的 VScode 的环境配置过程，其实非常轻松和简单。

软硬件情况：

- MacOS Tahoe26.2
- Apple M2 Pro

其实硬件没什么要求，这种编辑器一般软硬件兼容性较高，但是我推荐内存至少要在 16GB 以上，并且要好一点的内存哈。

## 准备工作

首先对于 MacOS 来讲，环境配置相对 Windows 会简单很多，我们需要下载 Xcode（*一般直接在 App Store 上下载即可*）。Xcode 包含了 clang++ 和 lldb 这两个必要的库。一般 MacOS 上用 lldb 就可以了。

在命令行上输入以下指令，查看你是否已经正确安装了这两个库：  

```bash title="bash" linenums="1"
clang++ --version
lldb --version
```

如果没有正常弹出版本信息，那么你就重新安装一下 Xcode，Xcode 也可以在 bash 中下载：  

```bash title="bash" linenums="1"
xcode-select --install
```

然后我们需要安装一些必要的 VScode 扩展：

- C/C++ author：Microsoft
- C/C++ Extension Pack author：Microsoft

其他的扩展根据的你需要可以按需下载安装。

## 配置 tasks.json 和 launch.json

准备工作完成之后，就可以直接在工作区创建 main.cpp 文件编写 C/C++ 程序了。但是这样有一个问题，就是我们没有办法添加多个 cpp 文件，这样就失去了 C/C++ 的特性，以及面向对象的能力，美观度也大打折扣，不能全部写在一个文件里面吧？太没有可视性了！

所以我们要建立能运行中小型项目的工程工作区，一般来说经典的目录如下：

```text title="text" linenums="1"
MyCppProject/
├── src/main.cpp
├── include/MyList.h
└── .vscode/       ← 即将配置
```

src 存放 CPP 文件，include 存放 hpp 文件，然后还可以建立一个 build 文件夹，来存放一些生成的文件。接下来，我们直接在 .vscode 这个目录下创建两个 json 文件：tasks.json 和 launch.json。我先贴出我的配置文件，然后再对其进行说明：

```json title="tasks.json" linenums="1"
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "shell",
            "label": "build",
            "command": "/usr/bin/clang++",
            "args": [
                "-g",
                "src/*.cpp",
                "-I",
                "include",
                "-o",
                "build/app",
                "-std=c++17"
            ],
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "group": "build"
        },
        {
            "label": "run",
            "type": "shell",
            "command": "./build/app",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "dependsOn": "build",
            "group": "build",
            "problemMatcher": []
        }
    ]
}
```

```json title="launch.json" linenums="1"
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug C++",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/build/app",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "lldb",
      "preLaunchTask": "build"
    }
  ]
}
```

OK，直接进行复制粘贴过后，你需要对部分内容进行更改，先来讲 tasks.json ：

- label 是用来供 VScode 识别不同 task ，我们 tasks.json 的任务就是用来进行编译任务的。其实底层上还是在命令行上调用指令，tasks 只是把指令给你使用这种好看的方式给你展现，并且更改的。第二个 task 如果你只是 DEBUG 的话，可以不粘贴上去。
- args 是编译过程需要的必要参数。其中，**-g** 用来生成调试信息；**src/*.cpp**识别你的所有的 cpp 文件；**-I**，**include**两个用来识别你添加的头文件；**-o**，**build/app**放置所有的生成文件，并且最终的可执行文件的名字就是这个 app，你可以改名字。可能会有个疑问就是问什么只有一个点o 文件？不是所有的 cpp 都会生成 .o？但是其实我们最终只是需要执行 main 函数的 .o，工业界的话，一般会生成 dll（动态扩展包）。我们如果是需要执行代码生成一些结果进行测试的话，就只需要生成一个 .o 文件即可。

接下来对于 launch.json：

- preLaunchTask 这里填写你命名的 task label 就行，名字要和 task 中的一样。作用就是用来识别你的 task，因为总会有不同的编译目标。

然后这两个文件就配置好了，然后就愉快地进行 coding 就好了。

