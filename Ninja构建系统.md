## Ninja构建系统入门
### 1. 介绍
开篇先介绍、先甩资料给大家看，之后再自己演示一下基本使用。Ninja 是Google的一名程序员推出的注重速度的构建工具，一般在Unix/Linux上的程序通过make/makefile来构建编译，而Ninja通过将编译任务并行组织，大大提高了构建速度。

官网：[ninja-build.org](https://ninja-build.org/)

Github：[github.com/ninja-build/ninja](https://github.com/ninja-build/ninja)

### 2. 参考资料
[《The Performance Of Open Source Application》第三章](https://www.ituring.com.cn/article/265177)

[使用Ninja代替make](https://www.jianshu.com/p/d118615c1943)

[零壹軒Ninja相关文章（推荐！）](https://note.qidong.name/tags/ninja/)

[Ninja编译过程分析](https://www.cnblogs.com/wangym/p/8317310.html)

[Ninja - chromium核心构建工具](https://www.cnblogs.com/x_wukong/p/4846179.html)

### 3. 使用
#### 3.1. cmake生成
一般是通过cmake来生成ninja的配置，进而进行编译。先从cmake-examples入门：[github.com/ttroy50/cmake-examples](https://www.cnblogs.com/x_wukong/p/4846179.html)

比如01-basic\hello-headers项目，运行指令：cmake -Bbuild -GNinja 即可生成ninja工程。

```
$ mkdir build

$ cd build

$ cmake ..
-- The C compiler identification is GNU 4.8.4
-- The CXX compiler identification is GNU 4.8.4
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build

```

运行ninja编译：

#### 3.2. 手动写ninja配置文件
本文重点演示一下手写ninja配置文件方法，Demo工程结构：
```
./build.ninja
./src/jfz.cpp
```
其中jfz.cpp：

```
#include "Hello.h"

int main(int argc, char *argv[])
{
    printf("sandeepin poi!");
    return 0;
}
```

build.ninja（注意结尾要有空行）：

```
# 指定ninja最小需要版本
ninja_required_version = 1.5

# 变量
GCC = D:\Library\MinGW\bin\g++.exe
cflags = -Wall

# 编译规则，指定depfile，可以用于生成ninja_deps文件
rule compile_jfz
  command = $GCC -c $cflags -MD -MF $out.d $in -o $out
  description = 编译 $in 成为 $out
  depfile = $out.d
  deps = gcc
build jfz.o : compile_jfz src/jfz.c

# 链接规则
rule link_jfz
  command = $GCC $DEFINES $INCLUDES $cflags $in -o $out
  description = 链接 $in 成为 $out
build jfz.exe : link_jfz jfz.o

# 编译all，就是做任务build jfz.exe
build all: phony jfz.exe

# 默认编译什么(单独运行ninja)
default all
```

第一次运行按任务先编译，再链接，最终产生了可执行文件，第二次运行由于没改文件，ninja不处理。ninja支持如下参数：

```
--version  # 打印版本信息
-v         # 显示构建中的所有命令行（这个对实际构建的命令核对非常有用）

-C DIR     # 在执行操作之前，切换到`DIR`目录
-f FILE    # 制定`FILE`为构建输入文件。默认文件为当前目录下的`build.ninja`。如 ./ninja -f demo.ninja

-j N       # 并行执行 N 个作业。默认N=3（需要对应的CPU支持）。如 ./ninja -j 2 all
-k N       # 持续构建直到N个作业失败为止。默认N=1
-l N       # 如果平均负载大于N，不启动新的作业
-n         # 排练（dry run）(不执行命令，视其成功执行。如 ./ninja -n -t clean)

-d MODE    # 开启调试模式 (用 -d list 罗列所有的模式)
-t TOOL    # 执行一个子工具(用 -t list 罗列所有子命令工具)。如 ./ninja -t query all
-w FLAG    # 控制告警级别
```

ninja -d list相关：
```
debugging modes:
  stats        print operation counts/timing info 打印统计信息
  explain      explain what caused a command to execute 解释导致命令执行的原因
  keepdepfile  don't delete depfiles after they're read by ninja 读取depfile后，不删除它
  keeprsp      don't delete @response files on success 读取@response后，不删除它
  nostatcache  don't batch stat() calls per directory and cache them 不对每个目录批量处理stat()调用和缓存它们
multiple modes can be enabled via -d FOO -d BAR 多模式调用可以接着几个-d
```
ninja -w list相关，主要指定几种情况下告警级别是多少：
```
warning flags:
  dupbuild={err,warn}  multiple build lines for one target
  phonycycle={err,warn}  phony build statement references itself
  depfilemulti={err,warn}  depfile has multiple output paths on separate lines
```
ninja -t list相关，主要集成了graphviz等一些对开发非常有用的工具。

```
ninja subtools:
    browse  # 在浏览器中浏览依赖关系图。（默认会在8080端口启动一个基于python的http服务）
     clean  # 清除构建生成的文件
  commands  # 罗列重新构建制定目标所需的所有命令
      deps  # 显示存储在deps日志中的依赖关系
     graph  # 为指定目标生成 graphviz dot 文件。如 ninja -t graph all |dot -Tpng -o graph.png
     query  # 显示一个路径的inputs/outputs
   targets  # 通过DAG中rule或depth罗列target
    compdb  # dump JSON兼容的数据库到标准输出
 recompact  # 重新紧凑化ninja内部数据结构
```
这里主要列举几种参数执行效果：

-n是假执行，实际未产生文件，由于假执行，keepdepfile没起到效果，这个受限于编译器分析依赖，下面的统计信息就是stats效果，explain解释了为什么执行这些任务。

这里-v打印每个任务执行了哪些指令，可见到keepdepfile生效了，保存了依赖.d文件。

ninja工具举例：

1、显示依赖

```
$ninja -t deps
```

2、显示执行指令
```
$ninja -t commands jfz.exe
```

3、显示目标
```
$ninja -t targets all
```

4、绘依赖图（要安装graphviz，直接打印出dot文本）
```
$ninja -t graph
```

转图片（支持png、svg等，大图推荐svg渲染，相关dot参数见graphviz文档）：
```
ninja -t graph | dot -Tpng -o jfz.png
```


这个demo比较简单，实际上依赖分析功能需要编译器提供，或者任务自己输出依赖文件，ninja只做一个任务编排和执行功能。

### 4. 信息补充
#### 4.1. 环境变量
通过环境变量NINJA_STATUS可以控制ninja打印进度状态的样式，有几个占位符：

```
%s 起始edges的数量。
%t 完成构建必须运行的edges总数。
%p 起始edges的百分比。
%r 当前运行的edges数。
%u 要开始的剩余edges数。
%f 完成的edges数。
%o 每秒完成edges的总速率
%c 当前每秒完成edges的速率（由-j或其默认值指定的构建的平均值）
%e 经过的时间（以秒为单位）。（自Ninja 1.2起可用。）
%% 一个普通的%字符。
默认进度状态为"[%f/%t] "（请注意尾随空格以与构建规则分开）。可能的进度状态的另一个示例可能是"[%u/%r/%f] "。
```
尝试改为export NINJA_STATUS="[%p/%f/%t %e] "（Windows下set NINJA_STATUS="[%p/%f/%t %e] "）的效果如下：

#### 4.2. ninja_log每项含义

依次为：开始时间、结束时间、mtime、output文件路径名、命令行hash。

其中mtime是输入文件们的最后修改时间的时间戳算出来的值，经测试，开始时间、结束时间、命令行hash均不会影响增量的判定。

#### 4.3. mtime检查文件测试
假设现在时间是2019-12-31 15:35:55，将输入.c文件修改时间改为之前的，不会触发重新编译；将输入.c文件修改时间改为未来的，每次都触发编译。

#### 4.4. frontend_file参数
特别的，AOSP定制版的ninja有frontend_file参数，可以将控制台输出信息转为流存储，通过其它的工具如tail -f xxx查看信息。soong源码中就是这样读取ninja日志的，效率更高，之前应该是靠cat捕获日志的吧。见这个提交。

#### 4.5. ninja检测的是任务名的文件是否生成
如果让输出文件和任务名不一样，ninja每次都会重新编译：

这点要注意，为了利用ninja的增量特性，除非迫不得已，不要让输出文件和任务名不同。

#### 源码编译
本文仅尝试官方版的编译，AOSP版本可以依赖不同，GCC版本要求不同，需要注意。
