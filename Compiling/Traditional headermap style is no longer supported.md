### Traditional headermap style is no longer supported

在用新的 Xcode 打开旧工程时经常会遇到这个问题，解决办法就是：将相应 Target 的 build setting 设置中 “Always Search User Paths” 选项设置为 “NO”，这样问题就解决了，可以这是为什么呢？

该设置已经于 Xcode 8.3 开始被废弃，强烈建议将其设置为“NO”。

如果开启此选项，无论 `#include <header.h>` 还是 `#include "header.h"`，两种形式的引用都会导致预处理优先在 “User Header Search Paths (USER_HEADER_SEARCH_PATHS)” 中检索相应头文件，而早于“Header Search Paths (HEADER_SEARCH_PATHS)”。如此会产生一个问题：假设工程中包含一个我们自己创建的 `String.h` ，此时我们又通过 `#include <String.h>` 的方式去引用系统的头文件，那么就会导致始终引用的是我们自己的头文件，而不是系统的。过去的解决办法是在相应的“User Header Search Path”引用项后添加一个 “-iquote” flag 标识，现在则不需要了。

弃用“Always search user paths” 后，如果要引用系统或者 Framework 形式的头文件，就用 `#include <header.h>` 的形式，找不到，可以将相应的路径添加到 “Header Search Paths” 中。要引用我们自己的创建的，或者是已引入工程的第三方源码头文件，请用 `#include "header.h"` 的形式，如果找不到，那么可以将相应的路径添加到 “User Header Search Paths” 中，这样两个路径独立，且各自独立搜索，就不会再出现“覆盖”的情况。