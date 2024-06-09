# Onethinx Sprouty Workshop

## Part 3: LoRaWAN (and Sleep)

The LoRaWAN and Sleep part will no longer requre PSoC Creator to setup. The whole configuration is dont in VSCode.

When you opened the VSCode Sprouty Starting Project, you saw some structures in the begginng:

```c
coreConfiguration_t	coreConfig = {					
	.Join.KeysPtr = 			&TTN_OTAAkeys,		
	.Join.DataRate =			DR_AUTO,			
	.Join.Power =				PWR_MAX,			
	.Join.MAXTries = 			5,					
	.Join.SubBand_1st =     	EU_SUB_BANDS_DEFAULT,
	.Join.SubBand_2nd =     	EU_SUB_BANDS_DEFAULT,
	.TX.Confirmed = 			false,				
	.TX.DataRate = 				DR_ADR,				
	.TX.Power = 				PWR_ADR,			
	.TX.FPort = 				1,					
	.RX.Boost = 				true,				
	.System.Idle.Mode = 		M0_DeepSleep,		
	.System.Idle.BleEcoON =		false,				
	.System.Idle.DebugON =		true				
};
```

In this structure you configure the LoRaWAN Stack. If you are new to LoRaWAN, you do not need to modify it, but we encurage you to study it a bit after the workshop has finished.

in the same folder as the main.c, you can see a folder called OnethinxCore. If you expand this folder, you will see a header file called LoRaWAN_keys.h. This is where we save the LoRaWAN keys. You can double click it to modify it.

