# Integrating STM32F407 with Azure IoT Hub.
This README file goes through detailed step by step method to integrate a microcontroller board(STM32F407) with Azure Cloud(IoT Hub).



Please note that this project is still under progress.

The project could be broken down into four major parts. Briefly explained as:

1. Make STM32F407 understand MicroPython, so that MicroPython code could be run on it.
 
2. Use external PHY device DP83848PHY connect to STM and setup IP connectivity to the outside world. This is because of the non-availability of Wifi or on -board ethernet port.

3. Configure MQTT telemtery protocol and program the board to send user button clicks. Do the same on Azure IoT hub to receive those clicks.

4. Use Azure IoT Explorer to discover and monitor the MQTT telmetry messages sent from STM board.
