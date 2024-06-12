# Onethinx Sprouty Workshop

## Part 1: Temperature and Light.

In the first part of the Workshop, we will implement the reading of the soil temperature, air temperature, light intensity. The temperatures are calculated from the NTC and the light intensity is calculated from the photoresistor. For this functionality we will us the analog block.

Firstly, download or clone Sprouty_StartCode repository from Onethinx Github page. You can open this folder in Visual Studio Code. 

To open a folder in Visual Studio Code, start by launching the application. Once it's open, click on File in the top menu and select Open Folder... from the dropdown menu. Alternatively, you can use the keyboard shortcut Ctrl + K followed by Ctrl + O on Windows/Linux or Cmd + K followed by Cmd + O on Mac. In the file dialog that appears, navigate to the folder you want to open, select it, and click Select Folder on Windows/Linux or Open on Mac. The folder will then open in Visual Studio Code, and you will see its contents in the Explorer pane on the left side.

Next, inside Visual Studio Code, navigate to the main.c file and double click it (main.c is located inside the source folder). This is where you write your code. You can now minimize the Visual Studio Code as we will come back later.

In your project folder, go in the **Onethinx_Creator.cydsn** folder and double click the **Onethinx_Creator.cyprj**. PSoC Creator will open (If pop up appears, you can click "Register Later"). Now, you can open the TopDesign by double clicking the **TopDesign.cysch** (Source) in the Workspace Explorer on the left side. This is where we do the hadware setup.

From the right pane (Component Catalog, Cypress), navigate to **Analog** -> **ADC** -> **Scanning SAR ADC** and drag it to the work area. Next, navitate to **Ports and Pins** and drag 3 **Analog Pin**s. Lastly, in the same **Ports and Pins** drag 4 **Digital Output Pin**s.

![PsoC Creator Part 1 Setup](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/P1Setup.png)

We use 3 Analog input pins:
* 1 analog pin is used for measuring the voltage of the voltage divider for the **air NTC**.
* 1 analog pin is used for measuring the voltage of the voltage divider for the **soil NTC**.
* 1 analog pin is used for measuring the voltage of the voltage divider for the **photoresistor**.

We use 4 Digital output pins:
* 1 digital output pin is used for powering the **Green LED**
* 1 digital output pin is used for powering the **Blue LED**
* 1 digital output pin is used for powering the **Red LED**
* 1 digital output pin is used for powering the resistor dividers of the sensors above. Before reading any of the analog pins, we will have to turn this pin **HIGH** and after we are done, we will turn this pin **LOW**. This done so when the device is in low power mode (sleep), it does not consume energy, as during sleep, we do not need these measurements. In short, we save power.

The schematic for the analog part looks like this:

![PsoC Creator Part 1 Analog Setup](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/P1Analog.png)

Setup digital output pins (double click on each pin):
* Name: LED_R, ✓ Digital output, Strong Drive (P12_5)
* Name: LED_G, ✓ Digital output, Strong Drive (P12_4)
* Name: LED_B, ✓ Digital output, Strong Drive (P10_3)
* Name: SPWR,  ✓ Digital output, Strong Drive (P11_5)
(Press Apply and OK for each to save the configuration.)

Setup analog pins (double click on each pin):
* Name: NTC_AIR,  ✓ Analog, High Impedance Analog (P10_1)
* Name: NTC_SOIL, ✓ Analog, High Impedance Analog (P10_0)
* Name: LSENS,    ✓ Analog, High Impedance Analog (P10_2)

We will set up the ADC component (double click on it): <br>
**Name**: ADC<br>
**Free-run scan rate (SPS)**: 1000<br>
**Vref select**: Vdda<br>
**Vneg for S/E**: Vssa<br>
**S/E result format**: Signed<br>
**Samples averaged**: 8<br>
**Number of Channels**: 3<br>
* Ch. 0: Single ended, Avg ✓<br>
* Ch. 1: Single ended, Avg ✓<br>
* Ch. 2: Single ended, Avg ✓<br>

Press Apply and OK to save the configuration.

You can now connect The analog pins to the ADC and you should have something like this:
* NTC Soil = Channel 0
* NTC Air = Channel 1
* Light = Channel 2

