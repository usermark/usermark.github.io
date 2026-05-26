---
title: "[Armbian] 安博機上盒刷機"
tags:
  - Armbian
  - 刷機
---
這次使用的是安博盒子 UBOX 9 PRO MAX，CPU 比較夠力，刷 Armbian 跑看看。順便認識 linux kernel 編譯。
<!--more-->

![](/assets/tx6s.jpg)

# 拆解

![](/assets/tx6s2.jpg)

拆解後，找到 TTL 接腳 (紅框)，依標示接上 CP2102 (USB 轉 TTL) 的晶片，波特率為 115200。

![](/assets/tx6s3.jpg)

完整的啟動資訊如下
```sh
[399]HELLO! BOOT0 is starting!
[402]BOOT0 commit : 4656b05
[405]set pll start
[407]periph0 has been enabled
[410]set pll end
[412]unknow PMU
[414]PMU: AXP806
[421]vaild para:16  select dram para4
[425]board init ok
[426]DRAM BOOT DRIVE INFO: V0.532
[430]the chip id is 0x5000
[432]chip id check OK
[435]DRAM_VCC set to 1500 mv
[439]read_calibration error
[443]read_calibration error
[447]read_calibration error
[451]read_calibration error
[455]read_calibration error
[459]read_calibration error
[463]read_calibration error
[467]read_calibration error
[471]read_calibration error
[475]read_calibration error
[477]retraining final error
[484][AUTO DEBUG]32bit,1 ranks training success!
[491]DRAM CLK =648 MHZ
[493]DRAM Type =3 (3:DDR3,4:DDR4,7:LPDDR3,8:LPDDR4)
[500]Actual DRAM SIZE =4096 M
[502]DRAM SIZE =4096 MBytes, para1 = 310b, para2 = 10000000, dram_tpr13 = 6041
[515]DRAM simple test OK.
[517]rtc standby flag is 0x0, super standby flag is 0x0
[523]dram size =4096
[526]card no is 2
[527]sdcard 2 line count 8
[530][mmc]: mmc driver ver 2019-12-19 10:41
[534][mmc]: set f_max to 50M, set f_max_ddr to 50M
[539][mmc]: mmc 2 bias 4
[547][mmc]: ***Try MMC card 2***
[560][mmc]: MMC 5.0
[562][mmc]: HSDDR52/DDR50 8 bit
[565][mmc]: 50000000 Hz
[568][mmc]: 59008 MB
[570][mmc]: ***SD/MMC 2 init OK!!!***
[639]Loading boot-pkg Succeed(index=0).
[643]Entry_name        = u-boot
[652]Entry_name        = monitor
[656]Entry_name        = dtbo
[659]Entry_name        = dtb
[662]tunning data addr:0x4a0003e8
[665]Jump to second Boot.
NOTICE:  BL3-1: v1.0(debug):09e19e7
NOTICE:  BL3-1: Built : 11:30:42, 2020-09-29
NOTICE:  BL3-1 commit: 8
ERROR:   Error initializing runtime service tspd_fast
NOTICE:  BL3-1: Preparing for EL3 exit to normal world
NOTICE:  BL3-1: Next image address = 0x4a000000
NOTICE:  BL3-1: Next image spsr = 0x1d3

U-Boot 2018.05-00007-g782b5e0-dirty (Aug 16 2021 - 10:46:05 +0800) Allwinner Tec                                                                                                                                                             hnology

[00.741]CPU:   Allwinner Family
[00.744]Model: sun50iw9
I2C:   ready
[00.748]DRAM:  2 GiB
[00.751]Relocation Offset is: 75ec5000
[00.789]secure enable bit: 0
[00.792]pmu_axp152_probe pmic_bus_read fail
[00.796]PMU: AXP806
[00.801]CPU=1008 MHz,PLL6=600 Mhz,AHB=200 Mhz, APB1=100Mhz  MBus=400Mhz
[00.992]sunxi overlay merged okqv
[00.995]drv_disp_init
[01.026]__clk_enable: clk is null.
[01.032]drv_disp_init finish
[01.034]gic: 20201231 sec monitor mode
[01.048]flash init start
[01.050]workmode = 0,storage type = 2
[01.053]MMC:     2
[01.055]********* 20210428 0************
[01.058]********* 20210428 1************
[01.062]get mem for descripter OK !
[01.071]get sdc2 sdc_boot0_sup_1v8 fail.
[01.076]io is 1.8V
[01.077]********* 20210428 2************

[01.081]********* 20210428 3************
[01.085]********* 20210428 mmc_start_init ************
sprite_led_gpio start
[01.094]********* 20210428 gpio ok************
[01.098]********* 20210428 6************
[01.102]********* 20210428 7************
[01.105]********* 20210428 8************
[01.109]********* 20210428 9************
[01.130]already at HSSDR52_SDR25 mode
[01.133]sunxi flash init ok
[01.137]Loading Environment from SUNXI_FLASH... OK
[01.147]usb burn from boot
delay time 0
weak:otg_phy_config
[01.160]usb prepare ok
[01.963]overtime
[01.967]do_burn_from_boot usb : no usb exist
[01.971]boot_gui_init:start
FAT: Misaligned buffer address (bbe80cd8)
32 bytes read in 5 ms (5.9 KiB/s)
sprite_time_func= blue only
[02.259]boot_gui_init:finish
[02.262]bmp_name=bootlogo.bmp
2764856 bytes read in 24 ms (109.9 MiB/s)
[02.307]hsddr 2-50000000
[02.309]hs200 5-200000000
[02.311]hs400 3-100000000
[02.314]get max-frequency ok 100000000 Hz
[02.318]0 0 0: 0 0 0
[02.323]update dts
** Unrecognized filesystem type **
[02.338]load file(ULI/factory/rootwait init.txt) error.
[02.343]name in map snum
[02.345]name in map mac
** Unrecognized filesystem type **
[02.361]load file(ULI/factory/wifi_mac.txt) error.
** Unrecognized filesystem type **
[02.379]load file(ULI/factory/bt_mac.txt) error.
** Unrecognized filesystem type **
[02.397]load file(ULI/factory/selinux.txt) error.
** Unrecognized filesystem type **
[02.414]load file(ULI/factory/specialstr.txt) error.
[02.433]update part info
[02.452]update bootcmd
[02.454]No ethernet found.
Hit any key to stop autoboot:  0
[04.685]Starting kernel ...

[04.687]mmc exit start
[04.699]mmc 2 exit ok
[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 4.9.170 (yifan@yifan-Z270-HD3) (gcc version 5.3.1 2                                                                                                                                                             0160412 (Linaro GCC 5.3-2016.05) ) #17 SMP PREEMPT Sun Aug 15 10:12:32 CST 2021
[    0.000000] Boot CPU: AArch64 Processor [410fd034]
[    0.000000] bootconsole [earlycon0] enabled
[    0.027741] BOOTEVENT:        27.727749: ON
[    0.251878] sunxi_i2c_probe()2209 - [i2c3] warning: failed to get regulator i                                                                                                                                                             d
[    0.252877] sunxi_i2c_probe()2209 - [i2c5] warning: failed to get regulator i                                                                                                                                                             d
[    0.303860] acx00_i2c_probe,l:282
[    0.304019] [ac200] pwm is NULL! Just initialize it.
[    0.304813] [ac200] pwm enable
[    0.304846] [ac200] pwm is initialized
[    0.304923] acx00_init_work,l:130
[    0.305644] acx00_init_work,l:137
[    0.306130] sunxi_i2c_do_xfer()1935 - [i2c5] incomplete xfer (status: 0x20, d                                                                                                                                                             ev addr: 0x10)
[    0.312013] [ac200] get ave_regulator_name failed!
[    0.403127] failed to get standby led pin assign
[ ▒[    0.409753] uart uart1: get regulator failed
[    0.442077] [NAND][NE] Not found valid nand node on dts
[    0.450795] sunxi-wlan soc@03000000:wlan: get gpio chip_en failed
[    0.457679] sunxi-wlan soc@03000000:wlan: get gpio power_en failed
[    0.596362] hci: request ohci1-controller gpio:232
[    0.787556] axp2101_pek: axp2101-pek can not register without irq
[    0.797816] sunxi_ir_startup: get ir protocol failed
[    0.806014] VE: get debugfs_mpp_root is NULL, please check mpp
[    0.806014]
[    0.814271] VE: sunxi ve debug register driver failed!
[    0.814271]
[    0.829634] mmc:failed to get gpios
[    0.909011] mmc:failed to get gpios
[    0.945678] sunxi-mmc sdc1: smc 2 p1 err, cmd 52, RTO !!
[    0.952484] sunxi-mmc sdc1: smc 2 p1 err, cmd 52, RTO !!
[    0.954243] failed get gpio-spdif gpio from dts,spdif_gpio:-2
[    0.956129] [sunxi_internal_codec_probe]:get audio avcc failed
[    0.956165] [sunxi_internal_codec_probe]:get audio vcc3v3-audio failed
[    0.956399] [audio-codec]dachpf_cfg configurations missing or invalid.
[    0.956409] lineout_vol:26, linein_gain:3, fmin_gain:3, digital_vol:0, adcdrc                                                                                                                                                             _cfg:0, adchpf_cfg:0, dacdrc_cfg:0, dachpf_cfg:0, ramp_func_used:1, pa_msleep_ti                                                                                                                                                             me:160, pa_ctl_level:0, gpio-spk:0
[    1.006385] sndhdmi sndhdmi: ASoC: CPU DAI (null) not registered
[    1.013225] sndhdmi sndhdmi: snd_soc_register_card() failed: -517
```

