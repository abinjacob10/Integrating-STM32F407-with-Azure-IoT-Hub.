# Integrating STM32F407 with Azure IoT Hub.
This README file goes through detailed step by step method to integrate a microcontroller board(STM32F407) with Azure Cloud(IoT Hub).



Please note that this project is still under progress(as of 23rd of October).

The project could be broken down into four major parts. Briefly explained as:

A. Make STM32F407 understand MicroPython, so that MicroPython code could be run on it.
 
B. Use external PHY device DP83848PHY connect to STM and setup IP connectivity to the outside world. This is because of the non-availability of Wifi or on -board ethernet port.

C. Configure MQTT telemtery protocol and program the board to send user button clicks. Do the same on Azure IoT hub to receive those clicks.

D. Use Azure IoT Explorer to discover and monitor the MQTT telmetry messages sent from STM board.



A. First Step(followed from https://github.com/micropython/micropython: 

A.1. Download micropython from (https://micropython.org/download/) for Linux based hosts.

A.2. Unzip the downloaded .tar.xz file using "tar -xvf micropython-1.19.1.tar.xz" for example. This gave me a folder "micropython-1.19.1".

A.3. 'make' (meaning build) MicroPython cross-compiler(compiles .py scripts into .mpy files.)
    Before running 'make' in mpy-cross folder, it had below contents.
      ls
      gccollect.c  main.c  Makefile  mpconfigport.h  mphalport.h  mpy-cross.vcxproj  qstrdefsport.h  README.md
    After running 'make' in mpy-cross folder, it has now below files/folders
      ls
      build  gccollect.c  main.c  Makefile  mpconfigport.h  mphalport.h  mpy-cross  mpy-cross.map  mpy-cross.vcxproj  qstrdefsport.h  README.md
      
A.4 An ARM compiler is required for the build
