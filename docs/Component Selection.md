---
title: Component Selection
---

# RGB Sensor:

| Manufacturer | Part Number| Image | Pros | Cons |
|--------------|------------|-------|------|------|
| Texas Instruments | [OPT4060DTSR](https://www.digikey.com/en/products/detail/texas-instruments/OPT4060DTSR/22116843) | ![TI](../images/TI_OPT4060DTSR.jpg) | - Configurable I2C Address <br> - Small form factor <br> - Low operating power| - Complex configuration <br> |
| Vishay Semiconductor | [VEML3328](https://www.digikey.com/en/products/detail/vishay-semiconductor-opto-division/VEML3328/10673129) | ![VS](../images/VS_VEML3328.jpg) | - Simple configuration <br> - 4-Channels (RGBC) <br> - Integrated signal conditioning | - Fixed I2C address |
| ams-OSRAM USA INC. | [AS7341-DLGM](https://www.digikey.com/en/products/detail/ams-osram-usa-inc/AS7341-DLGM/9996230) | ![AMS](../images/AMS_AS7341-DLGM.jpg) | - 11-Channel full spectrum analysis <br> - Configurable flicker detection | - Very complex configuration <br> - Expensive|  

**Choice: Texas Instruments OPT4060DTSR**  
The TI OPT4060DTSR will allow for a high degree of accuracy and flexibility at a reasonable price. The sensor provides 4 channels for red, green, blue, and white light as well as an auto ranging feature. Power requirements are low, requiring only 1.6V to 3.3V to function with an operating current of 24 μA. The small form factor (2.1mm x 1.9mm x 0.6mm), low PPU ($2.88/Unit) and configurable I2C allow for multiple sensors to be implemented with ease if required.  


# Voltage Regulator

| Manufacturer | Part Number| Image | Pros | Cons |
|--------------|------------|------ | ---- |------|
| Analog Devices Inc. | [ADP2302ARDZ-3.3-R7](https://www.digikey.com/en/products/detail/analog-devices-inc/ADP2302ARDZ-3-3-R7/2606526) | ![AD](../images/AD_ADP2302ARDZ.jpg) | - Wide operating voltage range <br> - High current limit | - Low efficiency at 5Vin/3.3Vout <br> - Expensive|
| Texas Instruments | [LM2674MX-3.3/NOPB](https://www.digikey.com/en/products/detail/texas-instruments/LM2674MX-3-3-NOPB/366902) | ![TI](../images/TI_LM2674MX.jpg) | - Wide operating voltage range <br> - High efficiency at 5Vin/3.3Vout| - Low current limit <br> - Moderately expensive |
| Microchip Technology | [MIC4680-3.3YM-TR](https://www.digikey.com/en/products/detail/microchip-technology/MIC4680-3-3YM-TR/1030018) | ![MC](../images/MT_MIC4680.jpg) | - Wide operating voltage range <br> - Moderate current lmit <br> - Affordable| - Low efficiency at 5Vin/3.3Vout |

**Choice: Texas Instruments LM2674MX-3.3**  
The TI LM2674MX-3.3 provides an efficient and stable 3.3V output with a wide range of voltage inputs. The datasheet claims it is accurate within ± 0.05V and has a possible energy efficiency of 90%. While the maximum current limit is relatively low, the overall goal of the sensor subsystem is to provide accurate measurements at low power and is unlikely to exceed 500 mA.  

# Illuminator LED

| Manufacturer | Part Number| Image | Pros | Cons |
|--------------|------------|------ | ---- |------|
| Luminus Devices Inc. | [SST-20-W35H-B120-J4351](https://www.digikey.com/en/products/detail/luminus-devices-inc/SST-20-W35H-B120-J4351/15903671) | ![ID](../images/ID_SST20_WLED.jpg) | - High CRI <br> - Low forward voltage <br> - balanced temperature <br> - High lumens | - High test current <br> - Expensive <br> - Low lm/W rating <br> - Expensive |
| XINGLIGHT | [XL-3020UWC-A](https://www.digikey.com/en/products/detail/xinglight/XL-3020UWC-A/25673208) | ![XL](../images/XL_3020UWC.jpg) | - Low test current <br> - Compact | - High forward voltage <br> - Low lumens <br> - No CRI listed <br> - Unbalanced temperature |
| Cree LED | [JB2835BWT-G-U40EA0000-N000P001](https://www.digikey.com/en/products/detail/cree-JB2835BWT-G-U40EA0000-N000P001/21531775) | ![CL](../images/CL_JB2835BWT.jpg) | - Very low forward voltage <br> - Low test current <br> - Affordable <br> -  Balanced temperature <br> - Very high lm/W rating | - Relatively low CRI <br> |

**Choice: Cree LED JB2835BWT-G-U40EA0000-N000P001**  
Illumination and color balance are critical for accurate measurements. The Cree J Series 2835 Pro9 white LED is capable of meeting these requirements with minimal power consumption. This model mas a typical forward voltage of 2.67V allowing it to be powered directly by the microcontroller. While the maximum current rating is 480 mA, the LED is capable of producing 28-31 lumens at only 55 mA. Finally, the rated temperature of 4000K and 90 CRI ensures a balanced light output for accurate measurements.  


# R/G LED

Debugging LEDs are required for ease of use and troubleshooting, both in the development phase and for the end user. These components are not as critical as the power regulator, sensor, or illumination LED, their power concumption must still be taken into consideration. The **Lite-On [LTST-C150KRKT](https://www.digikey.com/en/products/detail/liteon/LTST-C150KRKT/386761) and [LTST-C150KGKT](https://www.digikey.com/en/products/detail/liteon/LTST-C150KGKT/365085?s=N4IgTCBcDaIDIBUDKCC0BhAjAVgAwGkBzfBEAXQF8g)** have been selected due to their low power consumption and affordable price. Both models have a forward voltage of 2V and test current of 20 mA, allowing them to be powered directly by the microcontroller.  


# Microcontroller

The sensor subsystem is required to analyze and record information about the color of marbles before they are sent to the sorting actuator subsystem. This is done by measuring the amount of red, green, and blue light reflected off the surface of the marble to return 3 8-bit values to compare with an RGB sensor, and a white illuminator LED to ensure accurate measurements. Once these values have been found, they are sent to the actuator, HMI, and internet communication subsystem via UART. This subsystem is to run off of a 3.3V power supply under 500 mA.

Microchips [PIC18F27Q84](https://www.microchip.com/en-us/product/pic18f27q84) (QFN28) microcontroller is the best fit for the sensor subsystem. The chip is capable of operating at the designated 3.3 volts, and provides users with 1x I2C module as well as 5X UART modules. The sensor subsystem is relatively simple, and is only required to illuminate a ball on a track with a white LED and record R/G/B information in order to determine where the ball is sent. A single I2C module should be enough to control the TI OPT4060, and only one or two pins are required to control the LEDs. Multiple UART modules may be required to communicate with the HMI, sorting actuator, return actuator, and internet communication subsystems developed by the rest of team 201. The image below shows the selected I2C module, 3x UART modules, and 2x LED GPIO pins selected in the pin manager of the MCC Melody workspace.  

![PACKAGE](../images/QFN28.png)  
![PINS](../images/PIN%20SELECTION.png)
---