# 燒錄

這台機上盒使用的 CPU 是 Allwinner H616，板子為 TX6s，可惜 Armbian 沒有對應的板子

## 編譯

筆者使用 Ubuntu 24.04 (WSL2) 進行編譯

下載 Armbian
```shell
git clone https://github.com/armbian/build
cd build
```

安裝工具包
```shell
sudo apt-get update -y
sudo apt-get full-upgrade -y
sudo apt-get install -y acl aptly aria2 axel bc binfmt-support binutils-aarch64-linux-gnu bison bsdextrautils btrfs-progs build-essential busybox ca-certificates ccache clang coreutils cpio crossbuild-essential-arm64 cryptsetup curl debian-archive-keyring debian-keyring debootstrap device-tree-compiler dialog dirmngr distcc dosfstools dwarves e2fsprogs expect f2fs-tools fakeroot fdisk file flex gawk gcc-arm-linux-gnueabi gdisk git gpg gzip imagemagick jq kmod libbison-dev libc6-dev-armhf-cross libcrypto++-dev libelf-dev libfdt-dev libfile-fcntllock-perl libfl-dev libfuse-dev libgcc-12-dev-arm64-cross libgmp3-dev liblz4-tool libmpc-dev libncurses-dev libpython3-dev libssl-dev libusb-1.0-0-dev linux-base lld llvm locales lz4 lzma lzop make mtools ncurses-base ncurses-term nfs-kernel-server ntpdate openssl p7zip p7zip-full parallel parted patchutils pbzip2 pigz pixz pkg-config pv python3 python3-dev python3-setuptools qemu-user-static rdfind rename rsync sudo swig tar tree u-boot-tools udev unzip util-linux uuid uuid-dev uuid-runtime vim wget whiptail xfsprogs xsltproc xz-utils zip zlib1g-dev zstd
```

