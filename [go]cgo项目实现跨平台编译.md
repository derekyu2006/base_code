### 什么是跨平台编译？
就是在一个平台上生成另一个平台上的可执行代码。这里所谓平台，实际上包含两个概念：体系架构(Architecture)、操作系统 (Operating System）。

同一个体系架构可以运行不同的操作系统; 同一个操作系统也可以在不同的体系架构上运行。

Go是支持跨平台编译的, 其实现跨平台编译的思想其实很简单：通过保存可以生成最终机器码的多份翻译代码，在编译时根据GOARCH=xxx 和GOOS=xxx参数（对应体系架构和操作系统）进行初始化设置，最终调用对应平台编写的特定方法来生成机器码，从而实现跨平台编译。

### CGO编译存在的问题
有一点需要注意：Go所谓的跨平台编译只是针对Go代码部分，它是Go的交叉编译器(cross-compiler toolchains)。当我们使用了CGO时，要想实现跨平台编译, 需要让C/C++代码也支持跨平台。

```
package main

/*
#include <stdio.h>

void printint(int v) {
    printf("printint: %d\n", v);
}
*/
import "C"

func main() {
    v := 42
    C.printint(C.int(v))
}
```

小菜刀的开发机器：amd64架构，darwin系统。目标编译平台：amd64架构，linux系统。现想将上述含有CGO的代码编译为目标平台的可执行文件。

`$GOOS=linux GOARCH=amd64 CGO_ENABLED=1 go build -o main main.go`

通过以上命令，得到编译错误如下:

```
/usr/local/go/pkg/tool/darwin_amd64/link: running clang failed: exit status 1
ld: warning: ignoring file /var/folders/xk/gn46n46d503dsztbc6_9qb2h0000gn/T/go-link-220081766/go.o, building for macOS-x86_64 but attempting to link with file built for unknown-unsupported file format ( 0x7F 0x45 0x4C 0x46 0x02 0x01 0x01 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 )
Undefined symbols for architecture x86_64:
  "_main", referenced from:
     implicit entry/start for main executable
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

可以看到，由于CGO的存在，跨平台编译失败。那该如何解决呢？
其实思路可以很简单: 和Go一样，当我们拥有目标平台的C/C++代码翻译系统后，自然就能够编译为目标平台的可执行文件。

### Mac下的可行方案
下载linux编译工具链
`brew install FiloSottile/musl-cross/musl-cross`

或者windows编译工具链
`brew install mingw-w64`

以linux编译工具链为例。在下载完毕后，/usr/local/bin下会存在以下对应平台C/C++编译器
```
x86_64-linux-musl-addr2line   x86_64-linux-musl-elfedit     x86_64-linux-musl-gcov        x86_64-linux-musl-objcopy
x86_64-linux-musl-ar          x86_64-linux-musl-g++         x86_64-linux-musl-gcov-dump   x86_64-linux-musl-objdump
x86_64-linux-musl-as          x86_64-linux-musl-gcc         x86_64-linux-musl-gcov-tool   x86_64-linux-musl-ranlib
x86_64-linux-musl-c++         x86_64-linux-musl-gcc-9.2.0   x86_64-linux-musl-gprof       x86_64-linux-musl-readelf
x86_64-linux-musl-c++filt     x86_64-linux-musl-gcc-ar      x86_64-linux-musl-ld          x86_64-linux-musl-size
x86_64-linux-musl-cc          x86_64-linux-musl-gcc-nm      x86_64-linux-musl-ld.bfd      x86_64-linux-musl-strings
x86_64-linux-musl-cpp         x86_64-linux-musl-gcc-ranlib  x86_64-linux-musl-nm          x86_64-linux-musl-strip
```

上述指定下载命令只下载了x86_64体系下的编译器，但其实并不止这些。可通过brew info musl-cross命令进行查看。

```
$ brew info musl-cross
filosottile/musl-cross/musl-cross: stable 0.9.9 (bottled), HEAD
Linux cross compilers based on musl libc
https://github.com/richfelker/musl-cross-make
/usr/local/Cellar/musl-cross/0.9.9 (1,851 files, 245.8MB) *
  Poured from bottle on 2020-11-16 at 17:09:31
From: https://github.com/filosottile/homebrew-musl-cross/blob/master/musl-cross.rb
==> Dependencies
Build: gnu-sed ✔, make ✔
==> Options
--with-aarch64
    Build cross-compilers targeting arm-linux-muslaarch64
--with-arm
    Build cross-compilers targeting arm-linux-musleabi
--with-arm-hf
    Build cross-compilers targeting arm-linux-musleabihf
--with-i486
    Build cross-compilers targeting i486-linux-musl
--with-mips
    Build cross-compilers targeting mips-linux-musl
--with-mips64
    Build cross-compilers targeting mips64-linux-musl
--with-mips64el
    Build cross-compilers targeting mips64el-linux-musl
--with-mipsel
    Build cross-compilers targeting mipsel-linux-musl
--without-x86_64
    Do not build cross-compilers targeting x86_64-linux-musl
--HEAD
    Install HEAD version
```

此时, 通过指定C/C++编译器为/usr/local/bin/x86_64-linux-musl-gcc, 替换成默认的C/C++编译器(本机编译，可通过go env CC查看), 即可完成含有CGO的Go代码交叉编译任务。
```
$ GOOS=linux CC="/usr/local/bin/x86_64-linux-musl-gcc" GOARCH=amd64 CGO_ENABLED=1 go build -ldflags "-linkmode external -extldflags -static" main.go
```

最终，在本机mac系统上就编译得到了amd64 linux平台的可执行文件。