# ArchLinux-InstallationGuide
ArchLinux installation guide for partners who want to learn about Arch installation knowledge but do not want to do too many repetitive operations

 USE CHINESE/ENGLISH
https://fastly.mirror.pkgbuild.com/images/
先下载虚拟机文件,如果使用vm的话需要通过qemu转换
安装qemu后在安装文件夹打开然后
qemu-img convert -f qcow2 -O vmdk your_disk.qcow2 your_disk.vmdk (示例)
在我这里执行是
qemu-img convert -f qcow2 -O vmdk E:\VM_ROOT\ROOT_Other\Arch.qcow2 E:\VM_ROOT\ROOT_Other\Arch.vmdk
使用VMware建立一个空的虚拟机,进入你刚刚创建虚拟机的目录，把你转换出来的vmdk文件覆盖上去启动即可

官网这种虚拟机基本上都是最小化安装,编辑器都没有所以你要手动输入配置
先设置root用户方便以后调权限
sudo passwd root
我的密码是root
改完密码su切换用户后你就有权限改文件了,进一下/etc/system/network
里面的文件直接删了或者改个名,那个是dhcp的配置咱们不需要,然后新建个如下文件<img width="778" height="342" alt="image" src="https://github.com/user-attachments/assets/c86aad71-88a2-4673-a494-d12057a8202a" />
这里的文件名参考 数字-你想要的字符+wired.network
接下来在文件里填入你的ip,我这里虚拟机nat网段是192.168.10.0
tee /etc/systemd/network/20-ybywired.network > /dev/null <<'EOF'
[Match]
Name=eth0
[Network]
Address=192.168.10.192/24
Gateway=192.168.10.2
DNS=223.5.5.5
DNS=114.114.114.114
DNS=8.8.8.8
EOF
接下来重启一下网络,应用配置
systemctl restart systemd-networkd
systemctl restart systemd-resolved
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
改完ping就行,一般来说是会通

为了我们不被文件编辑折磨,现在我们要安装编辑器
安装编辑器之前需要做的
sudo pacman-key --init
sudo pacman-key --populate archlinux
sudo pacman -Sy archlinux-keyring
sudo pacman -S vim nano

ssh连接
sudo nano /etc/ssh/sshd_config
PasswordAuthentication yes
PermitRootLogin yes
sudo systemctl restart sshd

有网络了还说啥,换源呗
先进https://archlinux.org/mirrors/挑几个国内源,然后编辑 /etc/pacman.d/mirrorlist 文件，把新地址按格式写入其中
页面中 Available URLs 下有两个镜像地址，分别是 http 和 https 协议的。推荐复制 https 的地址，然后用编辑器打开 /etc/pacman.d/mirrorlist 文件，按以下格式粘贴并编辑镜像地址：
Server = https://mirrors.jxust.edu.cn/archlinux/$repo/os/$arch
注意只需要把镜像地址放到 Server = 和 $repo/os/$arch 之间就可以了。保存后用 pacman -Syu 命令更新一下本地软件库。

使用命令更换#
用 reflector 命令设置镜像源
Arch Linux 在安装时提供了这个命令，但在安装好的系统中并没有它，所以需要先安装：
sudo pacman -S reflector
安装好后就可以用这个命令来更换镜像源了。直接通过命令选项指定国家，协议和数量：
sudo reflector \
    --country China \
    --protocol https \
    --latest 3 \
    --save /etc/pacman.d/mirrorlist
上面这个命令会查询国内支持 https 协议的镜像源，并且是最近刚从官方同步过的 3 个地址，然后保存到 /etc/pacman.d/mirrorlist 镜像配置文件。等命令执行完，镜像源就更换成功了。

根据在安装arch时遇到的问题来浅谈ssh下的vim
在Xshell ssh连接会出现右键粘贴出现变 visual 的问题,这时候需要我们按住shift再右键或者shift+右键两下，出来菜单然后粘贴

Arch桌面
我这边装的是Gnome桌面,首先更新软件包数据库
sudo pacman -Syy
然后重新完整升级系统
sudo pacman -Syu
清理一下缓存
sudo pacman -Scc
装一下linux后端服务器
sudo pacman -S xorg
装一下桌面本体
sudo pacman -S gnome
开启登录管理器
systemctl enable gdm
然后重启即可

汉化Arch桌面
打开sudo vim /etc/locale.gen文件

把zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
去除注释符

sudo locale-gen

修改一下 /etc/locale.conf
你可以参照我的
echo "LANG=zh_CN.UTF-8" | sudo tee /etc/locale.conf

接下来安装中文字体
sudo pacman -S noto-fonts-cjk

sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-rime
echo -e "GTK_IM_MODULE=fcitx\nQT_IM_MODULE=fcitx\nXMODIFIERS=@im=fcitx" | sudo tee -a /etc/environment

搞定了就可以重启了
