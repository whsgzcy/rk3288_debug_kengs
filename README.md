# rk3288 debug kengs / that's so funny super_yu

**一、编译Android源码准备**

**1、外部环境建议**

最好是有一台Windows电脑，因为他可以为所欲为，如果非要在Ubuntu里面安装Windows虚拟机的话，至少惠普电脑貌似是怎么操作都操作不起来的。

为什么非要Windows电脑呢，因为我在Ubuntu下怎么都进不了loader，还是Windows好，工具种类多，在开发上不受限制。

如果是新的九代CPU的话，Windows必须装win10，相信我，没错的。

**2、编译源码**

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

你需要去拉一下代码，我试过，真特么的慢，这条路就别想了，直接到git上把rk3288分支上

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
