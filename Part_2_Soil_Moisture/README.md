# Onethinx Sprouty Workshop

## Part 2: Soil Moisture.

To measure soil moisture, we designed a capacitor using the traces of the PCB. When the moisture changes around the traces, the capacitance changes. In order to measure this capacitance change, we created an oscillator that uses an (internal) operational amplfilier to output a signal with frequency directly connected to the capacitance of the soil traces. The frequency of the signal changes when the moisture around the traces changes, giving us the opportunity to measure the frequency of the signal to determine moisture levels. 

In order to count the pulses, we use an integrated Timer-Counter block. Since the Counter counts pulses, we need to change the frequency we get from the oscillator, to pulses which the counter can count. The general idea looks something like this:

![Sprouty Moisture](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/HowSpoutyMoistureWorks.png)

If you go back to the PSoC Creator, we need to add these blocks and connect them.

Firstly, on the right side, you can drag and drop the following:
* 3 Analog Pins
* OpAmp
* Low Power Comparator
* VRef
* Timer Counter
* Clock

![Part 2 Components to use](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/P2Components.png)

Now that we have dragged the components, we need to connect and modify them. The following image shows the components.

![Part 2 Dragged](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/P2Dragged.png)


We use 3 Analog pins:
* 1 analog pin is used for Operational Amplifier Positive Terminal.
* 1 analog pin is used for Operational Amplifier Negative Terminal.
* 1 analog pin is used for Operational Amplifier Output Terminal.

Setup analog pins (double click on each pin):
* Name: OA_PLUS,  ✓ Analog, High Impedance Analog (P9_0)
* Name: OA_MINUS, ✓ Analog, High Impedance Analog (P9_1)
* Name: OA_OUT,   ✓ Analog, High Impedance Analog (P9_2)

![SCH setup opamp](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/Sprouty_Basic_SCH.png)

Modify the Opamp, double click on it and modify:
**Name:** Opamp
**Output Drive:** Output to pin
Press Apply and OK to save the configuration.

![Part 2 OpAmp](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/Config_OpAmp.png)

Modify the Comparator, double click on it and modify:
**Name:** LPComp
**Power/Speed:** Normal Power/Fast
Press Apply and OK to save the configuration.

![Part 2 LPComp](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/Config_LPComp.png)

Modify the Counter, double click on it and modify:
**Basic Tab**
* **Name:** TCounter
* **Resolution:** 32-bits
* **Period:** 4000000
**Inputs Tab**
* **Count Input:** Rising Edge
Press Apply and OK to save the configuration.

![Part 2 TC1](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/Config_Counter1.png)
![Part 2 TC2](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/Config_Counter2.png)

Modify the Clock, double click on it and modify:
**Frequency:** 8 MHz
Press Apply and OK to save the configuration.

Once you have all the components modified, you should connect them like this:

![Part 2 Connections](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/P2Connections.png)

Save the configuration by pressing Ctrl + S or File -> Save.

Now open the **Pins** again. You do that again by double clicking on the **Pins** on the left pane "Workspace Explorer".

You can now set the following pins:
* OA_PLUS,  = P9_0
* OA_MINUS, = P9_1
* OA_OUT,   = P9_2

Finally, all pins should look like this:

![Part 2 All pins connected](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/Config_Pins.png)

Save the configuration by pressing Ctrl + S or File -> Save.

Build the configuration by pressing Shift + F6 or by pressing Build -> Build Onethinx_Creator. 

Now you can go back to the VSC (Visual Studio Code) and do the Clean-Reconfigure.

Firstly, we need to add an extra global variable called **count** just before main.

```c
uint32_t 		count;
```

Next, in main function, after the ADC_Start();, we can add all the remaining init of the components (Start), which should look like this:

```c
ADC_Start();	
Opamp_Start();	
LPComp_Start();	
TCounter_Start();
```

Next, we need to count the pulses. In code we do this by:
1. Reset the Timer-Counter block to zero
2. Start the Counter
3. Wait for 1 second to count the fequency
4. Capture the count after 1 second
5. Wait for 1 ms while that count is being saved
6. Get the count from the counter
7. Restore the counter

In code, it looks like this:

```c
TCounter_SetCounter(0);			
TCounter_TriggerStart();		
CyDelay(1000);					
TCounter_TriggerCapture();		
CyDelay(1);						
count = TCounter_GetCapture();	
TCounter_TriggerReload();	
```

Now the count of the frequency should be save in the variable **count**.

Your code should look something like this:

```c
int16_t 		tempAir, tempSoil, valueLight;
uint32_t 		count;

int main(void)
 {
	CyDelay(2000);
	__enable_irq();

	Cy_GPIO_Write(LED_R_PORT, LED_R_NUM, 1);					// Turn off the red LED
	Cy_GPIO_Write(LED_G_PORT, LED_G_NUM, 1);					// Turn off the green LED
	Cy_GPIO_Write(LED_B_PORT, LED_B_NUM, 1);					// Turn off the blue LED

	ADC_Start();												 
    Opamp_Start();												
	LPComp_Start();												
	TCounter_Start();											
	
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
		
		TCounter_SetCounter(0);									
		TCounter_TriggerStart();								
		CyDelay(1000);											
		TCounter_TriggerCapture();								
		CyDelay(1);												
		count = TCounter_GetCapture();							
		TCounter_TriggerReload();						
	}
}
```

This is it for the Part 2.
