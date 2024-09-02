# 🚀 Onethinx Sprouty Workshop 🚀

## 1 Prerequisites
### 1.1 Hardware
1. Onethinx Sprouty
1. Programmer + cables + adapter PCB.

![Sprouty parts needed](https://raw.githubusercontent.com/onethinx/Sprouty_Workshop/main/assets/img/Sprouty_need_500px.jpeg)
---
### 1.2 PSoC Creator
PSoC Creator is the official tool from Infineon to do the chip configuration on the PSoC6
  - Download PSoC Creator [from here](https://drive.google.com/drive/folders/17IZQReRqCk6mNGf5SMYcHy2We6gLfeac?usp=share_link) or from [the official Infineon PSoC Creator page](https://www.infineon.com/cms/en/design-support/tools/sdk/psoc-software/psoc-creator/)
  - Download PDL 3.1.7 [from here](https://drive.google.com/drive/folders/17IZQReRqCk6mNGf5SMYcHy2We6gLfeac?usp=share_link) or from [the official Infineon PSoC Creator page](https://softwaretools.infineon.com/tools/com.ifx.tb.tool.psocperipheraldriverlibrarypdl)
  - Install PSoC Creator (after installation, check the bottom checkbox to continue without registration information)
  - Install PDL 3.1.7
  - Run PSoC Creator and set the freshly installed PDL by `TOOLS` -> `OPTIONS` -> `PDL v3 (PSoC6 devices) location:` -> `C:\Program Files (x86)\Cypress\PDL\3.1.7`

### 1.2 Visual Studio Code + OTX Maestro extension installation
  - Download and install the [Visual Studio Code IDE](https://code.visualstudio.com)
  - Follow the instructions in the [link to install the OTX Maestro Tools & Extension](https://github.com/onethinx/OTX-Maestro/blob/main/README.md#installation)

### 1.3 Sprouty and programming hardware
  - Make sure the programmer, adapter PCB and cable are connected to the Sprouty as shown:<br><br>
![Sprouty, connection](../assets/img/Sprouty_Debugger.jpg)<br>
  - **NOTE**: Do not install the battery when using the debugger.

## 2 Sprouty Project: Chip Configuration, Firmware Coding and Debugging

### 2.1 Sprouty Chip Setup / Configuration Project (uses Windows / PSoC Creator)

1. Download the Sprouty Project [from here](h../assets/Sprouty_StartCode.zip)
1. After downloading, right click the .zip file and select 'Extract All'. Choose a short folder path and make sure the folder ends with `\Sprouty_StartCode` (e.g. `C:\...\Sprouty_StartCode`)
1. Open (double click) `Onethinx_Creator.cyprj` inside the project folder `..\Sprouty_StartCode\Onethinx_Creator.cydsn`. PSoC Creator will open<br>
![PSoCCreator_WorkspaceExplorer](../assets/img/PSoCCreator_WorkspaceExplorer.png)<br>
1. Open (double click) `TopDesign.cysch` and view the internal wiring of the PSoC6, causing the LED to flash
1. Open `Pins` from the Design Wide Resources to configure the LED IO pin
1. Watch the schematic and find out the right IO pin the LED is connected with<br>
![LED pin](../assets/LEDpin.png)<br>
1. Click the dropdown arrow and select the IO pin we just found for the LED<br>
![PinConfig](../assets/PinConfig.png)<br>
1. Now the LED is configured for the right IO, the OTX / PSoC6 configuration project is ready to be built
1. Hit the Build Icon in the toolbar or select `Build >> Build Onethinx_Creator (Shift + F6)`<br>
![Build](../assets/Build.png)<br>
1. Wait for PSoC Creator to build the configuration project<br>
![Output](../assets/Output.png)<br>
1. The message `Build Succeeded` will appear when building is ready without issues
1. Chip configuration is done, PSoC Creator may now be closed.

### 2.2 Firmware Coding and Debugging (uses Visual Studio Code)

1. Start Visual Studio Code
2. Open the Sprouty folder (not the .zip file)
<br>

![VScode Open](../assets/SelectingFolder.png)

<br>
3. As the project has not run on the PC before, it needs to be Clean-Reconfigured before it can be built. Hit the `Clean-Reconfigure` button from the status bar at the bottom of VS Code<br>

![Clean Build Launch](../assets/Clean_Build_Launch.png)

<br>
5. After successfull configuration (the terminal window will show: "Terminal will be reused by tasks, press..." when ready), the project can be Built and Launched from the debugger. 
  Make sure the Sprouty is connected to the debugger and PC before launching the debug session
  Hit the `Build-And-Launch` button from the status bar at the bottom of VS Code
6. Press Select programmer and choose CMSIS DAP. If you have a different programmer, choose it. If you received your programmer with Sprouty, select CMSIS DAP.
   
![Select Programmer](../assets/SelectingProgrammer.png)

6. Cross fingers and hopefully the firmware will be programmed, the project will enter debug mode (yellow bar) and the blue LED will be flashing🎉
 <br>

![Succeeded](../assets/Succeeded.gif)

<br>
7. Congratulations!<br>🤓
