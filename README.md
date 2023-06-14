# 小米10 Pro pixel experience plus 内核构建kernelSU教程

## 内核4.19 参考教程，直接使用kprobe集成，无法开机，故需要修改源码

### 1、fork https://github.com/xiaoleGun/KernelSU_Action

### 2、修改 config.env

#### Kernel Source

修改为你的内核仓库地址

例如: https://github.com/Diva-Room/Miku_kernel_xiaomi_wayne

修改为 https://github.com/kissunyeason/kernel_xiaomi_sm8250-immensity

这里需要修改内核，所以fork过来，自己方便修改