## 取得設備樹

將設備樹複製到電腦上分析，筆者透過 TFTP 取檔
```shell
su
busybox tftp -p -l /sys/firmware/fdt -r tx6s.dtb 192.168.1.100
```

回到 Ubuntu，把檔案放到 \\\\wsl$\Ubuntu\home\your_name 底下，輸入以下指令將二進位 dtb 轉為可讀的 dts
```shell
dtc -I dtb -O dts -o tx6s.dts tx6s.dtb
```

獲得轉換後的檔案 [tx6s.dts](/assets/tx6s.dts)

# 新增設備

```shell
./compile.sh BOARD=tanix-tx6 BRANCH=current kernel-patch
```

看到暫停提示，先不要按 Enter
```
[💲|🚸] Make your changes in this directory: [ /home/XXX/build/cache/sources/linux-kernel-worktree/6.18__sunxi64__arm64 ]
[💲|🚸] Press <ENTER> after you are done [ editing files in /home/XXX/build/cache/sources/linux-kernel-worktree/6.18__sunxi64__arm64 ]
Press ENTER to show a preview of your patch, or type 'stop' to stop patching...
```

另開視窗將 dts 檔複製進去
```shell
cd build
sudo cp ../tx6s.dts cache/sources/linux-kernel-worktree/6.18__sunxi64__arm64/arch/arm64/boot/dts/allwinner/sun50i-h616-tx6s.dts
sudo nano cache/sources/linux-kernel-worktree/6.18__sunxi64__arm64/arch/arm64/boot/dts/allwinner/Makefile
```

