# <center>Get my Arch Linux</center>

# 写在前面

- 参考的视频：

  <https://www.bilibili.com/video/av81146687>

  <https://www.bilibili.com/video/av86485933>

- 非常感谢UP主（Github：[@theniceboy](https://github.com/theniceboy)）提供的保姆级教程。

- 这里是我自己安装过程的记录。

- 我在安装时使用的编辑器是`Nano`，如果你使用的是`vim`等编辑器，阅读下面内容时自行替换。

# 安装

## 1. 修改`tty`界面下的字体

- 所有字体都存放在`/usr/share/kbd/consolefonts`目录下。

- 这里将其设置为：

  ```bash
  $ setfont /usr/share/kbd/consolefonts/LatGrkCyr-12*22.psfz.gz
  ```

## 2. 连接网络

### 2.1 扫描当前互联网设备

```bash
$ ip link
```

### 2.2 启用设备

```bash
$ ip link set 设备名 up
```

### 2.3 扫描当前设备下的WiFi列表并得到所有WIFI的名字

```bash
$ iwlist 设备名 scan | grep ESSID
```

### 2.4 使用`wpa_supplicant`连接网络并后台运行

```bash
$ wpa_passphrase 网络名 密码 > internet.conf
$ wpa_supplicant -c internet.conf -i 设备名 &
```

### 2.5 动态分配IP地址

```bash
$ dhcpcd &
```

### 2.6 测试

```bash
$ ping baidu.com
```

## 3. 更正系统时间

```bash
$ timedatectl set-ntp true
```

## 4. 硬盘分区

### 4.1 查看现有的磁盘

```bash
$ fdisk -l
```

### 4.2 进入磁盘编辑

```bash
$ fdisk /dev/sda   # /dev/sda为磁盘设备的位置
$ g   # 清除原有分区并创建一个GPT分区表
$ n   # 创建一个新的分区/dev/sda1 -- 引导分区
      # 接下来选择分区的编号、起始位置、终止位置（分区大小，可用例如“+300M”的形式）
$ n   # 创建一个新的分区/dev/sda3 -- SWAP分区（用于保存内存中的文件以及作为内存的扩展，此
      # 分区不需要太大）
      # 这里我大小设置为1G
$ n   # 创建一个新的分区/dev/sda2 -- 主分区
      # 大小我设置为磁盘的所有剩余空间
$ p   # 查看待写入的分区结果
$ w   # 写入
```

### 4.3 定义分区格式

```bash
$ mkfs.fat -F32 /dev/sda1   # /dev/sda1为引导分区
$ mkfs.ext4 /dev/sda2       # /dev/sda2为主分区
$ mkswap /dev/sda3          # /dev/sda3为SWAP分区
```

### 4.4 打开`SWAP`

```bash
$ swapon /dev/sda3
```

## 5. 配置`pacman`

```bash
$ nano /etc/pacman.conf
```

- 去掉注释：
  
- `Color`
  
- `Arch Linux`软件源列表：`/etc/pacman.d/mirrorlist`

- 寻找中国的服务器，将它移动到`mirrorlist`的最顶上，保存退出。

- 我这里使用的是清华的源：

  ```
  ## China
  Server = http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
  ```

## 6. 使用`pacstrap`安装Arch Linux基础至磁盘中

### 6.1 查看当前的磁盘

```bash
$ fdisk -l
```

### 6.2 挂载磁盘

```bash
$ mount /dev/sda2 /mnt        # 挂载主分区
$ mkdir /mnt/boot             # 创建启动分区在Live CD上的目录
$ mount /dev/sda1 /mnt/boot   # 挂载启动分区
```

### 6.3 开始安装

```bash
$ pacstrap /mnt base linux linux-firmware   # 安装Linux所需的最基础的软件、框架等
```

### 6.4 生成挂载文件

```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
```

## 7. 使用`arch-chroot`对安装好的系统进行配置

### 7.1 进入`arch-chroot`

```bash
$ arch-chroot /mnt   # /mnt为安装好的系统的主分区在Live CD上的挂载位置
```

### 7.2  设置时区和时间

```bash
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime   # 创建链接
$ hwclock --systohc                                         # 同步时间
```

### 7.3 编辑本地化的文件

```bash
$ exit                       # 先退出arch-chroot
$ nano /mnt/etc/locale.gen   # 编辑本地化文件
                             # 去掉“en_US.UTF-8 UTF-8”前面的注释，保存退出         
```

### 7.4 生成本地化

```bash
$ arch-chroot /mnt
$ locale-gen   # 生成本地化
```

### 7.5 设置语言

```bash
$ exit                        # 退出arch-chroot
$ nano /mnt/etc/locale.conf   # 编辑本地化的配置文件
                              # 在其中输入“LANG=en_US.UTF-8”（将系统设置为英文），保存退出。
```

### 7.6 (可选)设置键盘布局和键位绑定

```bash
$ nano /mnt/etc/vconsole.conf
```

### 7.7 编辑网络相关的文件

```bash
$ nano /mnt/etc/hostname   # 编辑主机名称
                           # 我将其设置为niklaus，保存退出
$ nano /mnt/etc/hosts      # 编辑域名与IP地址的对应
                           # 127.0.0.1   localhost
                           # ::1         localhost
                           # 127.0.1.1   niklaus.localdomain   niklaus
```

### 7.8 更改`root`用户密码

```bash
$ arch-chroot /mnt
$ passwd
```

### 7.9 安装`grub`相关

```bash
$ pacman -S grub efibootmgr intel-ucode os-prober
# intel-ucode   厂家更新CPU驱动用，如果是AMD的显卡，则安装amd-ucode
# os-probe      用来寻找电脑中其他操作系统
$ mkdir /boot/grub
$ grub-mkconfig > /boot/grub/grub.cfg                      # 生成grub配置文件
$ uname -m                                                 # 查看系统架构
$ grub-install --target=x86_64-efi --efi-directory=/boot   # 根据自己的系统架构安装grub
```

### 7.10 安装基础工具

```bash
$ pacman -S zsh nano vim wpa_supplicant dhcpcd
# zsh              终端模拟器
# nano             编辑器
# vim              编辑器
# wpa_supplicant   上网工具
# dhcpcd           动态分配IP地址工具
```

### 7.11 重启，完成安装

```bash
$ exit                            # 退出arch-chroot
$ killall wpa_supplicant dhcpcd   # 终止掉网络相关的进程
$ reboot                          # 重启，电脑黑屏后就可以拔掉Live CD了
```

# 安装中出现问题的汇总

## 1. 分区时出现警告：逻辑分区和物理分区不对齐

- 解决方法：使用`shred`命令。

  ```bash
  $ shred -v /dev/sda
  ```

# 初步配置

- 在配置之前记得检查网络连接，确保连上了网。

## 1. 安装`man`

```bash
$ pacman -S man   # 用户手册
```

## 2. 安装`base-devel`

```bash
$ pacman -S base-devel   # sudo、编译器等等的基础工具
```

## 3. 添加用户

```bash
$ useradd -m -G wheel niklaus   # -m        创建家目录
                                # -G        用户所属的组
                                # niklaus   我的用户名
$ passwd niklaus   # 修改密码
```

## 4. 修改用户权限

```bash
$ nano /etc/sudoers   # 编辑sudoer file
                      # 去掉“%wheel ALL=(ALL) ALL”前面的注释，保存退出
```

## 5. 切换到低权限的用户

```bash
$ exit# 退出root用户，并登陆niklaus用户
```

## 6. 安装`Xorg`

```bash
$ sudo pacman -S xorg   # 图形界面的服务器
```

## 7. 安装桌面环境

- 这里我选择的桌面环境是`LXDE`。

- ```bash
  $ sudo pacman -S lxde
  ```

- 若选择最小安装，则为：

  ```bash
  $ sudo pacman -S lxde-common lxsession openbox
  ```

## 8. (可选)安装`lightdm`

- ```bash
  $ sudo pacman -S lightdm   # Display Manager   登陆管理器
  ```

- 配置`lightdm`：

  ```bash
  $ sudo nano /etc/lightdm/lightdm.conf   # 修改皮肤
  # 去掉“greeter-session=example-gtk-gnome”前面的注释
  # 将其改为自己需要使用的登陆界面皮肤
  ```

# 软件安装

## 终端用

### `yay`

- 

### `neofetch`

- 系统硬件信息查看。

-  ```bash
  $ sudo pacman -S neofetch
  ```

### `htop`

- 系统资源占用查看。

- ```bash
  $ sudo pacman -S htop
  ```

## 浏览器

### `chromium`

- 开源、支持多扩展的浏览器。

- ```bash
  $ sudo pacman -S chromium
  ```

## 录屏

### `SimpleScreenRecorder`

- 轻量的录屏软件。

- ```bash
  $ sudo pacman -S simplescreenrecorder
  ```

## 其他

### `gparted`

- 有图形界面的磁盘无损分区工具。

- ```bash
$ sudo pacman -S gparted
  ```
  
  
