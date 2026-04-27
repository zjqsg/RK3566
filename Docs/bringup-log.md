# 26/4/11 收到板子
 对于正常流程来说是先焊接PMIC然后上电测试电压后才焊接SOC，但是rk3566芯片引脚比较密集，加之身边没有比较专业的设备，为了保险，直接裸板贴了两片SOC，避免淘到有问题的芯片和手工焊接引起的虚焊等风险，并且soc背面有很多0210的小电容需要焊接，不得已只能使用万用表蜂鸣档测量各电源域的通断情况并且做了记录，以这种方法来来观察是否电路正常。（-代表无穷大，X代表数值反复横跳，？代表调一下数值变成无穷大）
  | 电源域 | 只有SOC   | 焊接SOC背面电容 | 焊接PMIC | 上电后电压  |  断电焊接DDR | 
  | :----- | :------: | -----:         |  -----:  | -----: | -----: |
  | VDD_LOGIC  | 50.1Ω  | 60.1Ω   |   70Ω      |    0.9v  |         |
  | VDD_GPU  | 110Ω     | 111Ω    |   0.053v   |   0.8v    |        |
  | VDD_CPU  | 0.053v   |  114Ω   |  114.8Ω    |   1.03v    |       |
  | VDD_NPU  |  0.124v  | 0.115v  |   0.137v  |    0v     |         |
  | VCC_1V8  | 1.128V   |  0.817v  |   0.859v  |   1.79v  |         |
  | VCC_DDR  | 0.245v   | 0.430v  |   0.442v   |   1.11v   | 0.078v |   
  | VDDA0V9_IMAGE  | 0.563v    | 0.510v  | 0.513v |   0v   |         |
  | VDDA_0V9 | 0.698v    | 0.631v  | 0.646v    |    0v    |         |
  | VDDA0V9_PMU  | 0.999v| 0.991v  | 0.992v    |   0.9v   |         |
  | VCCIO_ACODEC | 1.64v | 0.964v  |  0.951v   |    0v    |         |
  | VCCIO_SD  | 1.68v    |  0.943v |  1.079v   |   3.2v   |         |
  | VCC3V3_PMU | 1.263v  | 0.198v  |  0.907v   |  3.28v   |         |
  | VCCA_1V8  | 0.732v  | 1.0v     |  1.182v   |   1.8v   |         |
  | VCCA1V8_PMU |  -     |  -      |    x      |  1.8v    |          |
  | VCCA1V8_IMAGE  |  1.935v |  ？  |    ？    |   0v     |          |
  | VCC3V3_SD |  -       |  -      |     x     | 3.28v    |          | 
  | VCC_3V3  | 0.762v    | 0.87v   |   0.797v  |  3.28v   |          |
  | VCC3V3_SYS |  -      |  -      |      ?    |  3.28v    |         |
## 焊接PMIC
  发现问题：有几处LOD输出有跳变，貌似数值不正常，于是检查这一路电源的电容是否焊接过程中短路，仔细观察后没有发现明显的短路痕迹，看到PMIC的引脚的焊接状态比较不好，于是重新焊接了PMIC，但是发现还是这个状态，
  但是调换了一下表笔顺序，黑表笔接被测段，红表笔接地，发现都是稳定在0.42v左右，然后想到把可调电源限流200ma后上电观察电流值，开启电源后，电流此时稳定在100ma，测量这两个测试点后，发现这两个电压正常稳定在各自的电压范围，

 ## 开始焊接DDR
  非常幸运的是，在内存没有涨价前买了两片LPDDR4 2G，大概一个30，现在涨价到100了... 相对于RK3566这种密集的引脚来说，LPDDR4的焊接还是比较简单的，焊接的压力也比较小，热风枪风嘴拿掉，3档350轻松搞定，在焊接完成DDR后，测量了一下相关电路的情况，发现VCC_DDR这路的压降下降到0.078v，查找资料后发现是LPDDR4内部有大量的ESD和多个DDR芯片并联导致。
  
 ## 烧录MiniloaderAll.bin
  在焊接完成PMIC、SOC和DDR后，在电脑上安装瑞芯微的驱动，这里有个坑，我之前下载的Driverassistant不是最新版的，导致驱动识别不成功，复盘后发现这个版本还没有包含RK3566的驱动。下载最新版后，打开烧录软件已经显示找到了一个Maskrom设备，在烧录完成MiniloaderAll.bin后，成功的读取到了芯片的信息。
  ![](Docs/images/Read ChipInfo.png)
