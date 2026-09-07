# Task 4：思考题

## 1. Make 和简单的 build.sh 有什么区别？

build.sh 只是把若干条 gcc 命令按固定顺序依次执行：每次运行都会从头到尾把里面所有命令无条件执行一遍，不管源文件有没有变化；命令顺序写死，如果中间某一步失败，后面的命令通常还会继续执行（除非手动加 set -e 或用 && 连接）；它没有"哪些文件需要重新编译"的概念，也不会增量构建。

Make 是按"目标（target）和依赖（prerequisite）"来工作的：make 会比较源文件和 .o 文件的时间戳，只重新编译那些"过期"的文件，没有变化的文件直接复用旧的 .o，最后再重新链接；还支持 make -j 并行构建。修改 src/calculator.c 后再运行 make，只会重新编译 calculator.c 并重新链接，而 main.o 等没有变化的文件不会被重编。

共同点：Make 和 build.sh 本身都不是编译器，最终调用的都是 gcc。区别在于 Make 具备依赖跟踪和增量构建能力，而 build.sh 只是一个"每次全量执行"的简单脚本。

## 2. CMake 是编译器吗？

不是。CMake 是一个构建系统生成器：它读取 CMakeLists.txt，生成一份 Makefile（在 Linux 上默认）或 Ninja 等底层构建工具需要的文件，本身并不直接编译代码。

执行 cmake --build build 时，CMake 会调用 build 目录里生成的底层构建工具（默认是 make），由 make 根据时间戳判断哪些文件需要重新编译，最终真正执行编译的是 C 编译器 gcc（来自 build-essential 包）。可以用 cmake --build build --verbose 查看，能看到实际执行的是 gcc -c src/main.c ... 和 gcc ... -o calculator 这类命令。

完整流程：CMakeLists.txt -> cmake（生成 Makefile）-> make（判断增量）-> gcc（编译和链接）-> calculator 可执行文件。

## 3. 为什么不希望每次都重新编译所有 .c 文件？

因为编译本身很耗时：每个 .c 文件都要经过预处理、语法分析、代码生成等阶段，文件越多、项目越大，全量编译花费的时间越长。

如果只修改了其中一个 .c 文件，只有它对应的 .o 目标文件是"过期"的，其它 .o 文件的内容并没有变化，完全可以继续复用，最后只需要把改过的那个文件重新编译，再重新链接一次就可以了。这样增量构建通常能把一次构建从几分钟甚至几小时缩短到几秒钟。

需要注意一个例外：如果修改的是被很多源文件 include 的公共头文件，那么所有包含该头文件的源文件都需要重新编译，因为头文件的内容已经影响到了它们。这也正是 Makefile 里 main.o 的依赖要写上 include/calculator.h、include/logger.h 的原因。
