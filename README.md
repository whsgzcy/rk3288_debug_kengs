# rk3288 debug kengs / that's so funny super_yu

**一、编译Android源码准备**

**1、外部环境建议**

最好是有一台Windows电脑，因为他可以为所欲为，如果非要在Ubuntu里面安装Windows虚拟机的话，至少惠普电脑貌似是怎么操作都操作不起来的。

为什么非要Windows电脑呢，因为我在Ubuntu下怎么都进不了loader，还是Windows好，工具种类多，在开发上不受限制。

如果是新的九代CPU的话，Windows必须装win10，相信我，没错的。

如果你要在VBox上共享或挂载一个文件夹的时候，出了可以百度出来的步骤，还有就要执行

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
ifneq ($(WITHOUT_HOST_CLANG),false)
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

因为烧写的时候已经是xxx_user_debug的img，所以是有root权限的，但usb root权限和APP root权限不是一回事，如果有需要的话还是得root，

```
参考 https://blog.csdn.net/pcwung/article/details/80213674
```

后面在APP里面执行su的时候，一定还得注意，在mainfest中添加，否则还得报错，但不会有权限的错

```
android:sharedUserId="android.uid.system"
并系统签名 此时你的APP才是真正的system app
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