## 烧录系统
 在确认当前电路正常后，我便着手准备开始烧录系统事宜，当然，在我绘制PCB阶段，也买了块开发板学习了一下，稍微的补偿了一下Liunx的基础知识，我使用的SDK是野火的LubanCat_Linux_rk356x_SDK,但在编译系统的过程中，遇到了很多的问题。
  问题1：鲁班猫的板卡是根据ADC采样的值来自动选择dts文件的，但我的硬件上面没有考虑到这个问题，于是我各种查阅资料，好在网上有大佬也搞过这个SDK，虽然芯片不一样，但是也有迹可循。
  解决办法：由于我编译的是Debian系统，所以在SDK目录下的Debian/overlay/etc/init.d的boot_init.sh这个文件里面修改，将函数board_info()下RK3566判断下的0100改为自己的dts文件名，
  ```
		0100)
				BOARD_NAME='LubanCat-1N'
				BOARD_DTB='rk3566-myboard.dtb'
				BOARD_uEnv='uEnv.txt'
  ```
并且更改board_id()函数如下 注释掉获取ADC值的步骤，直接填写BOARD_ID的值去对应board_info()函数判断的值，这样在uboot结束后就会指向你的dts文件。
  ```
board_id() {
	#ADC_voltage_scale=$(cat /sys/bus/iio/devices/iio\:device0/in_voltage_scale)
	#echo "ADC_voltage_scale:"$ADC_voltage_scale

	SOC_type=$(cat /proc/device-tree/compatible | cut -d,  -f 3 | sed 's/\x0//g')
	echo "SOC_type:"$SOC_type

	#ADC_CH2_RAW=$(cat /sys/bus/iio/devices/iio\:device0/in_voltage2_raw)
	#echo "ADC_CH2_RAW:"$ADC_CH2_RAW
	#ADC_CH3_RAW=$(cat /sys/bus/iio/devices/iio\:device0/in_voltage3_raw)
	#echo "ADC_CH3_RAW:"$ADC_CH3_RAW

	#get_index $ADC_CH2_RAW
	#ADC_CH2_INDEX=$INDEX

	#get_index $ADC_CH3_RAW
	#ADC_CH3_INDEX=$INDEX

	BOARD_ID="0100"
	echo "BOARD_ID:"$BOARD_ID
}
  ```
问题2：由于Debian10系统已结束生命周期，但是编译Debian系统还需要在线下载一些文件，种种原因导致编译不通过。
解决办法：查找一番资料后，更换了Debian/ubuntu-build-service/buster-lite-arm64下的config的lb config\下地址更改为国内源
```
lb config \
 --mirror-bootstrap "https://mirrors.aliyun.com/debian-archive/debian" \
 --mirror-chroot "https://mirrors.aliyun.com/debian-archive/debian" \
 --mirror-chroot-security "https://mirrors.aliyun.com/debian-archive/debian-security" \
 --mirror-binary "https://mirrors.aliyun.com/debian-archive/debian" \
 --mirror-binary-security "https://mirrors.aliyun.com/debian-archive/debian-security" \
```
但是编译时，还是出现问题，这次的提示是
#----------------------------
zjqsg@ubuntu:~/SDK/debian$ lb config \
>     --distribution buster \
>     --archive-areas "main contrib non-free" \
>     --mirror-bootstrap http://archive.debian.org/debian \
>     --mirror-chroot http://archive.debian.org/debian \
>     --mirror-binary http://archive.debian.org/debian \
>     --parent-mirror-chroot-security http://archive.debian.org/debian-security \
>     --parent-mirror-binary http://archive.debian.org/debian \
>     --apt-options "--yes -o Acquire::Check-Valid-Until=false"
[2026-04-15 20:28:21] lb config --distribution buster --archive-areas main contrib non-free --mirror-bootstrap http://archive.debian.org/debian --mirror-chroot http://archive.debian.org/debian --mirror-binary http://archive.debian.org/debian --parent-mirror-chroot-security http://archive.debian.org/debian-security --parent-mirror-binary http://archive.debian.org/debian --apt-options --yes -o Acquire::Check-Valid-Until=false
P: Creating config tree for a ubuntu/buster/amd64 system
zjqsg@ubuntu:~/SDK/debian$
#----------------------------
