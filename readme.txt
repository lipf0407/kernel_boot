=================================================================================================
镜像文件相关说明：
Bootloader:
|                  \   
├── fwbl1 ---- |
├── bl2.bin ------ | => ubootpak.bin
├── u-boot.bin --- |
└── tzsw ----- /

Kernel:
|                  \
├── ramdisk.img -- | => boot.img --- \
└── zImage ------- /                 |
                                     | => update.zip
Filesystem:                          |
└── system.img --------------------- /

Tools:
├── exynos4x12-irom-sd.sh -- For burnning ubootpak.bin to external booting SDCARD
└── sd_fusing.sh ----------- For burnning fwbl1 bl2.bin u-boot.bin tzsw to external booting SDCARD

=================================================================================================
fastboot手动更新方式:
1、系统上电，在串口终端中按空格键进入uboot命令行
2、如果系统INAND未执行过任何分区命令，则键入命令：fdisk -c 0
3、确认USB线已正确连接至PC端，在uboot的串口终端中键入命令：fastboot 以启动传输服务
4、在PC端的命令行下，按需键入如下命令:
BOOTLOADER:
	fastboot flash ubootpak ubootpak.bin
	或者
	fastboot flash fwbl1 fwbl1
	fastboot flash bl2 bl2.bin
	fastboot flash bootloader u-boot.bin
	fastboot flash tzsw tzsw

BOOT IMAGE:
	fastboot flash boot boot.img
	或者
	fastboot flash kernel zImage
	fastboot flash ramdisk ramdisk-uboot.img
	
FILESYSTEM:
	fastboot flash system system.img
	擦除data cache及fat分区：
	fastboot -w
	fastboot erase fat

UPDATE：
	此命令可直接烧写BOOT IMAGE及FILESYSTEM
	fastboot update update.zip

备注：如果需要手动更新QT，bootloader及内核更新方法同上，仅仅需要将system分区对应的烧写文件改为qt-rootfs.img即可。
在更换文件系统后，请确认环境变量的设置是否正确，具体设置见下文说明。

=================================================================================================
SD卡更新方式:

Android系统：
1、准备一张SD卡，在其根目录下建立x4412-android文件夹
2、拷贝镜像文件：ubootpak.bin boot.img system.img至x4412-android目录下
3、并在x4412-android目录下，创建环境变量默认配置文件env.txt,不存在，则不更新系统环境变量
4、env.txt文件内容配置示例:(需正确设置环境变量,在文件末尾需要保留一空行)
	bootcmd=movi read kernel 0 40008000;movi read rootfs 0 41000000 400000;bootm 40008000 41000000
	bootargs=lcd=vga-1024x768 tp=ft5x06-1024x600 cam=ov2655 mac=00:09:c0:ff:ee:58
5、系统上电，系统自动检测是否需要升级，等待即可

QT系统：
1、准备一张SD卡，在其根目录下建立x4412-qt文件夹
2、拷贝镜像文件：ubootpak.bin zImage qt-rootfs.img至x4412-qt目录下
3、并在x4412-qt目录下，创建环境变量默认配置文件env.txt,不存在，则不更新系统环境变量
4、env.txt文件内容配置示例:(需正确设置环境变量,在文件末尾需要保留一空行)
	bootcmd=movi read kernel 0 40008000;bootm 40008000
	bootargs=root=/dev/mmcblk0p2 rw rootfstype=ext4 lcd=vga-1024x768 tp=ft5x06-1024x600 cam=ov2655 mac=00:09:c0:ff:ee:58
5、系统上电，系统自动检测是否需要升级，等待即可

UBUNTU系统:
1、准备一张SD卡，在其根目录下建立x4412-ubuntu文件夹
2、拷贝镜像文件：ubootpak.bin zImage ubuntu-rootfs.tar.bz2 burn-ubuntu-ramdisk.img至x4412-ubuntu目录下
3、并在x4412-ubuntu目录下，创建环境变量默认配置文件env.txt,不存在，则不更新系统环境变量
4、env.txt文件内容配置示例:(需正确设置环境变量,在文件末尾需要保留一空行)
	bootcmd=movi read kernel 0 40008000;bootm 40008000
	bootargs=root=/dev/mmcblk0p1 rw rootfstype=ext4 lcd=vga-1024x768 tp=ft5x06-1024x600 cam=ov2655 mac=00:09:c0:ff:ee:58
5、系统上电，系统自动检测是否需要升级，等待即可

备注：如需强制更新，按着Left键启动或者将环境变量恢复为默认设置，则会触发强制升级
env default -f
env save
注意，自动检测是否升级是依据文件的crc来实现的，为避免较长的计算时间，最大读取单个升级文件的前5MB并计算CRC。比如system.img这类镜像文件，如果前5MB没变化，且bootloader和内核都没改变，则不会触发自动更新。为解决此类情况，可执行强制更新模式。

