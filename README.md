# 小米10 Pro pixel experience plus 内核构建kernelSU教程

## 内核4.19 参考教程，直接使用kprobe集成，无法开机，故需要修改源码

## 1、fork https://github.com/xiaoleGun/KernelSU_Action

## 2、修改 config.env

### Kernel Source

修改为你的内核仓库地址

例如: https://github.com/Diva-Room/Miku_kernel_xiaomi_wayne

修改为 https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity

这里需要修改内核，所以fork过来，自己方便修改，原内核请查看上个fork

### Kernel Source Branch

修改为你的内核分支

例如: TDA

这里我们修改为thirteen （安卓13）

### Kernel Config

修改为你的内核配置文件名

例如: vendor/wayne_defconfig

这里我们修改成vendor/cmi_defconfig  一定要找到对应的文件名称

### Arch

例如: arm64

这里不动

### Kernel Image Name

修改为需要刷写的 kernel binary，一般与你的 aosp-device tree 里的 BOARD_KERNEL_IMAGE_NAME 是一致的

例如: Image.gz-dtb

常见还有 Image、Image.gz

这里其实就是内核格式，也是翻看文件，找到

我用的内核源码里面格式没有后缀，填写 Image

### Clang

Use custom clang

可以使用除 google 官方的 clang，如proton-clang

这里建议不懂得不要乱填

### Custom Clang Source

如果是 git 仓库，请填写包含.git的链接

支持 git 仓库或者 zip 压缩包的直链

这里建议不懂得不要乱填

### Custom cmds

都用自定义 clang 了，自己改改这些配置应该都会吧 :)

### Clang Branch

由于 [#23](https://github.com/xiaoleGun/KernelSU_Action/issues/23) 的需要，我们提供可自定义 Google 上游分支的选项，主要的有分支有
| Clang 分支 |
| ---------- |
| master |
| master-kernel-build-2021 |
| master-kernel-build-2022 |

或者其它分支，请根据自己的需求在 https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86 中寻找

### Clang version

填写需要使用的 Clang 版本
| Clang 版本 | 对应 Android 版本 | AOSP-Clang 版本 |
| ---------- | ----------------- | --------------- |
| 12.0.5 | Android S | r416183b |
| 14.0.6 | Android T | r450784d |
| 14.0.7 | | r450784e |
| 15.0.1 | | r458507 |

一般 Clang12 就能通过大部分 4.14 及以上的内核的编译

CLANG_BRANCH=master-kernel-build-2022
CLANG_VERSION=r450784d

不懂就照着我这个填写

### GCC

#### Enable GCC 64

启用 GCC 64 交叉编译

#### Enable GCC 32

启用 GCC 32 交叉编译

### Extra cmds

有的内核需要加入一些其它编译命令，才能正常编译，一般不需要其它的命令，请自行搜索自己内核的资料
请在命令与命令之间用空格隔开

例如: LLVM=1 LLVM_IAS=1

### Enable KernelSU

启用 KernelSU，用于排查内核故障或单独编译内核

这里一定要写为true，不然你以为你在干什么？

#### KernelSU Branch or Tag

选择 KernelSU 的分支或 tag:

- main 分支(开发版): `KERNELSU_TAG=main`
- 最新 TAG(稳定版): `KERNELSU_TAG=`
- 指定 TAG(如`v0.5.2`): `KERNELSU_TAG=v0.5.2`

默认就好了

### Disable LTO

LTO 用于优化内核，但有些时候会导致错误

### Disable CC_WERROR

用于修复某些不支持或关闭了Kprobes的内核，修复KernelSU未检测到开启Kprobes的变量抛出警告导致错误

这里关闭，填false

### Add Kprobes Config

自动在 defconfig 注入参数

这里别动了，反正要修改内核

### Add overlayfs Config

此参数为 KernelSU 模块和 system 分区读写提供支持

自动在 defconfig 注入参数

这里别动了，反正要修改内核

### Enable ccache

启用缓存，让第二次编译内核更快，最少可以减少 2/5 的时间

### Need DTBO

上传 DTBO


部分设备需要

这里不需要开启

### Build Boot IMG

> 从之前的 Workflows 合并进来的，可以查看历史提交

编译 boot.img，需要你提供`Source boot image`

### Source Boot Image

故名思义，提供一个源系统可以正常开机的 boot 镜像，需要直链，最好是同一套内核源码以及与你当前系统同一套设备树从 aosp 构建出来的。ramdisk 里面包含分区表以及 init，没有的话构建出来的镜像会无法正常引导。

例如: https://raw.githubusercontent.com/xiaoleGun/KernelSU_action/main/boot/boot-wayne-from-Miku-UI-latest.img

这里的建议是自己把对应ROM的boot.img提取出来作为release发布，然后复制地址就可以了
例如我：https://github.com/kissunyeason/Xiaomi10Pro_Pixel_Experience_Plus_Kernel_with_SU/releases/download/offical-boot/boot.img

## 3、开始编译
### 点到action
### build-kernel
### run workflow
### 等待编译完成
### 下载AnyKernel3-KernelSU-cmi-xxxxxxx.zip 
### 用TWRP刷入
### 运气好的话，应该开不了机（记得提前备份boot.img）
### 把手机扔垃圾桶

## 4、修改内核
### ① 修改 fs/exec.c（在你fork的内核源码改！）

找到下面这段话(大概1916行)

```C
return retval;
```



  和

static int do_execveat_common

之间添加这两行（分两次复制）

extern int ksu_handle_execveat(int *fd, struct filename **filename_ptr, void *argv,
		
void *envp, int *flags);

参考
https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/bd6276dd5249b85ada5b6caf479e5c74dd269639

还是这个文件

找到(大概1923行)

struct user_arg_ptr envp,

int flags)
			      
{

和

return __do_execve_file(fd, filename, argv, envp, flags, NULL);

之间插入

ksu_handle_execveat(&fd, &filename, &argv, &envp, &flags);

参照

https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity/commit/a0dfa44cbe79a2a532aadcfd33919e38ad753f26

