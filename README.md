# ESP8266 Driver to STM32 with UART Interruptions

AT Command driver to use the ESP8266. This driver was focused in a STM32 but it can be used with any UART Connection who supports C coding with small changes. The driver was developed taking count the interruptions possible in the UART of the STM32 Microcontroller. The ESP8266 driver was developed based on the old version driver of [nimaltd](https://github.com/nimaltd/ESP8266), without the guidance of this version, it would not be possible to make it. Thank you nimaltd.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

* STM32 Microcontroller with the debugger to make the connection with a computer.
* STM32 CubeMX to create the project to install in the microcontroller. It can be downloaded in the [official site](https://www.st.com/en/development-tools/stm32cubemx.html).
* An IDE who supports the STM32 and works with the code generated by STM32CubeMX. In my case I use the SW4STM32. Download in the [official site](https://www.st.com/en/development-tools/sw4stm32.html).

### Installing

* Open STM32 CubeMX and create a project with the exact version of the STM32 Microcontroller that you are using. 
* Config your USART port and enable interrupt on CubeMX.
* Add the header file `#include <ESP8266.h>` in the `main.c` file of the created code by CubeMX.
* Config your ESP8266Config.h file with the constants that you believe are better for your own project. Please make sure that you change ALL the parameters because there are some buttons (ENABLE and RST) in the ESP8266, which are necessary to start the module. 
* Add `Wifi_RxCallBack()` on **USART interrupt** routine. It is important that you search in the official documentation of your device, which are the steps to initiate the UART Interruptions. Normally, the start code from CubeMX is not complete. 
* If you make all those changes and the Code does not have an error in the compilation process, you can start the module using the next function in this order to initiate the module:

```C
// Start the ESP8266 module to start a UART communication
Wifi_Enable();

// Initiation of the WiFi
// As you can see, the boolean variables is to make sure of 
// a correct process inside of every function
if(Wifi_Init()==false)    
	return false;
```
* If there is no error in the compilation process until this moment, you can make a small code with those lines to test if the ESP8266 was intiaite (you can see that in the power LED and the TX and RX status). 
* With the successful connection of the ESP8266 module, you can use the function that you see in the header `ESP8266.h` to make a connection with a WiFi network or create one using the needed commands. To make sure a successful connection, please refer to the datasheet of the module. 

## Deployment

I am going to explain how the driver works and how you could create your own functions inside of the `ESP9266.c` file.

First of all, it uses the boolean type to ensure that the process inside of the function were finished without a problem. If you look inside of the created functions, they use a *timeout* for the response of the ESP8266 module, when this connection were not successful for any reason, the return value would be *false*. This makes easier the debugging process and it is one big help from the old version from nimaltd. 

All the functions have a similar construction like this one: 
```C
if(Wifi_SendString((char*)Wifi.TxBuffer)==false)
	break;
if(Wifi_WaitForString(_WIFI_WAIT_TIME_VERYHIGH,&result,3,"\r\nOK\r\n","\r\nERROR\r\n")==false)
	break;
if(result > 1)		// If the result is higher to 1 is because there were an error
	break;		// in the communication
```
It uses an internal function `Wifi_SendString` to send the array `TX_Buffer` through the UART port. Then it uses another function `Wifi_WaitForString` to make sure that there was one of the responses in the `RX_Buffer` that there are inputs of the function. Finally, if the variable *result* is greater than 1, it means that the response was “ERROR”. In that case, it was an error in the process and the function would return *false*. 

This is the big idea of how all the functions were created, according to your needs, you can create your own functions based on this model and the datasheet for the *AT Commands*, however, the most useful functions were already created.

## Authors

* **Jairo Mejia** - *Work for the new version without FreeRTOS* - [jrmejiaa](https://github.com/jrmejiaa)
* **Nima Askari** - *Initial work* - [nimaltd](https://github.com/nimaltd)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE) file for details

## Acknowledgments

I would like to thank again Nima Askari for your contributions with the first version of this project. I have improved my skills in C using his code as template for this version and for another projects. 
