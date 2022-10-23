# Integrating STM32F407 with Azure IoT Hub.
This README file goes through detailed step by step method to integrate a microcontroller board(STM32F407) with Azure Cloud(IoT Hub).

.
.


Please note that this project is still under progress(as of 23rd of October).

The project could be broken down into four major parts. Briefly explained as:

A. Make STM32F407 understand MicroPython, so that MicroPython code could be run on it.
 
B. Use external PHY device DP83848PHY connect to STM and setup IP connectivity to the outside world. This is because of the non-availability of Wifi or on -board ethernet port.

C. Configure MQTT telemtery protocol and program the board to send user button clicks. Do the same on Azure IoT hub to receive those clicks.

D. Use Azure IoT Explorer to discover and monitor the MQTT telmetry messages sent from STM board.

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
///
