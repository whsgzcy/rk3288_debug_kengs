# rk3288 debug kengs / that's so funny super_yu

**一、编译Android源码准备**

**1、外部环境建议**

最好是有一台Windows电脑，因为他可以为所欲为，如果非要在Ubuntu里面安装Windows虚拟机的话，至少惠普电脑貌似是怎么操作都操作不起来的。

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


