---
title: 在 Linux 中安装 NVIDIA 驱动
data: 2024-04-23T10:12:11+08:00
categories:
  - note
tags:
  - note
  - nvidia
  - driver
  - power
draft: false
---
自从 nvidia 带来了 [PRIME Render Offload](https://download.nvidia.com/XFree86/Linux-x86_64/435.17/README/primerenderoffload.html) 的支持，在 Linux 中安装 nvidia 驱动就变得简单了。

从官网下载适合的驱动，然后在非图形界面中执行即可。

<!--more-->

## 安装
+ 下载驱动 <https://www.nvidia.cn/drivers/lookup/> ，建议选择生产版本
+ 安装依赖 `dkms libglvnd0 libglvnd-dev`
+ stop 图形界面，然后切换到 tty 登录
+ 执行下载驱动，需要特权
  - 启用 dkms
  - 启用签名，一般 dkms 签名的 key 存储于 `/var/lib/dkms/{mok.key, mok.pub}`
+ 将驱动添加到 `initrd` 中，如 deepin 中可编辑 `/etc/modules-load.d/nvidia.conf` 添加内容

``` ini
nvidia
nvidia_modeset
nvidia_uvm
nvidia_drm
```

+ 然后更新 `update-initramfs -u -k all`，重启，登录后执行 `xrandr --listproviders` 查看是否成功，存在 `NVIDIA-GO` 即表示成功
+ 可在程序执行前，添加 `__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia` 变量来指定使用 `nvidia driver`

若有其他问题，可参考 [archlinux wiki - nvidia](https://wiki.archlinux.org/title/NVIDIA)

## Power Management
`nvidia driver` 提供了 [PCI-Express Runtime D3 (RTD3) Power Management](https://download.nvidia.com/XFree86/Linux-x86_64/435.17/README/dynamicpowermanagement.html) 功能，可动态管理功耗。

此特性仅支持笔记本电脑，依赖 ACPI，需要 `4.18` 以上的内核，并启用了 `CONFIG_PM`，另外需要较新的 `GPU` 。

启用此功能的方法如下：

+ 编辑 `/lib/udev/rules.d/80-nvidia-pm.rules` 文件，添加：

``` ini
# Remove NVIDIA USB xHCI Host Controller devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c0330", ATTR{remove}="1"

# Remove NVIDIA USB Type-C UCSI devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c8000", ATTR{remove}="1"

# Remove NVIDIA Audio devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x040300", ATTR{remove}="1"

# Enable runtime PM for NVIDIA VGA/3D controller devices on driver bind
ACTION=="bind", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030000", TEST=="power/control", ATTR{power/control}="auto"
ACTION=="bind", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030200", TEST=="power/control", ATTR{power/control}="auto"

# Disable runtime PM for NVIDIA VGA/3D controller devices on driver unbind
ACTION=="unbind", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030000", TEST=="power/control", ATTR{power/control}="on"
ACTION=="unbind", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030200", TEST=="power/control", ATTR{power/control}="on"
```

+ 编辑 `/etc/modprobe.d/nvidia-pm.conf` 文件，添加：

``` ini
options nvidia "NVreg_DynamicPowerManagement=0x02"
```

然后执行 `update-initramfs -u -k all` ，成功后重启即可。

## 禁用 NVIDIA
开启 nvidia 后会导致功耗增加，为了增加续航，可考虑禁用 nvidia，方法如下：
+ 移除 `/etc/modules-load.d/nvidia.conf`
+ 移除 `/etc/modprobe.d/nvidia-pm.conf`
+ 移除 `/lib/udev/rules.d/80-nvidia-pm.rules`
+ 编辑 `/lib/udev/rules.d/90-nvidia-disable.rules`，添加

``` ini
# Remove NVIDIA USB xHCI Host Controller devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c0330", ATTR{power/control}="auto", ATTR{remove}="1"

# # Remove NVIDIA USB Type-C UCSI devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c8000", ATTR{power/control}="auto", ATTR{remove}="1"

# # Remove NVIDIA Audio devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x040300", ATTR{power/control}="auto", ATTR{remove}="1"

# --------------- Disable NVIDIA ------------
# Remove NVIDIA VGA/3D controller devices
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x03[0-9]*", ATTR{power/control}="auto", ATTR{remove}="1"
```

+ 编辑 `/etc/modprobe.d/nvidia-disable.conf`，添加

``` ini
blacklist nouveau
install nouveau /bin/true
options nouveau modeset=0

blacklist nvidia
install nvidia /bin/true
options nvidia modeset=0
```

然后执行 `update-initramfs -u -k all` ，成功后重启即可。
