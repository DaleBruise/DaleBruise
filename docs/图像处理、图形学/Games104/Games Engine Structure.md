# 1. Games Engine Sturcture

## 1.1 引擎导论

### 我们为什么需要游戏引擎？或者说为什么我们需要引擎？

游戏引擎出现的目的就是为了在数字世界中实现现实世界的物理世界。通过将抽象的物理公式、数学公式在计算机中可视化、具像化。

### Father of Game Engine——**John Karmac**

Karmac做的工作就是将当年的大部分游戏中代码中的大部分重复代码进行整理和重构，并且抽离出来作为一个独立的平台进行开发。

### OpenGL 3D Graphic Acceleration

**Quarc**这款游戏中，是一款典型的初代将图形卡融入计算机图形学计算加速的游戏。

### What is Game Engine？

- Tech foundation of *Matrix*
- A tool of creation
- Mix of Art 

### RealTime

游戏引擎有一个很重要的特点——实时性。因为我们作为**控制器**，大脑在不断地进行思考，不断地对计算机进行输入操作，同时我们也在不断地需要对应输入的输出从而得到反馈后，进行下一步输入操作，因此游戏引擎的实时性是很重要的。

但是对于现代计算机的架构来讲，游戏引擎会得到很多条件的约束，例如：计算效率、存储上限、带宽限制、传输时间等等，从而计算机是无法真正地模拟出现实世界的每一分每一秒的细节。

同时这也决定着游戏引擎将会是一个非常复杂、庞大的系统，这不仅包含Shading，还会包含很多的内容和架构体系，大致体量从300万行到1000万行代码不等。

### Course Content

1. Basic Element
2. Rendering
3. Animation
4. Physics
5. Gameplay
6. Misc. System(Effect Navigation Camera)
7. Tool Set
8. Online Gaming

## 1.2 引擎的架构

游戏引擎的五层架构:

1. Tool Layer
2. Fucntion Layer
3. Resource Layer
4. Core Layer
5. PLatform Layer

### Resrouce Transformation

Data to Asset. 游戏会用到很多种不同格式的文件，以及通用格式文件。但是这些文件的格式都是不同的，加密方式和压缩方式也会很复杂，这并不利于GPU和CPU去读取和计算。所以我们要将这些**Data**转换成游戏引擎适合的格式，从而成为我们引擎的**Asset**。并且我们要对每一个**Asset**进行编号，从而来找到其所在的位置，编号的方式就是**GUID**。

### Resource Runtime Manager

文件Asset进入引擎运行之后，需要对其进行管理，来保证游戏中的Asset是能够有一定的指向性。就比如某一个文件是用来进行A操作，那么这个文件就要被送到A操作的进程中去。

### Resource LifeCycle Manager

这个部分用来管理文件Asset的资源如何进行回收和分配。

### Function How to Make World Alive

Tick，我们使用Tick来在很短的时间内，把游戏世界的所有模块全部跑一遍

### Core Math Efficiency

由于很多时候，在游戏引擎中不需要太精确的计算，所以我们可以使用一些很巧妙的定制数学计算。

### Core Data Structure and Memory Management

一般来讲，C++的STL库的效率不会很高，所以为了更高的内存使用利用率，要定制数据结构。这就是为了要进行极致的内存管理。内存的管理大致的原则有三点：

- 数据要遵循局部性原理
- 按照一定的顺序访问数据
- 按照一定的块或者页来进行分配和回收

### Platform Target on Different Platform

RHI，由于不同的平台会使用不同的图形驱动，所以我们需要写一个中间层来封装不同驱动之间的转换。

### Tool Allow Anyone to Make Game

Like 不同功能开放的编辑器，对应着不同层能够实现的功能，并将其可视化，通过点点鼠标的方式来方便开发人员进行开发游戏。
