# Description

**RV-Debugger-BL702** is an opensource project that implement a JTAG+UART debugger with BL702C-A0.

[BL702](https://www.bouffalolab.com/bl70X) is highly integrated BLE and Zigbee combo chipset for IoT applications, contains **32-bit RISC-V** CPU with FPU, frequency up to **144MHz**, with **132KB RAM** and **192 KB ROM**, 1Kb eFuse, **512KB embedded Flash**, USB2.0 FS device interface, and many other features.

The firmware implement is inspired by [open-ec](https://github.com/rgwan/open-ec), implement by Sipeed teams and community developers.

The firmware emulate an [FT2232D](https://ftdichip.com/products/ft2232d/) device, defaultly implement an JTAG+UART debugger, and can be implement as a Dual-Serial Port debugger, a bluetooth debugger, etc. 

Community Forum:  bbs.sipeed.com


# Yuki's Notes:

See https://github.com/sipeed/RV-Debugger-BL702/issues/14

这个仓库里面的 usb2uartjtag 已经patch过了

编译固件:

```bash
cd firmware/bl_mcu_sdk
make BOARD=bl702_debugger APP_DIR=../app APP=usb2uartjtag
```

编译好的固件： https://github.com/yuk1i/RV-Debugger-BL702/releases/tag/v1.0

```shell
$ sha256sum usb2uartjtag_bl702.bin
3c28389c44e00ec7741f0f324e0832dec6ee216a63d184d17214afd83e6a09aa  usb2uartjtag_bl702.bin
```

如果正确烧录，插入电脑后可以看到：usb的 iSerial 有 SUSTC 的标签

```shell
$ lsusb -v -d 0403:6010

Bus 001 Device 085: ID 0403:6010 Future Technology Devices International, Ltd FT2232C/D/H Dual UART/FIFO IC
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 [unknown]
  bDeviceSubClass         0 [unknown]
  bDeviceProtocol         0 
  bMaxPacketSize0        64
  idVendor           0x0403 Future Technology Devices International, Ltd
  idProduct          0x6010 FT2232C/D/H Dual UART/FIFO IC
  bcdDevice            5.00
  iManufacturer           1 SIPEED
  iProduct                2 JTAG+UART    
  iSerial                 3 SUSTC-8a76a221 
  bNumConfigurations      1
```

烧录：

**用 Windows 刷**

https://bouffalolab.gitee.io/bl_mcu_sdk/get_started/bl_dev_cube.html

别tm用linux,傻逼厂商

# Hardware
## Offical Board

Sipeed RV-Debugger-Plus  (BL702C-A0)

Purchase link: taobao.com/xxx

Schematic: [BL702_USB2JTAG_3610_sch.pdf](hardware/BL702_USB2JTAG_3610_sch.pdf)

Assembly: [BL702_USB2JTAG_3610_asm.pdf](hardware/BL702_USB2JTAG_3610_asm.pdf)

(LED0 is the led close to edge, indicate for RX)

<img height=480 src="res/rv-debugger.png">
<br />

## Third-party Hardware

TODO

# Firmware
## Flash Tutorial

Press "boot" button then plug usb cable, and you will see "CDC Virtual ComPort" in device manager , remember the com number.

The flash tool is in tools/bflb_flash_tool directory, and input the command (replace port number and firmware name):

~~~
Windows:
.\bflb_mcu_tool.exe --chipname=bl702 --port=COM9 --xtal=32M --firmware="main.bin"
Linux:
./bflb_mcu_tool --chipname=bl702 --port=/dev/ttyACM0 --xtal=32M --firmware="main.bin"
~~~

~~~
tools\bflb_flash_tool> .\bflb_mcu_tool.exe --chipname=bl702 --port=COM9 --xtal=32M --firmware="main.bin"
[22:07:28.296] - ==================================================
[22:07:28.296] - Chip name is bl702
[22:07:28.297] - Serial port is COM9
[22:07:28.299] - Baudrate is 115200
[22:07:28.299] - Firmware is main.bin
[22:07:28.300] - Default flash clock is 72M
[22:07:28.300] - Default pll clock is 144M
[22:07:28.311] - ==================================================
[22:07:28.483] - Update flash cfg finished
[22:07:28.500] - EFUSE_CFG
[22:07:28.500] - BOOTHEADER_CFG
......
[22:07:31.274] - Load 53856/53856 {"progress":100}
[22:07:31.274] - Write check
[22:07:31.274] - Flash load time cost(ms): 267.942626953125
[22:07:31.275] - Finished
[22:07:31.276] - Sha caled by host: 825d198270c2cf509acda8f8e0830751c532da802060c324a4479e1fe599ae1f
[22:07:31.276] - xip mode Verify
[22:07:31.288] - Read Sha256/53856
[22:07:31.288] - Flash xip readsha time cost(ms): 12.508056640625
[22:07:31.288] - Finished
[22:07:31.288] - Sha caled by dev: 825d198270c2cf509acda8f8e0830751c532da802060c324a4479e1fe599ae1f
[22:07:31.288] - Verify success
[22:07:31.289] - Program Finished
[22:07:31.289] - All time cost(ms): 2220.2548828125
[22:07:31.390] - [All Success]
~~~
Another GUI flash tool is BouffaloLabDevCube:
- [Bouffalo Lab Dev Cube For Windows](https://dev.bouffalolab.com/media/upload/download/BouffaloLabDevCube-1.5.2-win32.zip)
- [Bouffalo Lab Dev Cube For Ubuntu](https://dev.bouffalolab.com/media/upload/download/BouffaloLabDevCube-1.5.2-linux-x86.tar.gz)

## usb2uartjtag (default)
Support JTAG+UART function

UART support baudrate below 2Mbps, and 3Mbps, and some experimental baudrate (stability is not guaranteed):

~~~
12M, 9.6M, 8M, 6.4M, 6M, 4.8M, 4M, 3.2M
we remap baudrate in 10000~12000 to (baud-10000)*10000
for example, 11200bps -> 12Mbps
~~~

LED0 for RX indication, LED1 for TX indication.

JTAG function is verified for :

- RV32 Xuantie E906/E907
- RV64 Xuantie C906
- Gowin FPGA GW1N-1, GW1NS-4C. (need enable GOWIN_INT_FLASH_QUIRK)

## usb2dualuart
TODO.


# Project Structure

```
RV-Debugger-BL702
├── bsp
│   ├── board
│   │   └── bl702_debugger
│   └── bsp_common
│       └── platform
├── common
│   ├── device
│   ├── libc
│   ├── list
│   ├── memheap
│   ├── misc
│   ├── partition
│   ├── ring_buffer
│   └── soft_crc
├── components
│   ├── shell
│   └── usb_stack
├── drivers
│   └── bl702_driver
├── examples
│   └── usb
│       ├── usb2uartjtag
│       └── usb2dualuart
├── out
├── tools
│   ├── bflb_flash_tool
│   ├── cdk_flashloader
│   ├── cmake
│   └── openocd
└── hardware
    
```
BL SDK usage tutorial refer to http://bouffalolab.gitee.io/bl_mcu_sdk/

## Develop Guide
### Build
just "make"

### Code Explanation
the main file is:
~~~
examples/usb/usb2uartjtag:
├── main.c
├── uart_interface.c
├── jtag_process.c
└── io_cfg.h         //main io cfg, another file is pinmux_config.h in bsp/board/bl702_debugger
components/usb_stack/class/vendor:
└── usbd_ftdi.c      //all FTDI vendor request process, like baudrate set, dtr/rts set, Latency_Timer
~~~

TODO

