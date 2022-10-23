# Integrating STM32F407 with Azure IoT Hub.
This README file goes through detailed step by step method to integrate a microcontroller board(STM32F407) with Azure Cloud(IoT Hub).

.
.


Please note that this project is still under progress(as of 23rd of October).

The project could be broken down into four major parts. Briefly explained as:

A. Make STM32F407 understand MicroPython, so that MicroPython code could be run on it.
 
B. Use external PHY device DP83848PHY, connect to STM and setup IP connectivity to the outside world. This is because of the non-availability of Wifi or on-board ethernet port.

C. Configure MQTT telemtery protocol and program the board to send user button clicks. Do the same on Azure IoT hub to receive those clicks.

D. Use Azure IoT Explorer to discover and monitor the MQTT telemetry messages sent from STM board.

.
.
.

.
.
.

.
.
.


A. First Step(followed from https://github.com/micropython/micropython): 

A.1. Download micropython from (https://micropython.org/download/) for Linux based hosts.

A.2. Unzip the downloaded .tar.xz file using "tar -xvf micropython-1.19.1.tar.xz" for example. This gave me a folder "micropython-1.19.1".

A.3. 'make' (meaning build) MicroPython cross-compiler(compiles .py scripts into .mpy files.)

    Before running 'make' in mpy-cross folder, it had below contents.
      ls
      gccollect.c  main.c  Makefile  mpconfigport.h  mphalport.h  mpy-cross.vcxproj  qstrdefsport.h  README.md
      
    After running 'make' in mpy-cross folder, it has now below files/folders
      ls
      build  gccollect.c  main.c  Makefile  mpconfigport.h  mphalport.h  mpy-cross  mpy-cross.map  mpy-cross.vcxproj  qstrdefsport.h  README.md
      
A.4. An ARM compiler is required for the build

    Inside Ubuntu, open a terminal and input
    
       "sudo add-apt-repository ppa:team-gcc-arm-embedded/ppa"
       "sudo apt-get update"
       
    Continue to input to install toolchain
    
        sudo apt-get install gcc-arm-none-eabi
        
A.5.  make(meaning build) the  Micropython for a microcontroller board(BOARD=STM32F4DISC),(ran inside /micropython-1.19.1/ports/stm32/ folder) 

      "make BOARD=STM32F4DISC"  
    
    
    This gave me a new folder 'build-STM32F4DISC' with below files and folders.
      ls
      accel.o            drivers         firmware.elf      i2c.P                machine_i2s.P    modstm_qstr.h     pin_named_pins.o    pyb_can.P                 rtc.P        stm32_it.P            usbd_conf.o
      accel.P            eth.o           firmware.hex      irq.o                machine_spi.o    modutime.o        pin_named_pins.P    pyb_i2c.o                 sdcard.o     storage.o             usbd_conf.P
      adc.o              eth.P           firmware.map      irq.P                machine_spi.P    modutime.P        pin.o               pyb_i2c.P                 sdcard.P     storage.P             usbd_desc.o
      adc.P              extint.o        flashbdev.o       lcd.o                machine_timer.o  mphalport.o       pin.P               pyb_spi.o                 sdram.o      system_stm32.o        usbd_desc.P
      boardctrl.o        extint.P        flashbdev.P       lcd.P                machine_timer.P  mphalport.P       pins_af.py          pyb_spi.P                 sdram.P      system_stm32.P        usbdev
      boardctrl.P        extmod          flash.o           led.o                machine_uart.o   mpnetworkport.o   pins_qstr.h         pybthread.o     s         servo.o      systick.o             usbd_hid_interface.o
      bufhelper.o        factoryreset.o  flash.P           led.P                machine_uart.P   mpnetworkport.P   pins_STM32F4DISC.c  pybthread.P               servo.P      systick.P             usbd_hid_interface.P
      bufhelper.P        factoryreset.P  frozen_content.c  lib                  main.o           mpthreadport.o    pins_STM32F4DISC.o  qspi.o                     shared       timer.o               usbd_msc_interface.o
      build-STM32F4DISC  fatfs_port.o    frozen_mpy        machine_adc.o        main.P           mpthreadport.P    pins_STM32F4DISC.P  qspi.P                     softtimer.o  timer.P               usbd_msc_interface.P
      can.o              fatfs_port.P    gccollect.o       machine_adc.P        modmachine.o     network_lan.o     powerctrlboot.o     resethandler.o             softtimer.P  uart.o                usb.o
      can.P              fdcan.o         gccollect.P       machine_bitstream.o  modmachine.P     network_lan.P     powerctrlboot.P     rfcore.o                   spibdev.o    uart.P                usb.P
      dac.o              fdcan.P         genhdr            machine_bitstream.P  modpyb.o         pendsv.o          powerctrl.o         rfcore.P                   spibdev.P    ulpi.o                usrsw.o
      dac.P              firmware0.bin   help.o            machine_i2c.o        modpyb.P         pendsv.P          powerctrl.P         rng.o                     spi.o        ulpi.P                usrsw.P
      dma.o              firmware1.bin   help.P            machine_i2c.P        modstm.o         pin_defs_stm32.o  py                  rng.P                     spi.P        usbd_cdc_interface.o  wdt.o
      dma.P              firmware.dfu    i2c.o             machine_i2s.o        modstm.P         pin_defs_stm32.P  pyb_can.o           rtc.o                     stm32_it.o   usbd_cdc_interface.P  wdt.P

A.5.  make the mboot(ran inside /micropython-1.19.1/ports/stm32). Meaning build the mbootloader


      "make -C mboot BOARD=STM32F4DISC"


      THis gave below built files
      
      ls (ran under micropython-1.19.1/ports/stm32/mboot/build-STM32F4DISC)
      
      
      drivers  elem.P        firmware.dfu  firmware.hex  fsload.o  genhdr      gzstream.P  main.o  pack.o  pins_af.py          ports     sdcard.P  ui.o           vfs_fat.o  vfs_lfs.o
      elem.o   firmware.bin  firmware.elf  firmware.map  fsload.P  gzstream.o  lib         main.P  pack.P  pins_STM32F4DISC.c  sdcard.o  shared    ui.P           vfs_fat.P  vfs_lfs.P
      
      
      Notice the firmware.dfu file. This Device Firmware Upgrade(DFU) file will be used to write to the firmware( flash the firmware !!)
      
A.6.  Install the dfu utility(dfu-util). This utility will help to flash the above seen .dfu file into the STM board.


      "apt-get install dfu-util
      
      
A.7   Create a file named "49-stmdiscovery.rules" to the following location   "/etc/udev/rules.d/", with following contents.

      # f055:9800 - STM32F4 Discovery running MicroPython in USB Serial Mode (CN5)
      ATTRS{idVendor}=="f055", ATTRS{idProduct}=="9800", ENV{ID_MM_DEVICE_IGNORE}="1"
      ATTRS{idVendor}=="f055", ATTRS{idProduct}=="9800", ENV{MTP_NO_PROBE}="1"
      SUBSYSTEMS=="usb", ATTRS{idVendor}=="f055", ATTRS{idProduct}=="9800", MODE:="0666"
      KERNEL=="ttyACM*", ATTRS{idVendor}=="f055", ATTRS{idProduct}=="9800", MODE:="0666"
      # 0483:df11 - STM32F4 Discovery in DFU mode (CN5)
      SUBSYSTEMS=="usb", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="df11", MODE:="0666"
      
      
   Notice above, Vendor ID and Product ID attributes 0483:df11 respectively, will be used to discover the microcontroller, firmware booted with .dfu file.

   After adding the file. Reload the added rules:
         
      "sudo udevadm control --reload-rules"


A.8   Put the board in DFU mode. 
      To do this, you need to make BOOT0 high(connect physically BOOT0 to VDD(3v) on STM32board), and BOOT1 low(no need to do anything about this), and           reset the board.
      
A.9   List the attached dfu capable devices using:
       
      dfu-util -l
      
      .
      .
      
      
      dfu-util 0.9

      Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
      Copyright 2010-2016 Tormod Volden and Stefan Schmidt
      This program is Free Software and has ABSOLUTELY NO WARRANTY
      Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

      dfu-util: Cannot open DFU device 04f2:b724



Ubuntu machine was rebooted to detect the STM board in dfu mode.

      lsusb
      Bus 006 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
      Bus 005 Device 003: ID 0489:e0cd Foxconn / Hon Hai Wireless_Device
      Bus 005 Device 002: ID 06cb:00bd Synaptics, Inc. Prometheus MIS Touch Fingerprint Reader
      Bus 005 Device 004: ID 0483:374b STMicroelectronics ST-LINK/V2.1
      Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
      Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
      Bus 003 Device 002: ID 0483:df11 STMicroelectronics STM Device in DFU Mode                         << STM in DFU mode
      Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
      Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
      Bus 001 Device 002: ID 04f2:b724 Chicony Electronics Co., Ltd Integrated Camera
      Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub


      dfu-util -l
      
      .
      .
      
      dfu-util 0.9

      Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
      Copyright 2010-2016 Tormod Volden and Stefan Schmidt
      This program is Free Software and has ABSOLUTELY NO WARRANTY
      Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

      dfu-util: Cannot open DFU device 04f2:b724
      Found DFU: [0483:df11] ver=2200, devnum=2, cfg=1, intf=0, path="3-2", alt=3, name="@Device Feature/0xFFFF0000/01*004 e", serial="326237753036"
      Found DFU: [0483:df11] ver=2200, devnum=2, cfg=1, intf=0, path="3-2", alt=2, name="@OTP Memory /0x1FFF7800/01*512 e,01*016 e", serial="326237753036"
      Found DFU: [0483:df11] ver=2200, devnum=2, cfg=1, intf=0, path="3-2", alt=1, name="@Option Bytes  /0x1FFFC000/01*016 e", serial="326237753036"
      Found DFU: [0483:df11] ver=2200, devnum=2, cfg=1, intf=0, path="3-2", alt=0, name="@Internal Flash  /0x08000000/04*016Kg,01*064Kg,07*128Kg", serial="326237753036"
      
A.10  Upload DFU into STM32F4 board using dfu-util with the board in DFU mode as seen in step A.8. Correspoding argument meaning are added below.

      "sudo dfu-util -a 0 -d 0483:df11 -D build-STM32F4DISC/firmware.dfu"
      
                      a(alt-setting=0 (?))
                          -d(device 0483:df11 ,is connected to STM micro-usb port)
                                       -D(download- Write firmware from build-STM32F4DISC/firmware.dfu into device(0483:df11).
                                       
      dfu-util 0.9

      Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
      Copyright 2010-2016 Tormod Volden and Stefan Schmidt
      This program is Free Software and has ABSOLUTELY NO WARRANTY
      Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

      Opening DFU capable USB device...
      ID 0483:df11
      Run-time device DFU version 011a
      Claiming USB DFU Interface...
      Setting Alternate Setting #0 ...
      Determining device status: state = dfuERROR, status = 10
      dfuERROR, clearing status
      Determining device status: state = dfuIDLE, status = 0
      dfuIDLE, continuing
      DFU mode device DFU version 011a
      Device returned transfer size 2048
      DfuSe interface name: "Internal Flash  "
      file contains 1 DFU images
      parsing DFU image 1
      image for alternate setting 0, (2 elements, total size = 334808)
      parsing element 1, address = 0x08000000, size = 14608
      Download	[=========================] 100%        14608 bytes
      Download done.
      parsing element 2, address = 0x08020000, size = 320184
      Download	[=========================] 100%       320184 bytes
      Download done.
      done parsing DfuSe file


      Microcontroller board is listed as a flash drive
      
We could see that the .dfu image was copied successfully into STM32 board.
At this point, Ubuntu could not see the flash drive of the Pyboard. 


      lsblk
      NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
      loop0         7:0    0     4K  1 loop /snap/bare/5
      loop1         7:1    0    62M  1 loop /snap/core20/1611
      loop2         7:2    0  63.2M  1 loop /snap/core20/1623
      loop4         7:4    0 238.1M  1 loop /snap/firefox/1918
      loop5         7:5    0 346.3M  1 loop /snap/gnome-3-38-2004/115
      loop6         7:6    0 346.3M  1 loop /snap/gnome-3-38-2004/119
      loop7         7:7    0  91.7M  1 loop /snap/gtk-common-themes/1535
      loop8         7:8    0  37.1M  1 loop /snap/hunspell-dictionaries-1-7-2004/2
      loop9         7:9    0  45.9M  1 loop /snap/snap-store/592
      loop10        7:10   0  45.9M  1 loop /snap/snap-store/599
      loop11        7:11   0    48M  1 loop /snap/snapd/16778
      loop12        7:12   0    48M  1 loop /snap/snapd/17029
      loop13        7:13   0   284K  1 loop /snap/snapd-desktop-integration/14
      loop14        7:14   0 236.8M  1 loop /snap/firefox/1943
      sda           8:0    1     1M  0 disk /media/abin/DIS_F407VG
      nvme0n1     259:0    0 476.9G  0 disk 
      ├─nvme0n1p1 259:1    0   260M  0 part /boot/efi
      ├─nvme0n1p2 259:2    0    16M  0 part 
      ├─nvme0n1p3 259:3    0 225.7G  0 part 
      ├─nvme0n1p4 259:4    0  1000M  0 part 
      └─nvme0n1p5 259:5    0   250G  0 part /var/snap/firefox/common/host-hunspell
      
This time a reset on the board, by pressing on black reset button helped to get the Pyboard flash get detected in Ubuntu.

      NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
      loop0         7:0    0     4K  1 loop /snap/bare/5
      loop1         7:1    0    62M  1 loop /snap/core20/1611
      loop2         7:2    0  63.2M  1 loop /snap/core20/1623
      loop4         7:4    0 238.1M  1 loop /snap/firefox/1918
      loop5         7:5    0 346.3M  1 loop /snap/gnome-3-38-2004/115
      loop6         7:6    0 346.3M  1 loop /snap/gnome-3-38-2004/119
      loop7         7:7    0  91.7M  1 loop /snap/gtk-common-themes/1535
      loop8         7:8    0  37.1M  1 loop /snap/hunspell-dictionaries-1-7-2004/2
      loop9         7:9    0  45.9M  1 loop /snap/snap-store/592
      loop10        7:10   0  45.9M  1 loop /snap/snap-store/599
      loop11        7:11   0    48M  1 loop /snap/snapd/16778
      loop12        7:12   0    48M  1 loop /snap/snapd/17029
      loop13        7:13   0   284K  1 loop /snap/snapd-desktop-integration/14
      loop14        7:14   0 236.8M  1 loop /snap/firefox/1943
      sda           8:0    1     1M  0 disk /media/abin/DIS_F407VG
      sdb           8:16   1   240K  0 disk 
      └─sdb1        8:17   1   112K  0 part /media/abin/PYBFLASH                                          << second flash drive
      nvme0n1     259:0    0 476.9G  0 disk 
      ├─nvme0n1p1 259:1    0   260M  0 part /boot/efi
      ├─nvme0n1p2 259:2    0    16M  0 part 
      ├─nvme0n1p3 259:3    0 225.7G  0 part 
      ├─nvme0n1p4 259:4    0  1000M  0 part 
      └─nvme0n1p5 259:5    0   250G  0 part /var/snap/firefox/common/host-hunspell


      lsusb
      Bus 006 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
      Bus 005 Device 003: ID 0489:e0cd Foxconn / Hon Hai Wireless_Device
      Bus 005 Device 002: ID 06cb:00bd Synaptics, Inc. Prometheus MIS Touch Fingerprint Reader
      Bus 005 Device 005: ID 0483:374b STMicroelectronics ST-LINK/V2.1                       <<1(connected via mini-usb port on board)
      Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
      Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
      Bus 003 Device 003: ID f055:9800 MicroPython Pyboard Virtual Comm Port in FS Mode      <<2 (THis was in DFU mode, before second reset, micro-usb on board)
      Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
      Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
      Bus 001 Device 002: ID 04f2:b724 Chicony Electronics Co., Ltd Integrated Camera
      Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
      
The Pyboard is seen in Disk Manager.
      
      
![MicroPython pyboard listeds as a drive](https://user-images.githubusercontent.com/24519192/197406211-3e8d27c9-12ca-4e87-b421-78f0b5631ea4.png)

      
      
Executing dfu-util list command, does not show the dfu capable device. This is correct. STM board is no more in DFU mode, it is in FS mode.      
      
      dfu-util -l
      dfu-util 0.9

      Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
      Copyright 2010-2016 Tormod Volden and Stefan Schmidt
      This program is Free Software and has ABSOLUTELY NO WARRANTY
      Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

      dfu-util: Cannot open DFU device 04f2:b724  
      
   
     
A.11  Testing if MicroPython is accessible by turning on a LED on STM32 board.

Knowing the available tty ports is important.

      sudo dmesg | grep tty
      [    0.089570] printk: console [tty0] enabled
      [   47.347642] cdc_acm 5-2:1.2: ttyACM0: USB ACM device
      [10178.960329] cdc_acm 5-2:1.2: ttyACM0: USB ACM device
      [10200.309347] cdc_acm 3-2:1.1: ttyACM1: USB ACM device     <<ACM1 worked 
      
Using rshell to access the board.
Install rshell

      sudo apt-get install rshell
      
Run rshell by using tty port ttyACM1      
      sudo python3 -m rshell.main -p /dev/ttyACM1
      Using buffer-size of 512
      Connecting to /dev/ttyACM1 (buffer-size 512)...
      Trying to connect to REPL  connected
      Retrieving sysname ... pyboard
      Testing if ubinascii.unhexlify exists ... Y
      Retrieving root directories ... /flash/
      Setting time ... Oct 14, 2022 18:34:49
      Evaluating board_name ... pyboard
      Retrieving time epoch ... Jan 01, 2000
      Welcome to rshell. Use Control-D (or the exit command) to exit rshell.


      Entered repl !!!
      /home/abin/Downloads/micropython-1.19.1> repl
      Entering REPL. Use Control-X to exit.
      >
      MicroPython v1.19.1 on 2022-10-14; F4DISC with STM32F407
      Type "help()" for more information.
      >>
      >> 
      >> pyb.LED(1).on()


This turned ON one of the LED in STM32board.


      
      