=================================================================================================
制作量产启动SDCARD
1、准备一张SD卡，通过gparted工具将前面保留100多MB空间，后面格式化为FAT32分区
2、运行脚本exynos4x12-irom-sd.sh或者sd_fusing.sh制作启动卡
UBOOT四合一镜像ubootpak.bin烧写命令：
	sudo ./exynos4x12-irom-sd.sh /dev/sdb ubootpak.bin
UBOOT分离镜像fwbl1 bl2.bin u-boot.bin tzsw烧写命令：
	sudo ./sd_fusing.sh /dev/sdb
3、拷贝相应的升级文件至SD卡，方法同上。

====================================================================================
UBOOT环境变量参数说明：
1)设置网卡MAC地址为
env set bootargs "mac=00:09:c0:ff:ee:58"
env save

2)Camera模块,如果没有传递任何"cam=xxx"参数，则默认使能OV2655模块
cam变量可选参数列表如下：
	ov2655
	tvp5150
	tvp5146
示例，选择tvp5150 TVIN模块
env set bootargs "cam=tvp5150"
env save

3)选择LCD液晶屏
lcd变量可选择参数列表如下：
	ek070tn93 		(800 X 480)
	vs070cxn		(1024 X 600)
	vga-1024x768	(1024 X 768)
	vga-1440x900	(1440 X 900)
	vga-1280x1024	(1280 X 1024)
示例，选择EK070TN93标清屏(800 X 480)
env set bootargs "lcd=ek070tn93"
env save

4)选择触摸屏分辨率
tp变量可选择参数列表如下：
	ft5x06-800x480
	ft5x06-1024x600
	gslx680
示例，选择ft5x06-1024x600
env set bootargs "tp=ft5x06-1024x600"
env save

5)示例参数：
标清屏：
env set bootargs "lcd=ek070tn93 tp=ft5x06-800x480 cam=ov2655 mac=00:09:c0:ff:ee:58"
env save

高清屏:
env set bootargs "lcd=vs070cxn tp=ft5x06-1024x600 cam=ov2655 mac=00:09:c0:ff:ee:58"
env save

高清屏－思利微电容触摸:
env set bootargs "lcd=vs070cxn tp=gslx680 cam=ov2655 mac=00:09:c0:ff:ee:58"
env save

VGA-1024x768:
env set bootargs "lcd=vga-1024x768 tp=ft5x06-1024x600 cam=ov2655 mac=00:09:c0:ff:ee:58"
env save

VGA-1440x900:
env set bootargs "lcd=vga-1440x900 tp=ft5x06-1024x600 cam=ov2655 mac=00:09:c0:ff:ee:58"
env save

VGA-1280x1024:
env set bootargs "lcd=vga-1280x1024 tp=ft5x06-1024x600 cam=ov2655 mac=00:09:c0:ff:ee:58"
env save

=================================================================================================
其他：
1) 破坏INAND引导，操作如下：
	系统上电，按空格键进入uboot命令行，执行命令mmc erase boot 0 1 1 则INAND启动已破坏，可以从外部SDCARD启动

2) system分区以读写方式挂载：
	mount -t ext4 -o remount,rw /dev/block/mmcblk0p2 /system

3) 不烧写INAND，调试kernel及ramdisk
	fastboot boot zImage ramdisk-uboot.img
	启动DEBUG版ramdisk则使用如下命令：
	fastboot boot zImage debug-ramdisk-uboot.img

4) 切换INAND的boot及user模式
	emmc open 0
	emmc close 0

5) 烧写QT文件系统
	fastboot flash system qt-rootfs.img
	设置uboot启动参数:
	env set bootargs "root=/dev/mmcblk0p2 rw rootfstype=ext4 lcd=vs070cxn tp=ft5x06-1024x600 cam=ov2655 mac=00:09:c0:ff:ee:58"
	env set bootcmd "movi read kernel 0 40008000;bootm 40008000"
	env save

6)　烧写lubuntu系统至外部SDCARD
	制作一张sdcard并将其格式化为EXT4分区，在PC平台上执行解压到sdcard命令:
	tar xvf lubuntu-12.04-x4412-roofs.tar.bz2 -C /xxxxx/sdcard
	设置uboot启动参数:
	env set bootargs "root=/dev/mmcblk1p1 rw rootfstype=ext4 lcd=vs070cxn tp=ft5x06-1024x600 cam=ov2655 mac=00:09:c0:ff:ee:58"
	env set bootcmd "movi read kernel 0 40008000;bootm 40008000"
	env save

7) 恢复uboot默认配置参数
   在uboot的命令行里执行如下指令,即可
   env default -f
   env save

