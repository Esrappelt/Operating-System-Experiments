## 准备工作
- 首先是下载Linux内核headers，包括了module.h和kernel.h这两个文件

```
sudo apt-get install linux-headers-$(uname -r)
```
- 查看是否安装成功

```
uname -r
```
若有版本信息，则成功

- Makefile的编写

```
# To build modules outside of the kernel tree, we run "make"
# in the kernel source tree; the Makefile these then includes this
# Makefile once again.
# This conditional selects whether we are being included from the
# kernel Makefile or not.

# called from kernel build system: just declare what our modules are
obj-m := helloworld.o

CROSS_COMPILE =

CC            = gcc



    # Assume the source tree is where the running kernel was built
    # You should set KERNELDIR in the environment if it's elsewhere
    KERNELDIR ?= /usr/src/linux-headers-$(shell uname -r)
    # The current directory is passed to sub-makes as argument
    PWD := $(shell pwd)

all:     modules


modules:
        $(MAKE) -C $(KERNELDIR) M=$(PWD) modules


clean:
        rm -rf *.o *~ core .depend *.symvers .*.cmd *.ko *.mod.c .tmp_versions $(TARGET)

```
- Makefile的注意点
- [ ]  modules和clean下面这句的前面是必须按tab，有空隙的
- [ ] obj-m := helloworld.o的helloworld要和c文件同名
- [ ] /usr/src/linux-headers-$(shell uname -r)里面的这个uname -r就是你下载linux-headers文件，整个代码是这个文件的路径
## 编写内核c代码

```
#include <linux/module.h>
#include <linux/kernel.h>

int init__module(void)//初始化模块,insmod命令会调用
{   
    printk("---------------begin------------------------\n");
    printk("<1>Hello World!\n");
    printk("<2>第二级别的信息\n");
    printk("<3>第三级别的信息\n");
    return 0;
}

void cleanup__module(void)//清除模块,rmmod命令会调用
{
    printk("----------------END--------------------\n");
    printk("Hello World! End of hello world module!\n");
}


MODULE_LICENSE("Dual BSD/GPL");
module_init(init__module);
module_exit(cleanup__module);

```
其中，printk代表Linux中的输出，<1>代表的是优先级。init__module是向内核注册新功能，cleanup__module是清除新功能。

## 开始编译和加载
- 编译
    
```
sudo make
```
当编译成功的时候，会出现这些文件

```
helloworld.c   helloworld.mod.c  helloworld.o  modules.order
helloworld.ko  helloworld.mod.o  Makefile      Module.symvers

```

- 加载

```
insmod helloworld.ko
```
加载完成后，输入命令查看printk的内容

```
sudo dmesg
```

- 查看模块


```
lsmod
```
即可查看到helloworld

- 卸载模块

```
rmmod helloworld
```
卸载完模块后，可以用dmesg查看内容
