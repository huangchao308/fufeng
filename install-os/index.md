# 记一次装机过程

## 起因
老婆使用的电脑系统（Windows 10）坏了，会一直卡在登录页面，登录成功之后又返回登录页面，陷入死循环。

于是，修好电脑的重担就落在了笔者的头上。

笔者脑子一热，直接装了一个 Ubuntu 系统（装机过程省略，网上很多教程），然后又吭哧吭哧地安装了微信和QQ，但是用起来非常不得劲，微信还好，除了UI偏小没啥大问题，但是 QQ 就丑爆了~

迫不得已，只好重新安装 Windows 系统，磨难正式开始~~

## 磨难
### 制作装机 U 盘
以前有过制作装机 U 盘的经历，不过是使用了大白菜（[大白菜官网](https://www.dabaicai.com/)），然而大白菜支持在 Windows 系统上制作装机 U 盘，本人的 Windows 电脑已经坏了，工作电脑是 Mac，所以大白菜就没法使用了。

之前安装 Ubuntu 系统，是使用 [Etcher](https://www.balena.io/etcher/) 制作装机 U 盘。但是制作的 Windows 系统装机 U 盘是无法正常使用的，在尝试从 U 盘启动系统时，也无法正常启动。

然后尝试了使用命令行工具制作装机 U 盘，命令如下：
```bash
diskutil eraseDisk FAT32 "WINDOWS10" MBR disk4
cp -rpv /Volumes/CCHA_X64FREO_ZH-CN_DV9/* /Volumes/WINDOWS10
```
但是在安装的时候提示 install.win 损坏。经过查询，原来是 FAT32 格式的文件系统无法存储单个大于 4GB 的文件，导致 install.win 文件损坏了。

#### 解决办法
* 第一步，分割一下 Windows 镜像，使其能够完整的写入 FAT32 格式的 U 盘中，分割的方式是使用 [Boot Camp ISO Converter](https://twocanoes.com/using-larger-windows-10-isos-with-boot-camp-assistant/)，下载地址如下图所示
![boot camp ISO converter](/fufeng/images/1d8bd35634d742c2ae211d508da82f13.png)
* 第二步，格式化 U 盘
```bash
# disk4 需要替换成 U 盘的盘符，可以使用命令 diskutil list 查看
diskutil eraseDisk FAT32 "WINDOWS10" MBR disk4
```
* 第三步，双击下载的 Windows 镜像，使其在 Mac 上挂载，此时可以通过 diskutil list 查看对应的盘符
* 第四步，通过 CP 命令拷贝镜像到 U 盘中
```bash
cp -rpv /Volumes/CCHA_X64FREO_ZH-CN_DV9/* /Volumes/WINDOWS10
```
### 硬盘分区和格式化
制作完装机 U 盘后，在装机时遇到另一个问题： Windows 系统只能安装在 NTFS 的文件系统中。而在安装 Ubuntu 系统时，电脑硬盘已经格式化成了 EXT3 的文件系统，所以需要重新格式化硬盘。然后就遭遇了各种格式化硬盘的打击：
* 格式化时提示硬盘使用中
* umount 时提示`umount: target is busy`
#### 解决办法
* 第一步，使用 Unetbootin 制作 Gparted 启动盘
Unetbootin 下载地址： [unetbootin](https://unetbootin.github.io/)
Gparted 镜像下载地址： [gparted](https://gparted.org/download.php)
* 第二步，使用第一步制作的 Gparted 启动盘格式化硬盘和分区
这里建议分区后再格式化，这样可以不影响原本的 Ubuntu 系统，也就是可以双系统。具体步骤可以参考 [gparted 教程](https://gparted.org/display-doc.php?name=moving-space-between-partitions)

## References
 [知乎: 2021年mac系统下制作win10引导安装盘，亲测可用](https://zhuanlan.zhihu.com/p/273305963)

 [[详细教程] 在现有Ubuntu系统上安装Windows 10 （双系统）](https://dalewushuang.blog.csdn.net/article/details/90475070)

