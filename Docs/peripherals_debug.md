## 外设调试记录
在bringup成功后，我开始了外设的焊接调试

##USB2.0/3.0调试
USB2.0/3.0的调试比较简单，在焊接完成外围电路后，在DTS里面添加
```
/* USB Host_2/USB Host_3 USB 2.0 Comb */
&usb2phy1 {
	status = "okay";
};

&u2phy1_otg {
	status = "okay";
};

&u2phy1_host {
	status = "okay";
};

&usb_host1_ehci {
	status = "okay";
};

&usb_host1_ohci {
	status = "okay";
};

/* USB3.0 OTG Only USB2.0 */
&usbdrd30 {
	status = "okay";
};

&usbdrd_dwc3 {
	status = "okay";
};

/* USB3.0 Host */
&combphy1_usq {
	status = "okay";
};

&usbhost30 {
	status = "okay";
};

&usbhost_dwc3 {
	status = "okay";
};
```
烧录后

##AP6212适配固件

由于使用的鲁班猫SDK中没有适配AP6212的固件，所以要手动适配，第一步是要配置menuconfig
在sdk的kernel目录使用
···
make ARCH=arm64 rockchip_linux_defconfig menuconfig
···
按下/键搜索，输入AP6，

##HDIM调试
  bringup时使用是Debian lite的镜像，为了使用HDMI，我重新构建了xfec的镜像系统，我先通过VNC远程桌面测试了一下，结果很好，成功的显示了桌面，然后我在dts里面使用了鲁班猫的HDMI头文件，但是在连接到显示器的时候出现了问题，在uboot阶段
  
>  hdmi@fe0a0000:  detailed mode clock 148500 kHz, flags[5]
>    H: 1920 2008 2052 2200
>    V: 1080 1084 1089 1125
> bus_format: 2025
> VOP update mode to: 1920x1080p60, type: HDMI0 for VP0
> VP0 set crtc_clock to 148500KHz
> VOP VP0 enable Smart0[772x525->772x525@574x277] fmt[1] addr[0x7e029000]
> CEA mode used vic=16
> final pixclk = 148500000 tmdsclk = 148500000
> PHY powered down in 0 iterations
> PHY PLL locked 1 iterations
> PHY powered down in 0 iterations
> PHY PLL locked 1 iterations
> sink has audio support
> hdmi_set_clk_regenerator: fs=48000Hz ftdms=148.500MHz N=6144 cts=148500

在内核阶段
> [    0.199586] rockchip-drm display-subsystem: Linked as a consumer to fe0a0000.hdmi
> [    0.201661] dwhdmi-rockchip fe0a0000.hdmi: Detected HDMI TX controller v2.11a with HDCP (DWC HDMI 2.0 TX PHY)
> [    0.202246] dwhdmi-rockchip fe0a0000.hdmi: registered DesignWare HDMI I2C bus driver
> [    0.202827] rockchip-drm display-subsystem: bound fe0a0000.hdmi (ops 0xffffff80090fbe08)
> [    0.447556] rockchip-vop2 fe040000.vop: [drm:vop2_crtc_atomic_disable] Crtc atomic disable vp0
> [    0.464625] rockchip-vop2 fe040000.vop: [drm:vop2_crtc_atomic_enable] Update mode to 2560x1440p60, type: 11 for vp0
> [    0.485189] dwhdmi-rockchip fe0a0000.hdmi: Rate 241500000 missing; compute N dynamically
> [    0.548362] rockchip-drm display-subsystem: fb0:  frame buffer device
> [    0.549276] [drm] Initialized rockchip 2.0.0 20140818 for display-subsystem on minor 0
  