Save the configuration by pressing Ctrl + S or File -> Save.

![PsoC Creator Part 1 Done](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/P1Done.png)

Now that you have the hardware configuration done, we just need to connect these pins, to the actual pins of the microcontroller. You do that by double clicking on the **Pins** on the left pane "Workspace Explorer". There you can set the:
* LED_R     = P12_5
* LED_G     = P12_4
* LED_B     = P10_3
* SPWR      = P11_5
* NTC_AIR   = P10_1
* NTC_SOIL  = P10_0
* LSENS     = P10_2

![Part 1 Pins](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/P1Pins.png)

Save the configuration by pressing Ctrl + S or File -> Save.

Build the configuration by pressing Shift + F6 or by pressing Build -> Build Onethinx_Creator. 

Now that you have it built, it generated the API which can be used in Visual Studio Code. You can now open the Visual Studio Code which you previously minimised. In Visual Studio Code, in your project, you need to press "Clean-Reconfigure" in order for the program to recognize the newly generate API.

If you are having dificulty, you can see this link which will show you how to buld, clean reconfigure and run your project: https://github.com/onethinx/FarmBug_Workshop_Introduction

Now that you have your main.c file opened in Visual Studio Code (VSC for short), you can start editing it. Before main, we can create some global variables:

```c
int16_t 		tempAir, tempSoil, valueLight;
```

In main, after the enable_irq() function, you can start the ADC by writing:

```c
ADC_Start();
```

What we need to do in the for loop, every time we need to read the measurement:
1. We need to turn on the power so we can measure the voltages using the ADC. (Having a 1ms delay is good so the pin has time to pull up)
2. Start ADC conversion
3. Wait for the conversion to finish
4. Get the voltage from Channel 0 and calculate soil temperature with the help of a NTC table
5. Get the voltage from Channel 1 and calculate air temperature with the help of a NTC table
6. Get the voltage from Channel 3 to get light intensity
7. Turn off the power to these "sensors".

The temperature conversion from the voltage read from the ADC to the Celsius value is done using the **NTC table** found in **sprouty.h**.

This is how it looks like in code:

```c
Cy_GPIO_Write(SPWR_PORT, SPWR_NUM, 1);					
CyDelay(1);												
ADC_StartConvert();										
ADC_IsEndConversion(CY_SAR_WAIT_FOR_RESULT);			
tempSoil = NTC_table[ (ADC_GetResult16(0) >> 3)];		
tempAir  = NTC_table[ (ADC_GetResult16(1) >> 3)];		
valueLight = ADC_CountsTo_mVolts(2, ADC_GetResult16(2));
Cy_GPIO_Write(SPWR_PORT, SPWR_NUM, 0);					
```

If we debug this code, we should get:
* Soil Temperature in degrees Celsius (1°C resolution)
* Air Temperature in degrees Celsius (1°C resolution)
* Light intensity in mV (by testing, we can set what is a low, and what is high value)

This is how your main code will look like:

```c
int16_t 		tempAir, tempSoil, valueLight;

int main(void)
 {
	CyDelay(2000);
	__enable_irq();

	Cy_GPIO_Write(LED_R_PORT, LED_R_NUM, 1);
	Cy_GPIO_Write(LED_G_PORT, LED_G_NUM, 1);
	Cy_GPIO_Write(LED_B_PORT, LED_B_NUM, 1);

	ADC_Start();

	for(;;)
	{					
		Cy_GPIO_Write(SPWR_PORT, SPWR_NUM, 1);							
		CyDelay(1);														
		ADC_StartConvert();												
		ADC_IsEndConversion(CY_SAR_WAIT_FOR_RESULT);					
		tempSoil = NTC_table[ (ADC_GetResult16(0) >> 3)];				
		tempAir  = NTC_table[ (ADC_GetResult16(1) >> 3)];				
		valueLight = ADC_CountsTo_mVolts(2, ADC_GetResult16(2));		
		Cy_GPIO_Write(SPWR_PORT, SPWR_NUM, 0);							
	}
}
```

This is it for Part 1.
