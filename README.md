# rk3288 debug kengs / that's so funny super_yu

**如果你觉得这个对你有帮助，请加我微信whsgzcy，我给你发5块钱红包**

**一、编译Android源码准备**

**1、外部环境建议**

最好是有一台Windows电脑，因为他可以为所欲为，如果非要在Ubuntu里面安装Windows虚拟机的话，至少惠普电脑貌似是怎么操作都操作不起来的。

为什么非要Windows电脑呢，因为我在Ubuntu下怎么都进不了loader，还是Windows好，工具种类多，在开发上不受限制。

如果是新的九代CPU的话，Windows必须装win10，相信我，没错的。

如果你要在VBox上共享或挂载一个文件夹的时候，除了可以百度出来的步骤，还有就要执行

```
//也是，网上一搜真特么多，好标准的坑
sudo usermod -a -G vboxsf darren（你的用户名称）

eg

sudo usermod -a -G vboxsf whsgzcy

后

reboot
```

**2、编译源码**

首先必须install的有，我的环境是Ubuntu16.04 rk3288 5.1

```
sudo apt-get install lzop
sudo apt-get install libc6:i386
sudo ln -s /usr/lib/jvm/jdk1.7.0_76/bin/javap(java javah javadoc javac) /bin/javap(java javah javadoc javac)
sudo apt-get install bison
sudo apt-get install flex
sudo apt-get install libncurses5-dev
sudo apt-get install gperf
sudo apt-get install lib32stdc++6
sudo apt-get  install libxml2-utils
sudo apt-get install libswitch-perl
sudo apt-get install g++-multilib
sudo apt-get install lib32z1
```

sudo make update-api

```
如果出现了

clang: error: linker command failed with exit code 1 (use -v to see invocation)
build/core/host_shared_library_internal.mk:44: recipe for target 'out/host/linux-x86/obj32/lib/libnativehelper.so' failed

你需要

cd  <source_android>/art/build/
vim Android.common_build.mk    //修改第119行
修改前：
# Host.
ART_HOST_CLANG := false
ifneq ($(WITHOUT_HOST_CLANG),true)
# By default, host builds use clang for better warnings.
ART_HOST_CLANG := true
endif
修改后：
# Host.
ART_HOST_CLANG := false

特别注意！！！ 下述的一定得为true 否则还会抛出上述错误，有些博客特么的自己都没调试就写上去，我已经看到了两个版本
ifneq ($(WITHOUT_HOST_CLANG),true)
# By default, host builds use clang for better warnings.
ART_HOST_CLANG := true
endif

编译还是报同样的错误
sudo cp /usr/bin/ld.gold   <source_android>/prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.11-4.6/x86_64-linux/bin/ld

最好修改之后都要

sudo make update-api

————————————————
版权声明：本文为CSDN博主「HelloBirthday」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014770862/article/details/52624851
```

建议还是用官方提供的脚本编译，不过在编译之前一定得先编译kernel

```
编译u-boot
编译kernel

cd ~/proj/firefly-rk3288-lollipop/kernel
make firefly_defconfig
make -j8 firefly-rk3288.img
```

如果遇到诸如下述问题：

