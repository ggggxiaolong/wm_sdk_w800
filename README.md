**[原文](https://github.com/IOsetting/wm-sdk-w806/blob/main/README.cn.md)**
- 增加usb权限处理方式

# 关于 

[联盛德MCU W801 开发套件](https://www.winnermicro.com/html/1/156/158/558.html)

## 文件结构

```
wm-sdk-w806
├─app              # 用户应用代码
├─bin              # 编译中间及结果产物
├─demo             # 功能演示代码
├─include          # SDK头文件
├─ld               # 链接脚本
├─lib              # 库文件
├─Makefile
├─platform         # SDK源代码
└─tools            # 编译脚本和工具
```

# Linux环境说明

## 下载

* https://occ.t-head.cn/community/download 下载编译工具
* 导航->工具->工具链-800系列->(当前是V3.10.29)
* 根据自己的操作系统, 下载对应版本, 对于Ubuntu20.04, 下载 csky-elfabiv2-tools-x86_64-minilibc-yyyymmdd.tar
* 其它下载方式
   * 百度盘下载 https://pan.baidu.com/s/1Mp-oHNM3k4Hb8vEybv8pZg code:vw42
   * http://82.157.145.101/download/toolkits/winnermicro/w806/
   * AUR https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=csky-toolchain-bin (在source中找到下载地址)

## 安装

上面下载的tar.gz文件外层路径用的是`./`目录, 建议放到一个单独的目录中解压, 或者指定解压目录解压目录, 参考命令如下
```bash
mkdir csky-elfabiv2-tools-x86_64-minilibc-20210423
tar xvf csky-elfabiv2-tools-x86_64-minilibc-20210423.tar.gz  -C csky-elfabiv2-tools-x86_64-minilibc-20210423/
```
移动到/opt下, 目录可以自己定, 设置权限禁止普通用户修改
```bash
cd /opt/toolchains/
sudo mv ~/Download/csky-elfabiv2-tools-x86_64-minilibc-20210423/ .
sudo chown -R root:root csky-elfabiv2-tools-x86_64-minilibc-20210423/
```
不需要添加系统路径

## 编译

导出此项目
```bash
git clone https://github.com/ggggxiaolong/wm_sdk_w800.git
```
运行menuconfig, 配置工具路径
```bash
cd wm-sdk-w806
make menuconfig
```
在menuconfig界面中, Toolchain Configuration -> 第二个toolchain path, 将刚才的路径填进去, 需要完整路径, 带最后的斜杆, 例如
```
/opt/toolchains/csky-elfabiv2-tools-x86_64-minilibc-20210423/bin/
```
其他不用动, Save后退出menuconfig

执行编译
```bash
make
```

### 更多编译选项

* 串口0打印`printf()`输出  
  在 /include/arch/xt804/csi_config.h 中通过`USE_UART0_PRINT`控制是否使用串口0打印`printf()`输出, 默认开启. 注意: 此功能会占用串口0, 如果需要使用串口0与其他设备通信, 请关闭此选项

* 免按键自动下载  
  在 /include/arch/xt804/csi_config.h 中通过`USE_UART0_AUTO_DL`控制是否开启免按键自动下载, 默认关闭. 需要先开启`USE_UART0_PRINT`才能开启此选项, 如果关闭`USE_UART0_PRINT`则此选项关闭.

## 写入开发板

首先通过`dmesg`,`lsusb`, `ls /dev/tty*`等命令确定自己开发板在系统中对应的USB端口, 例如`ttyUSB0`.  

运行menuconfig, 配置端口名称
```bash
cd wm-sdk-w806
make menuconfig
```
在menuconfig界面中, Download Configuration -> download port, 填入开发板在你的系统中对应的USB端口, 例如`ttyUSB0`, 注意这里只需要填纯端口名, 不需要用完整的路径. 可以调高波特率加快下载, 只支持`115200`, `460800`, `921600`, `1000000`, `2000000`, Save后退出menuconfig

执行烧录
```bash
make flash
```
根据输出的提示, 按一下reset键就会开始下载. 如果前一次写入的固件已经开启了`USE_UART0_AUTO_DL`则不需要按键, 会自动开始下载
```
enerate compressed image completed.
build finished!
connecting serial...
serial connected.
wait serial sync.........         <----- 这里按下reset
please manually reset the device. <----- 或者这里
....
serial sync sucess.
mac CC-CC-CC-CC-CC-CC.
start download.
0% [###] 100%
download completed.
reset command has been sent.
```
下载完成后, 下载工具会发送复位指令, 复位成功后程序会自动开始执行.

### 更多下载选项

显示串口列表
```bash
make list
```
烧录并打开串口监视器 
```bash
make run
```
只打开串口监视器 
```bash
make monitor
```

### 处理USB权限问题

make run 遇到这样的错误

```shell
generate normal image completed.
generate normal image completed.
compress binary completed.
generate compressed image completed.
build finished!
connecting serial...
can not open serial
can not open serial
make: *** [tools/w800/rules.mk:150：run] 错误 255
```

查看串口的VendorID和ProductId
```shell
➜  wm_sdk_w800_20211203 lsusb

Bus 001 Device 024: ID 1a86:7523 QinHeng Electronics CH340 serial converter
```

创建文件 /etc/udev/rules.d/70-ttyusb.rules
文件中的 VendorID 和 ProductId 替换为上面获取的Id

```
KERNEL=="ttyUSB[0-9]*", MODE="0666"
SUBSYSTEM=="usb", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", MODE="0666"
SUBSYSTEM=="usb_device", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", MODE="0666"
```
重新插拔usb