![Part 3 Keys](https://github.com/onethinx/Sprouty_Workshop/blob/main/img/P3Keys.png)

In the following structure you can configure the Sleep parameters. For now, you do not need to modify it, but we encurage you to study it a bit after the workshop has finished.

```c
sleepConfig_t sleepConfig=							
{
	.sleepMode = modeDeepSleep,						
	.BleEcoON = false,								
	.DebugON = true,								
	.sleepCores = coresBoth,						
	.wakeUpPin = wakeUpPinHigh(true),				
	.wakeUpTime = wakeUpDelay(0, 0, 2, 0) 			
};
```

To go forward we need 2 new global variables:
* coreStatus_t coreStatus;
* uint8_t data[4]

Variable coreStatus is good to have for easier debugging. Each LoRaWAN API call returns coreStatus_t data which is helpful for debugging. In the **data** array, we will save the data which we will later send via LoRaWAN.

The global variables should look something like this by now:

```c
coreStatus_t 	coreStatus;
int16_t 		tempAir, tempSoil, valueLight;
uint32_t 		count;
uint8_t 		data[4];
```

Next, in main, after the component initialisations, we should Init the LoRaWAN stack by giving it the coreConfiguration structure (the LoRaWAN configuration structure above). We whould also Join the LoRaWAN network (The LoRaWAN Join function accepts the type of wait we want to perform, or no wait. In this case, we use M4_WaitDeepSleep which makes your program wait in low power for the LoRaWAN stack to join the network). In case we do not join the network after a given amount of tries, we should do something about it. We will turn on the Red LED and stay in an endless while loop.

```c
coreStatus = LoRaWAN_Init(&coreConfiguration);
coreStatus = LoRaWAN_Join(M4_WaitDeepSleep);
if (!coreStatus.mac.isJoined)
{
	Cy_GPIO_Write(LED_R_PORT, LED_R_NUM, 1);
	while(1){}
}
```

Now that we have Initialized the LoRaWAN stack and joined the LoRaWAN network, we should parse the data so we can send it. After reading the data from the ADC and the Counter, we will use the **data** array to save the data to send. Since the data is an array of 4 elements of type uint8_t (4 x 8 bits), we will need to compress the data we got in order to send it. 

* **Data 1** - Moisture data in Counts (Frequency). It goes between 10,000 Hz and 100,000 Hz. Dividing it by 1000, we get kHz, from 10 to 100.
* **Data 2** - Soil Temperature Data in Celsius (resolution 1°C). This is expected to be in range between -40°C and +100°C (extremes). Substract by 40 on the server side to get the real temperature. 
* **Data 3** - Air Temperature Data in Celsius (resolution 1°C). This is expected to be in range between -40°C and +100°C (extremes). Substract by 40 on the server side to get the real temperature.
* **Data 4** - Light mV value which we divide by 20 in order for it to fit in 8 bits (uint_8).

```c
data[0] = (uint8_t)(count/1000);	
data[1] = (uint8_t)tempSoil+40;		
data[2] = (uint8_t)tempAir+40;		
data[3] = (uint8_t)(valueLight/20);	
```

Now we have a **data** array of uint8_t of size 4. We can now send this array by calling LoRaWAN_Send, giving it the data array address, size we want to send, and wait type:

```c
LoRaWAN_Send(&data, 4, M4_WaitDeepSleep);
```

After sending the data, we place the module in sleep by calling LoRaWAN_Sleep (giving it the sleep configuration from earlier/above).

```c
LoRaWAN_Sleep(&sleepConfig);
```

You can now **build** and **launch** your code.

In the end, your code should look something like this:

```c
#include "project.h"
#include "OnethinxCore01.h"
#include "OnethinxExt01.h"
#include "LoRaWAN_keys.h"
#include "cyfitter_cfg.h"
#include "sprouty.h"

coreConfiguration_t	coreConfig = {					
	.Join.KeysPtr = 			&TTN_OTAAkeys,		
	.Join.DataRate =			DR_AUTO,			
	.Join.Power =				PWR_MAX,			
	.Join.MAXTries = 			5,					
	.Join.SubBand_1st =     	EU_SUB_BANDS_DEFAULT
	.Join.SubBand_2nd =     	EU_SUB_BANDS_DEFAULT
	.TX.Confirmed = 			false,				
	.TX.DataRate = 				DR_ADR,				
	.TX.Power = 				PWR_ADR,			
	.TX.FPort = 				1,					
	.RX.Boost = 				true,				
	.System.Idle.Mode = 		M0_DeepSleep,		
	.System.Idle.BleEcoON =		false,				
	.System.Idle.DebugON =		true				
};

sleepConfig_t sleepConfig=							
{
	.sleepMode = modeDeepSleep,						
	.BleEcoON = false,								
	.DebugON = true,								
	.sleepCores = coresBoth,						
	.wakeUpPin = wakeUpPinHigh(true),				
	.wakeUpTime = wakeUpDelay(0, 0, 2, 0) 			
};

coreStatus_t 	coreStatus;
int16_t 		tempAir, tempSoil, valueLight;
uint32_t 		count;
uint8_t 		data[4];

int main(void)
 {
	CyDelay(2000);
	__enable_irq();

	Cy_GPIO_Write(LED_R_PORT, LED_R_NUM, 1);		
	Cy_GPIO_Write(LED_G_PORT, LED_G_NUM, 1);		
	Cy_GPIO_Write(LED_B_PORT, LED_B_NUM, 1);		

	Opamp_Start();									
	LPComp_Start();									
	TCounter_Start();								
	ADC_Start();									
 
	coreStatus = LoRaWAN_Init(&coreConfig);			
    coreStatus = LoRaWAN_Join(M4_WaitDeepSleep);				/

	if (!coreStatus.mac.isJoined)							
	{
		Cy_GPIO_Write(LED_R_PORT, LED_R_NUM, 1);
	 	while(1){}
	}

	for(;;)
	{
		Cy_GPIO_Write(LED_G_PORT, LED_G_NUM, 0);				
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
		Cy_GPIO_Write(LED_G_PORT, LED_G_NUM, 1);				

		data[0] = (uint8_t)(count/1000);						 
		data[1] = (uint8_t)tempSoil+40;							
		data[2] = (uint8_t)tempAir+40;							
		data[3] = (uint8_t)(valueLight/20);						
		
		LoRaWAN_Send(&data, 4, M4_WaitDeepSleep);			
		LoRaWAN_Sleep(&sleepConfig);							
	}
}
```

This is it.
