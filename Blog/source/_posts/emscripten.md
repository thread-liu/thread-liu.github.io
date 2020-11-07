---
title: Emscripten 使用指南
date: 2020-11-7
tags: Emscripten
---

## 要事优先

首先是确保你已经下载并且安装好了 Emscripten。根据你的操作系统不同，下载和安装过程稍有不同。

Emscripten 主要是通过 emcc（Emscripten Compiler Frontend）来工作的。这是个命令行工具，它会调用其他编译需要的工具，可以将它看成是标准编译器比如 gcc 或者 clang 的命令行版本。wimdows 系统的话，命令行中使用 emcc，Linux 下使用./emcc。

## 验证 Emscripten

第一次使用 Emscripten，请先使用以下命令验证 Emscripten 是否正确安装：

```text
emcc -v
```

如果有警告发生，可能是因为缺少一些工具，请去看 [这个链接解决]([Verifying the Emscripten Development Environment](https://link.zhihu.com/?target=http%3A//kripken.github.io/emscripten-site/docs/building_from_source/verify_emscripten_environment.html%23verifying-the-emscripten-environment))

如果没有警告或报错，就往下看。

## 运行 Emscripten

现在就可以使用 Emscripten 把 C/C++ 代码编译成 JavaScript 了。

首先，写一个待编译为 JavaScript 的 C 文件。比如 hello_world.c，像下面这样：

```text
#include <stdio.h>
   int main() {
     printf("hello, world!\n");
     return 0;
   }
```

注：这是 Emscripten 提供的测试集中最简单的一个 C 文件。

为了编译这个 C 文件，你只需要在 test 目录下打开命令行，emcc 后面跟上这个文件名就行了。

```text
emcc tests/hello_world.c
```

注：test 目录是 Emscripten 测试集的目录。

这样就会在 test 目录下生成一个 a.out.js 文件，你可以使用 node 运行这个 a.out.js。

```text
node a.out.js
```

就会在 node 控制台打印出 hello world！了。

```text
如果编译失败，你可以在 emcc tests/hello_world.c 后面加个 - v，也就是变成 emcc tests/hello_world.c -v ，这样呢就会有一些调试信息，你可以参考他们，找出编译失败的原因。
```



## 生成 HTML

Emscripten 能为刚才输出的那个 JavaScript 生成 HTML 文件，你可以使用 - o 命令指定要输出的 html 文件名。

```text
emcc tests/hello_world.c -o hello.html
```

在浏览器打开这个 hello.html。你会看到，这个 HTML 页面中有一块文本区域是为了显示 C 代码中 printf() 函数打印的内容。

实际上，这块区域不仅可以显示文本。如果你 C 代码中调用了 SDL 的 API，那么也可以在一块 canvas 中显示一个五彩斑斓的 cube。比如，hello_world_cube.cpp 那个测试用例就是这样。那个测试用例的代码是：

```text
#include <stdio.h>
#include <SDL/SDL.h>

#ifdef __EMSCRIPTEN__
#include <emscripten.h>
#endif

extern "C" int main(int argc, char** argv) {
 printf("hello, world!\n");

 SDL_Init(SDL_INIT_VIDEO);
 SDL_Surface *screen = SDL_SetVideoMode(256, 256, 32, SDL_SWSURFACE);

#ifdef TEST_SDL_LOCK_OPTS
 EM_ASM("SDL.defaults.copyOnLock = false; SDL.defaults.discardOnLock = true; SDL.defaults.opaqueFrontBuffer = false;");
#endif

 if (SDL_MUSTLOCK(screen)) SDL_LockSurface(screen);
 for (int i = 0; i < 256; i++) {
   for (int j = 0; j < 256; j++) {
#ifdef TEST_SDL_LOCK_OPTS
     // Alpha behaves like in the browser, so write proper opaque pixels.
     int alpha = 255;
#else
    // To emulate native behavior with blitting to screen, alpha component is ignored. Test that it is so by outputting
     // data (and testing that it does get discarded)
     int alpha = (i+j) % 255;
#endif
     *((Uint32*)screen->pixels + i * 256 + j) = SDL_MapRGBA(screen->format, i, j, 255-i, alpha);
   }
 }
 if (SDL_MUSTLOCK(screen)) SDL_UnlockSurface(screen);
 SDL_Flip(screen);

 printf("you should see a smoothly-colored square - no sharp lines but the square borders!\n");
 printf("and here is some text that should be HTML-friendly: amp: |&| double-quote: |\"| quote: |'| less-than, greater-than, html-like tags: |<cheez></cheez>|\nanother line.\n");

 SDL_Quit();

 return 0;
}
```

## 使用文件

```text
C/C++ 中，可以用 libc 库的 fopen,fclose 等 API 来访问文件
```

js 运行在浏览器的沙盒环境中，并不能直接访问本地文件系统，不过，Emscripten 模拟了一个文件系统，这样你可以在你的 C/C++ 代码中继续使用 libc 的 API。

你想访问的文件应该通过 preload 或者 embedded 的方式打包到 Emscripten 虚拟的文件系统中。

测试集中，hello_world_file.cpp 展示了怎么加载一个文件。测试代码和测试文件 hello_world_file.txt 如下面所示：

```text
#include <stdio.h>
   int main() {
     FILE *file = fopen("tests/hello_world_file.txt", "rb");
     if (!file) {
       printf("cannot open file\n");
       return 1;
     }
     while (!feof(file)) {
       char c = fgetc(file);
       if (c != EOF) {
         putchar(c);
       }
     }
     fclose (file);
     return 0;
   }
```

```text
==
   This data has been read from a file.
   The file is readable as if it were at the same location in the filesystem, including directories, as in the local filesystem where you compiled the source.
   ==
```

下面命令是 ** 在任何编译代码运行前 ** 指定一个数据文件预加载到 Emscripten 的虚拟文件系统。这个方法很有用，因为浏览器只能异步获取数据的，而原生代码（C/C++）很多都是使用的同步文件 API，那么，用这个方法可以确保数据加载完成之前，编译代码（C/C++ 编译之后的 js）不会从 Emscripten 的虚拟文件系统中去取数据，也就不会出错。下面是编译命令：

```text
./emcc tests/hello_world_file.cpp -o hello.html --preload-file tests/hello_world_file.txt
```

运行生成的 HTML，就能看到 hello_world_file.txt 文件的内容。

## 优化代码

默认情况下，和 gcc 以及 clang 等编译器一样，Emscripten 生成的编译代码没有经过编译优化。那么，你可以在命令行参数中使用 - O1，生成轻微优化的代码。

```text
./emcc -O1 tests/hello_world.cpp
```



因为编译生成 a.out.js 的过程实际上并不真的需要优化，所以实际上你加不加 - O1，从编译时间（或者说编译速度）上，你是看不出区别的。

但是真的没区别吗？

你可以看看生成的 a.out.js 文件，就能发现还是有区别的。-O1 的优化有一些微小的优化并且清除了一些运行时断言，比如，在生成的代码中，printf 函数，会被替换成 put。

想编译优化，不仅可以用 - O1，还可以用 - O2，-O2 优化的程度更厉害。你可以试一下，它的编译代码跟 - O1 又有很大差别。

## Emscripten 测试集

Emscripten 给大家提供了非常多的测试用例，几乎覆盖了 Emscripten 的所有功能。对于开发者来说，这是非常好的资源。

关于测试集的更多情况，[可以点击了解]([Emscripten Test Suite](https://link.zhihu.com/?target=http%3A//kripken.github.io/emscripten-site/docs/getting_started/test-suite.html%23emscripten-test-suite))。