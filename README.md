# object4-781 源码调试和编译
## 源码下载地址

[Apple开发源码](https://opensource.apple.com/)：根据对应的版本找到相应的源码，例如：<font color=#FF4500 size=3> macos --> 10.15 -->搜索 objc4</font>

[MacOS 10.15源码](https://opensource.apple.com/release/macos-1015.html)：直接搜索想要下载的源码名称，例如：<font color=#FF4500 size=3>搜索objc --> objc4/ -->选择相应的objc版本下载 </font>

## 依赖资源文件

编译object4的时候需要添加一些依赖文件，这些依赖文件通过以下源码查找（文件名“-”后面的的是版本号）：

```
dyld-733.6
libauto-187
Libc-1353.41.1
libclosure-74
libdispatch-1173.40.5
libplatform-220
libpthread-416.40.3
xnu-6153.11.26
```

# 源码编译

## 1、`unable to find sdk 'macosx.internal'`

``选择`target ->objc -> Build Setting -> Base SDK - > macOS``

``选择`target ->objc-trampolines -> Build Setting -> Base SDK - > macOS``

## 2、`'sys/reason.h' file not found`

- 从下载的依赖资源文件中找`reason.h`，`reason.h`文件的目录`xnu/bsd/sys/reason.h`

- 在``objc4``目录下新建 ``include``文件，将`reason.h`文件放到``include/sys``文件夹下
- 设置`reason.h`路径，`target -> objc -> Build Settings - > Header Search Paths`中 添加``$(SRCROOT)/include``

## 3、`mach-o/dyld_priv.h' file not found`

- 在`include`文件中创建`mach-o`文件
- 在下载的依赖资源文件中找到 `dyld -> include -> mach-o -> dyld_priv.h`
- 拷贝到 `include/mach-o`文件夹下
- 打开`dyld_priv.h`文件，在顶部添加

```cpp
  #define DYLD_MACOSX_VERSION_10_11 0x000A0B00
  #define DYLD_MACOSX_VERSION_10_12 0x000A0C00
  #define DYLD_MACOSX_VERSION_10_13 0x000A0D00
  #define DYLD_MACOSX_VERSION_10_14 0x000A0E00
```

![dyld_priv添加宏定义](http://blog.guohuaden.com/dyld_priv_define%202.png)

## 4、`'os/lock_private.h' file not found`

- 在下载的资源文件`libplatform ->private -> os ->lock_private.h、base_private.h`两个文件，拷贝到`include/os`中

## 5、`'pthread/tsd_private.h' file not found`和`'pthread/spinlock_private.h' file not found`

- 在下载的资源文件`libpthread ->private ->tsd_private.h、spinlock_private.h`两个文件，拷贝到`include/pthread`中

## 6、`'System/machine/cpu_capabilities.h' file not found`

- 在下载的资源文件`xnu --> osfmk --> machine --> cpu_capabilities.h`文件，拷贝到`include/System/machine`中

## 7、`'os/tsd.h' file not found`

- 在下载的资源文件`xnu --> libsyscall --> os --> tsd.h`文件，拷贝到新建的`include/os`中

## 8、`'System/pthread_machdep.h' file not found`

- 在下载的资源文件`Libc --> pthreads --> pthread_machdep.h`文件，拷贝到`include/System`中

## 9、`'CrashReporterClient.h' file not found`

- 在[Libc-825.24](https://links.jianshu.com/go?to=https%3A%2F%2Fopensource.apple.com%2Fsource%2FLibc%2FLibc-825.24) 按路径`Libc/include/CrashReporterClient.h`找到对应文件导入`include`目录下
- 在`target -> Build Setting -> Preprocessor Macros`中加入`LIBC_NO_LIBCRASHREPORTERCLIENT`或者在`CrashReporterClient.h`文件中更改了里面的宏信息 `#define LIBC_NO_LIBCRASHREPORTERCLIENT`

## 10、`'objc-shared-cache.h' file not found`

- 在下载的资源文件`dyld --> include --> objc-shared-cache.h`文件，拷贝到`include`目录下中

## 11、`Mismatch in debug-ness macros`

- 将`objc-runtime.mm`  中 `#error mismatch in debug-ness macros`注释掉

## 12、`'_simple.h' file not found`

- 在下载的资源文件`libplatform --> private --> _simple.h`文件，拷贝到`include`目录下中

## 13、`'Block_private.h' file not found`

- 在下载的资源文件`llibclosure-74 --> Block_private.h`文件，拷贝到`include`目录下中

## 14、`'kern/restartable.h' file not found`

- 在下载的资源文件`xnu --> osfmk --> kern --> restartable.h`文件，拷贝到新建的`include/kern`中

## 15、`can't open order file: /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.15.sdk/AppleInternal/OrderFiles/libobjc.order`

- 在`target -> objc -> Build Setting -> Order File`中添加搜索路径`$(SRCROOT)/libobjc.order`

## 16、`/xcodebuild:1:1: SDK "macosx.internal" cannot be located.`

- 在`target -> objc -> Build Phases -> Run Script(markgc)`把脚本文本里的`macosx.internal`改成`macosx`

## 17、`library not found for -lCrashReporterClient`

- 在`target -> objc -> Build Settings -> Other -> Linker Flags -> Debug/Release -> Any macOS SDK`中删除`-lCrashReporterClient`