![please clone](https://raw.githubusercontent.com/whsgzcy/rk3288_debug_kengs/master/images/a.jpg)

![please clone](https://raw.githubusercontent.com/whsgzcy/rk3288_debug_kengs/master/images/b.png)

你需要去拉一下代码，我试过，真特么的慢，白天你就别想了，晚上12点左右的时候再pull，一觉醒来就好了，或直接到git上把rk3288分支上

```
arch/arm/configs/
直接把 firefly_defconfig 拉下来，再去执行脚本编译
```

上述问题解决完之后，就可以编译源码了，

```
./FFTools/make.sh -d firefly-rk3288 -j8 -l rk3288_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3288_box-userdebug
http://wiki.t-firefly.com/zh_CN/Firefly-RK3288/compile_android.html
```

**二、APP Root**

注意了，又是一个巨坑

看我的提交记录这块也能看出来，也是经过多次修改的，因为根据我的业务的变化，之前认为的东西可能不是一成不变的；

因为烧写的时候已经是xxx_user_debug的img，可执行su，所以是有root权限的，但usb root权限和APP root权限不是一回事，如果有需要的话还是得root；

至少的我的过程是

```
仅供参考

1、此链接第二标题
别忘了
android:sharedUserId="android.uid.system"
https://blog.csdn.net/qq_33750826/article/details/80848102

2、在packages/app/settings找到setting的源码，可以看出需要去修改
android.os.SystemProperties中的su_root

3、这样你的app才有了最高的权限，实验在/system/etc下touch个文件删除文件，用java代码，执行su

```

**三、GPIO调试**

```
cat /sys/kernel/debug/gpio

·申请gpio口，设置输出高电平

echo 237 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio237/direction
echo 1 > /sys/class/gpio/gpio237/value

echo in > /sys/class/gpio/gpio237/direction

cat /sys/class/gpio/gpio237/value
```

**四、设备树**

其实这个我煎熬了一段时间，我试图想把他说清楚，说的不清不楚也行，但程序员优良淳朴的天性告诉我，不行，所以，我想就我目前能触达到的地方，整理出来，以希望有大佬给我点拨

Tortoise on the way

kernel源码

https://github.com/FireflyTeam/kernel/tree/rk3328/firefly

按照这个教程去走（Linux）

http://wiki.t-firefly.com/zh_CN/Firefly-RK3288/linux_compile.html

编译配置文件

```
device/rockchip/rk3288/firefly-rk3288.mk

# Kernel dts rk3288-firefly-aio rk3288-firefly-vga
export RK_KERNEL_DTS=rk3288-firefly-aio
```
./build.sh firefly-rk3288.mk

将必要的文件都建立link

以此为切入点

从脚本得知

```
#=========================
# build target
#=========================
...
if [ ! -n "$1" ];then
	echo "build all and save all as default"
	BUILD_TARGET=allsave
else
	BUILD_TARGET=$1
   	NEW_BOARD_CONFIG=$(find $CFG_DIR -name "$1")
fi
...
echo $NEW_BOARD_CONFIG
rm -f $BOARD_CONFIG
ln -s $NEW_BOARD_CONFIG $BOARD_CONFIG
unset RK_PACKAGE_FILE
unset RK_MKUPDATE_FILE
source $NEW_BOARD_CONFIG
```

按照参数 干的哪些事？

```
将 device/rockchip/rk3288/firefly-rk3288.mk link 到 device/rockchip/.BoardConfig.mk
删除 tools/linux/Linux_Pack_Firmware/rockdev/package-file
将 rk3288-ubuntu-package-file link 到 tools/linux/Linux_Pack_Firmware/rockdev/package-file
将 rk3288-mkupdate.sh link mkupdate.sh
```

```
./build.sh uboot
./make.sh firefly-rk3288        ---- build for firefly-rk3288_defconfig

u-boot/make.sh
make CROSS_COMPILE=${TOOLCHAIN_GCC}  all --jobs=${JOB} ${OUTOPT}
```

主要的输出结果

```
...
load addr is 0x0!
pack input ./u-boot.bin
pack file size: 505481
crc = 0x691e4eee
pack uboot.img success!
pack uboot okay! Input: ./u-boot.bin
out:rk3288_loader_v1.06.236.bin
fix opt:rk3288_loader_v1.06.236.bin
merge success(rk3288_loader_v1.06.236.bin)
pack loader okay! Input: /home/whsgzcy/Documents/linux-sdk-c/rkbin/RKBOOT/RK3288MINIALL.ini
Image Type:   Rockchip RK32 (SD/MMC) boot image
Data Size:    14336 bytes
/home/whsgzcy/Documents/linux-sdk-c/u-boot

load addr is 0x8400000!
pack input /home/whsgzcy/Documents/linux-sdk-c/rkbin/./bin/rk32/rk3288_tee_ta_v1.35.bin
pack file size: 734044
crc = 0x13e06865
pack ./trust.img success!
trust.img with ta is ready
pack trust okay! Input: /home/whsgzcy/Documents/linux-sdk-c/rkbin/RKTRUST/RK3288TOS.ini

Platform RK3288 is build OK, with new .config(make firefly-rk3288_defconfig)
/home/whsgzcy/Documents/linux-sdk-c
====Build uboot ok!====
...
```

这里我理解就是编译出必要的文件

firefly-rk3288_defconfig

./build.sh Kernel

```
...
function build_kernel(){
	# build kernel
	echo "============Start build kernel============"
	echo "TARGET_ARCH          =$RK_ARCH"
	echo "TARGET_KERNEL_CONFIG =$RK_KERNEL_DEFCONFIG"
	echo "TARGET_KERNEL_DTS    =$RK_KERNEL_DTS"
	echo "=========================================="
	cd $TOP_DIR/kernel && make ARCH=$RK_ARCH $RK_KERNEL_DEFCONFIG && make ARCH=$RK_ARCH $RK_KERNEL_DTS.img -j$RK_JOBS && cd -
	if [ $? -eq 0 ]; then
		echo "====Build kernel ok!===="
	else
		echo "====Build kernel failed!===="
		exit 1
	fi
}
...
```

```
============Start build kernel============
TARGET_ARCH          =arm
TARGET_KERNEL_CONFIG =firefly_linux_defconfig
TARGET_KERNEL_DTS    =rk3288-firefly-aio
==========================================
```

**编译kernel是大头，从脚本的意思就是前面步骤是为kernel服务的，也包含 编译uboot**

**从一个需求切入**

需求：把底下的摄像头打开

camera.png

![please clone](https://raw.githubusercontent.com/whsgzcy/rk3288_debug_kengs/master/images/camera.png)

我比较了Firefly的提交记录，前后版本的比较，得出 修改的地方有

```
arch\arm\boot\dts\rk3288-firefly.dts
...
&vcc_mipi {
	status = "okay";
	gpio = <&gpio0 11 GPIO_ACTIVE_HIGH>;
};

&dvdd_1v2 {
	status = "okay";
};

&ov13850 {
	status = "okay";
	reset-gpios = <&gpio2 15 GPIO_ACTIVE_HIGH>;
};

&mipi_pwr {
	rockchip,pins = <0 11 RK_FUNC_GPIO &pcfg_pull_down>;
};

&mipi_phy_tx1rx1 {
	status = "okay";
};

&isp {
	status = "okay";
};

&isp_mmu {
	status = "okay";
};
...
```

因为之前尝试过 想把一整个组织起来，发现力气总使不出来，而且代码量还是挺多的，目前先以&ov13850为切入点吧，

&ov13850

gpio0 11 设置高电平是可以起作用的

```
arch\arm\boot\dts\rk3288-firefly.dts
/dts-v1/;
#include "rk3288-firefly-port.dtsi"
#include "rk3288-linux.dtsi"
&ov13850

arch\arm\boot\dts\rk3288-firefly-port.dtsi
#include "rk3288.dtsi"
#include "rk3288-rkisp1.dtsi"
#include <dt-bindings/pwm/pwm.h>
#include <dt-bindings/input/input.h>
...
&i2c3 {
	status = "okay";

	ov13850: ov13850@10 {
		status = "disabled";
		compatible = "ovti,ov13850";
		reg = <0x10>;
		clocks = <&cru SCLK_VIP_OUT>;
		clock-names = "xvclk";

		reset-gpios = <&gpio3 8 GPIO_ACTIVE_HIGH>;
		pwdn-gpios = <&gpio2 14 GPIO_ACTIVE_HIGH>;
		avdd-supply = <&vcc_mipi>; /* VCC28_MIPI */
		dovdd-supply = <&vcc_mipi>; /* VCC18_MIPI */
		dvdd-supply = <&dvdd_1v2>; /* DVDD_1V2 */

		port {
			ucam_out0: endpoint {
				remote-endpoint = <&mipi_in_ucam0>;
				data-lanes = <1 2>;
			};
		};
	};

	ov13850_1: ov13850@20 {
		status = "disabled";
		compatible = "ovti,ov13850";
		reg = <0x20>;
		clocks = <&cru SCLK_VIP_OUT>;
		clock-names = "xvclk";

		reset-gpios = <&gpio3 8 GPIO_ACTIVE_HIGH>;
		pwdn-gpios = <&gpio2 13 GPIO_ACTIVE_HIGH>;
		avdd-supply = <&vcc_mipi>; /* VCC28_MIPI */
		dovdd-supply = <&vcc_mipi>; /* VCC18_MIPI */
		dvdd-supply = <&dvdd_1v2>; /* DVDD_1V2 */

		port {
			ucam_out1: endpoint {
				remote-endpoint = <&mipi_in_ucam1>;
				data-lanes = <1 2>;
			};
		};
	};
};
...
```

有点不明白的是，在rk3288-firefly.dts里，&ov13850是独立的，和&vcc_mipi一样，在rk3288-firefly-port.dtsi里面也应该有个&ov13850，貌似语法这样才能对的上，为什么在
这里要写在&i2c3里面，谁可以解释一下

```
arch\arm\boot\dts\rk3288.dtsi
#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/interrupt-controller/irq.h>
#include <dt-bindings/interrupt-controller/arm-gic.h>
#include <dt-bindings/pinctrl/rockchip.h>
#include <dt-bindings/clock/rk3288-cru.h>
#include <dt-bindings/thermal/thermal.h>
#include <dt-bindings/power/rk3288-power.h>
#include <dt-bindings/soc/rockchip,boot-mode.h>
#include <dt-bindings/suspend/rockchip-rk3288.h>
#include <dt-bindings/display/drm_mipi_dsi.h>
#include "skeleton64.dtsi"
...
aliases {
	ethernet0 = &gmac;
	i2c0 = &i2c0;
	i2c1 = &i2c1;
	i2c2 = &i2c2;
	i2c3 = &i2c3;
	i2c4 = &i2c4;
	i2c5 = &i2c5;
	mshc0 = &emmc;
	mshc1 = &sdmmc;
	mshc2 = &sdio0;
	mshc3 = &sdio1;
	serial0 = &uart0;
	serial1 = &uart1;
	serial2 = &uart2;
	serial3 = &uart3;
	serial4 = &uart4;
	spi0 = &spi0;
	spi1 = &spi1;
	spi2 = &spi2;
	dsi0 = &dsi0;
	dsi1 = &dsi1;
};
...
```

**知识点罗列 《Linux设备驱动开发详解 4.0 宋宝华》**

arch/arm/plat-xxx & arch/arm/match-xxx都是垃圾

定义

设备树：基本就是一棵电路板上cpu、总线、设备组成的树，BootLoader会将这棵树传递给内核，然后内核可以识别这棵树，并根据它展开的Linux内核中的platform_device、i2c_client、spi_device等设备，而这些设备用到的内存、IRQ等资源，也被传递给了内核，内核会将这些资源帮顶给展开的相应的设备

绑定

对于设备树中的结点和属性一般需要文档来进行讲解，文档后缀名一般为.txt，

Documentation/devicetree/bindings/i2c/i2c-xiic.txt 描述了Xilinx的I2C控制器
一些属性直接在这里copy

Linux内核下的scripts/checkpatch.pl会运行一个检查，如果有人在设备树中新添加了compatible字符串，而没有添加相应的文档解释，程序会报出警告

Bootloader

为了使能设备树，需要在编译Uboot的时候再config中加入

#define CONFIG_OF_LIBFDT

在Uboot中，可以从NAND、SD或者TFTP等任意介质中将.dtb读入内存

**18.2.2 根节点兼容性**

首先 根节点 "/"的兼容属性
compatible="acme,coyotes-revenge"，

定义了整个系统(设备级别)的名称，他的组织形式为
< manufacturer>,< model>

Linux内核通过根节点的兼容属性即可判断他启动的是什么设备，在真实项目中，这个顶层的设备的兼容属性一般包括两个或者两个以上的兼容性字符串，首个兼容性字符串是板子的名字，后面一个兼容性是芯片级别或者芯片系列级别的名字

```
kernel/arch/arm/boot/dts/rk3288-firefly-vga.dts
compatible = "firefly,rk3288-firefly", "rockchip,rk3288";
```

在这里 firefly,rk3288-firefly 和 rockchip,rk3288 都是可以的

这些不同的设备会有不同的MACHINED_ID，Uboot启动Linux内核会将MACHINED_ID存放在rl寄存器，Linux启动时会匹配BootLoader传递的MACHINED_ID和MACHINED_START，然后执行相应设备的一系列初始化函数。

在引入设备树之后，MACHINED_START，变更为DT_MACHINED_START，其中一个包含.dt_compat成员，用来表明相关的设备与dts根节点兼容属性兼容关系，从而也是执行相应设备的一系列初始化函数

```
int of_machine_is_compatible(const char *compat)
include/linux/rockchip/cpu.h:67:	       of_machine_is_compatible("rockchip,rk3128")
drivers/usb/dwc_otg_310/usbdev_bc.c:42:	if (of_machine_is_compatible("rockchip,rk3188"))
```

所以，每个设备每个根的代码里面都会有这样的判断代码

**18.2.3 设备节点兼容性**

设备节点和根节点的兼容性是类似的，都是从具体到抽象

需要时常的读一下

```
Documentation/devicetree/bindings/xxxx
```

对于I2C、SPI还有一点提醒的是，I2C和SPI外设驱动和设备树种设备节点的兼容性属性还有一种弱式匹配方法
如果别名出现在设备spi_driver的id_table里面，或者别名与spi_driver的name字段相同，SPI设备与驱动都可以匹配上
通过别名匹配，实际上SPI和I2C的外设驱动即使没有of_match_table，还是可以和设备树中的节点匹配上

```
static int spi_match_device(struct device *dev, struct device_driver *drv)
{
	const struct spi_device	*spi = to_spi_device(dev);
	const struct spi_driver	*sdrv = to_spi_driver(drv);

	/* Attempt an OF style match */
	if (of_driver_match_device(dev, drv))
		return 1;

	/* Then try ACPI */
	if (acpi_driver_match_device(dev, drv))
		return 1;

	if (sdrv->id_table)
		return !!spi_match_id(sdrv->id_table, spi);

	return strcmp(spi->modalias, drv->name) == 0;
}
```

当一个驱动支持两个或多个设备的时候，这些不同的dts中的设备兼容属性都会写入驱动OF匹配表，因此驱动可以通过BootLoader传递给内核设备树中真正节点的兼容性以确定研究是哪一种设备