Makefile 加入底下這段
```
dtb-$(CONFIG_ARCH_SUNXI) += sun50i-h616-tx6s.dtb
```

回到原本的視窗按下 Enter
會在底下自動生成 output/patch/kernel-sunxi64-current.patch


```shell
mkdir -p userpatches/kernel/sunxi64-current/
cp output/patch/kernel-sunxi64-current.patch userpatches/kernel/sunxi64-current/
```

新增開發板設定檔
nano config/boards/tx6s.conf
```
# Allwinner H616 TVBox with 4GB of RAM and EMMC
BOARD_NAME="Unblock TX6s"
BOARD_VENDOR="allwinner"
BOARDFAMILY="sun50iw9"
BOARD_MAINTAINER=""
INTRODUCED="2021"
KERNEL_TARGET="current,edge"
KERNEL_TEST_TARGET="current"
OVERLAY_PREFIX="sun50i-h616"
KERNEL_CONFIG="linux-sunxi64-current.config"
```

```shell
./compile.sh BOARD=tx6s BRANCH=current uboot-patch
```

sudo nano cache/sources/u-boot-worktree/u-boot/v2024.01/configs/tx6s_defconfig
```
CONFIG_ARM=y
CONFIG_ARCH_SUNXI=y
CONFIG_DEFAULT_DEVICE_TREE="allwinner/sun50i-h616-tx6s"
CONFIG_SPL=y
CONFIG_SUNXI_DRAM_H616_DDR3_1333=y
CONFIG_DRAM_CLK=648
CONFIG_DRAM_ODT_EN=y
CONFIG_DRAM_SUNXI_UNKNOWN_FEATURE=y
CONFIG_DRAM_SUNXI_BIT_DELAY_COMPENSATION=y
CONFIG_DRAM_SUNXI_READ_CALIBRATION=y
CONFIG_DRAM_SUNXI_DX_ODT=0x03030303
CONFIG_DRAM_SUNXI_DX_DRI=0x0e0e0e0e
CONFIG_DRAM_SUNXI_CA_DRI=0x00001c12
CONFIG_DRAM_SUNXI_ODT_EN=0x00000001
CONFIG_DRAM_SUNXI_TPR0=0xc0000c05
CONFIG_DRAM_SUNXI_TPR2=0x00000000
CONFIG_DRAM_SUNXI_TPR10=0x2f0007
CONFIG_DRAM_SUNXI_TPR11=0xffffdddd
CONFIG_DRAM_SUNXI_TPR12=0xfedf7557
CONFIG_MACH_SUN50I_H616=y
CONFIG_R_I2C_ENABLE=y
CONFIG_SPL_I2C=y
CONFIG_SPL_I2C_SUPPORT=y
CONFIG_SPL_SYS_I2C_LEGACY=y
CONFIG_SYS_I2C_MVTWSI=y
CONFIG_SYS_I2C_SLAVE=0x7f
CONFIG_SYS_I2C_SPEED=100000
CONFIG_PHY_REALTEK=y
CONFIG_SUN8I_EMAC=y
CONFIG_I2C3_ENABLE=y
CONFIG_USB_EHCI_HCD=y
CONFIG_USB_OHCI_HCD=y
CONFIG_USB_MUSB_GADGET=y
CONFIG_SUPPORT_EMMC_BOOT=y
CONFIG_SYS_MMCSD_RAW_MODE_U_BOOT_SECTOR=0x40
CONFIG_MMC_SUNXI_SLOT_EXTRA=2
CONFIG_OF_LIBFDT_OVERLAY=y
CONFIG_CMD_NET=y
CONFIG_CMD_DHCP=y
CONFIG_CMD_WGET=y
CONFIG_CMD_TFTPBOOT=y
CONFIG_CMD_PING=y
CONFIG_PROT_TCP=y
CONFIG_LIB_RAND=y
CONFIG_WGET=y
CONFIG_NET_LWIP=y
CONFIG_CMD_WGET_LWIP=y
CONFIG_BOOTCOMMAND="if dhcp ${scriptaddr} /PXEclient/pxelinux.cfg/sun50i-h616-tx6s.scr; then source ${scriptaddr}; fi; run distro_bootcmd"
CONFIG_CMD_MII=y
CONFIG_CMD_MDIO=y
```

```shell
./compile.sh BOARD=tx6s RELEASE=trixie BUILD_DESKTOP=no BUILD_MINIMAL=yes KERNEL_CONFIGURE=no
```