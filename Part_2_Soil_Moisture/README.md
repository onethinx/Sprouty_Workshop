# 🚀 Onethinx Sprouty Workshop 🚀

## Part 2: Soil Moisture.

To measure soil moisture, we designed a capacitor using the traces of the PCB. When the moisture changes around the traces, the capacitance changes. In order to measure this capacitance change, we created an oscillator that uses an (internal) operational amplfilier to output a signal with frequency directly connected to the capacitance of the soil traces. The frequency of the signal changes when the moisture around the traces changes, giving us the opportunity to measure the frequency of the signal to determine moisture levels. 

In order to count the pulses, we use an integrated Timer-Counter block. Since the Counter counts pulses, we need to change the frequency we get from the oscillator, to pulses which the counter can count. The general idea looks something like this:

![Sprouty Moisture](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/HowSpoutyMoistureWorks.png)

If you go back to the PSoC Creator, we need to add these blocks and connect them.

Firstly, on the right side, you can drag and drop the following:
* 3 Analog Pins
* OpAmp
* Low Power Comparator
* VRef
* Timer Counter
* Clock

![Part 2 Components to use](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/P2Components.png)

Now that we have dragged the components, we need to connect and modify them. The following image shows the components.

![Part 2 Dragged](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/P2Dragged.png)


We use 3 Analog pins:
* 1 analog pin is used for Operational Amplifier Positive Terminal.
* 1 analog pin is used for Operational Amplifier Negative Terminal.
* 1 analog pin is used for Operational Amplifier Output Terminal.

Setup the analog pins:
| Name     | Analog | Drive Type              | Pin   |
|----------|--------|-------------------------|-------|
| OA_PLUS  | ✓      | High Impedance Analog   | P9_0  |
| OA_MINUS | ✓      | High Impedance Analog   | P9_1  |
| OA_OUT   | ✓      | High Impedance Analog   | P9_2  |


![SCH setup opamp](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/Sprouty_Basic_SCH.png)

Configure the Opamp:
| Property     | Value              |
|--------------|--------------------|
| Name.        | Opamp              |
| Output Drive | Output to pin      |

![Part 2 OpAmp](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/Config_OpAmp.png)

Set up the Comparator:
| Property     | Value              |
|--------------|--------------------|
| Name         | LPComp             |
| Power/Speed  | Normal Power/Fast  |


![Part 2 LPComp](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/Config_LPComp.png)

Modify the Counter
`Basic Tab`
| Property     | Value    |
|--------------|----------|
| Name         | TCounter |
| Resolution   | 32-bits  |
| Period       | 4000000  |

![Part 2 TC1](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/Config_Counter1.png)

`Inputs Tab`
| Property    | Value        |
|-------------|--------------|
| Count Input | Rising Edge  |

![Part 2 TC2](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/Config_Counter2.png)

Configure the Clock:
| Property   | Value  |
|------------|--------|
| Frequency  | 8 MHz  |

Once you have all the components modified, you should connect them like this:

![Part 2 Connections](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/P2Connections.png)

Now open the `Pins` again. You do that again by double clicking on the `Pins` on the left pane `Workspace Explorer`.

You can now set the following pins:
| Name      | Pin   |
|-----------|-------|
| OA_PLUS   | P9_0  |
| OA_MINUS  | P9_1  |
| OA_OUT    | P9_2  |

Finally, all pins should look like this:

![Part 2 All pins connected](https://github.com/onethinx/Sprouty_Workshop/blob/main/assets/img/Config_Pins.png)

Save the configuration by pressing Ctrl + S or File -> Save.

Generate the configuration by hitting the `Generate Application` icon in the toolbar or select `Build` >> `Generate Application`

Now you can go back to the VS Code and do `Clean-Reconfigure` to include the newest API files in the build.

Firstly, we need to add an additional global variable called `count` just before main.

```c
uint32_t 		count;
```

Next, in main function, after `ADC_Start();`, we can add all the remaining init of the components (Start), which should look like this:

```c
ADC_Start();	
Opamp_Start();	
LPComp_Start();	
TCounter_Start();
```

Next, we need to count the pulses. In code we do this by:
1. Reset the Timer-Counter block to zero
1. Start the Counter
1. Wait for 1 second to count the fequency
1. Capture the count after 1 second
1. Wait for 1 ms while that count is being saved
1. Get the count from the counter
1. Restore the counter

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

Now the count of the frequency should be saved in the variable `count`.

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

That is it for Part 2. You can find [Part 3: LoRaWAN implementation (and Sleep)](../Part_3_LoRaWAN) over here.<br><br>
*NOTE: if you are experiencing issues, you may contact us directly at* [our Discord channel](https://discord.gg/CvzZwXDk)<br>
