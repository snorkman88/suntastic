# DRAFT: Suntastic
An ultra low-power management board for meshtastic solar repeater devices.

## Overview

This repository contains the design proposal of a power management board for ultra low power embedded devices. The board features a solar charger circuit with Maximum Power Point Tracking (MPPT) for 5V solar panels, two DC-DC converters, a load switch control with adjustable hysteresis and an external Watchdog Timer.

![Hardware block diagram](https://github.com/snorkman88/suntastic/blob/main/block_diagram/HW_block_diagram.png)


## Why did I start this project?

### MPPT is a must
I recently started working with [Meshtastic](https://meshtastic.org/) and decided to deploy solar powered repeaters based on the WisBlock Core module that contains a RAK4631. So far it's been recommended to use this core module with any of the available base boards, however after inspecting [RAK's 19007](https://store.rakwireless.com/products/rak19007-wisblock-base-board-2nd-gen) base board, I noticed that the battery charging stage based on the [TP4054](https://www.laskakit.cz/user/related_files/tp4054.pdf) is not performing any kind of optimization to maximize the power available from the solar input.  

In contrast to linear chargers, MPPT chargers excel in extracting energy from solar panels under conditions such as partial shading, changing light angles, and fluctuating weather conditions. This adaptability is crucial for maximizing energy harvesting in real-world scenarios. By efficiently managing the charging process, MPPT helps prevent overcharging or undercharging of the accumulators in small IoT devices. This contributes to extending the overall lifespan of the battery, reducing the frequency of maintenance or battery replacement, and enhancing the device's reliability.  

### Reduce the current consumption of all the additional electronics
The recommended Wisblock base board RAK19007 contains extra electronics that are not really necessary once the RAK4631 core module has been correctly configured and flashed. This extra electronics however, do imply extra power consumption.  

Once the core is flashed it only needs a proper 3.3V power source to start working.

### Supercapacitors as main energy storage
Another point that motivated the start of this project is the use of supercapacitors instead of conventional LiPo or LiFePo batteries. Whether supercapacitors are better than batteries for IoT devices depends on the specific requirements of the application. Here are some factors to consider:

1. Power Density:
Supercapacitors generally have higher power density than batteries. They can quickly charge and discharge, making them suitable for applications that require rapid bursts of energy. This can be advantageous in IoT devices that need to transmit data or perform periodic tasks quickly.

2. Cycle Life:
Supercapacitors often have a longer cycle life compared to batteries. Compared to batteries, they can withstand a large number of charge-discharge cycles without significant degradation in their structure even when being completely depleted and its output voltage drops down to 0V. This is beneficial for IoT devices that may need to operate for extended periods without maintenance.

3. Lifetime and Maintenance:
Supercapacitors can have a longer overall lifetime than batteries, as they do not suffer from the same degradation issues over time. They also require less maintenance since they do not have the same chemical processes that can lead to wear and tear in batteries.

4. Size and Weight:
Supercapacitors can be more compact and lighter than batteries, which is important for IoT devices with size and weight constraints. This is particularly relevant for small, portable devices or those with limited space.

5. Temperature Sensitivity:
Supercapacitors are known for their resilience to temperature variations, making them suitable for applications in harsh environments. This can be an advantage in IoT devices deployed in diverse environments where temperature fluctuations are common. Supercapacitors have a temperature range of operation og -40°C to 85°C or even higher in some cases.

6. Energy Density:
Batteries typically have higher energy density, meaning they can store more energy per unit of volume or weight. If the IoT device requires long periods of operation without recharging, a battery might be a more suitable choice.

In summary, the choice between supercapacitors and batteries for powering IoT devices depends on the specific requirements of the application. Supercapacitors may be a better choice for applications that require high power density, quick charge/discharge cycles, and long cycle life, while batteries may be preferable when higher energy density and longer operating periods without recharging are crucial. Each technology has its own strengths and limitations, and the decision should be based on the specific needs and constraints of the IoT deployment.  

### External Watchdog timers are sometimes needed
For IoT devices deployed in remote areas, the incorporation of an external watchdog timer becomes even more critical due to the challenges associated with limited accessibility and maintenance. Remote locations often entail harsh environmental conditions and restricted physical access, making it challenging to address issues promptly. The watchdog timer becomes a crucial asset in such scenarios as it autonomously monitors the device's operational state and intervenes in the case of failures. This capability ensures that even if a device experiences a software glitch or hardware malfunction, it can autonomously reset itself, mitigating potential downtime without requiring manual intervention.  

The reliability and self-correcting nature of an external watchdog timer contribute significantly to the overall resilience of IoT devices in remote deployments, enhancing their ability to function autonomously and maintain consistent operation in challenging and hard-to-reach environments.


## IMPORTANT
IMPORTANT: this board has been specifically thought to handle 5V solar panels and supercapacitors up to a voltage of 5.4V max.

## Table of Contents

- [Solar Charger Circuit](#solar-charger-circuit)
- [DC-DC Converters](#dc-dc-converters)
  - [Boost Converter](#boost-converter)
  - [Buck-Boost Converter](#buck-boost-converter)
- [UVLO - Schmitt Trigger Control](#schmitt-trigger-control)
- [External Watchdog timer](#watchdog-timer)
- [Monitoring of the voltage seen on the accumulator]()
- [Getting Started](#getting-started)
- [License](#license)



## Solar Charger Circuit

The solar charger circuit is designed to efficiently charge an accumulator using 5V solar panels. It incorporates Maximum Power Point Tracking (MPPT) to optimize the energy harvesting from the solar panels. **TO BE DEVELOPED**

## DC-DC Converters

### Boost Converter

The first DC-DC converter that is presented is a boost converter responsible for energizing the control stage **ONLY**. It is based on a [TPS610981](https://www.ti.com/lit/ds/symlink/tps610985.pdf?ts=1706487312837&ref_url=https%253A%252F%252Fwww.google.com%252F) from Texas Instruments and takes variable voltage seen at the terminals of the accumulator (i.e a supercapacitor) and converts it to a 4.3V one.  
This boost converter features a very low start-up voltage (0.7) and Iq=2uA when in operation.

### Buck-Boost Converter

The second DC-DC converter is a buck-boost converter in charge of supplying power to the 3.3 V to which the IoT device (RAK4631) and watchdog timer are connected to.  
This converter should start operating after the load switch controlled by the UVLO is closed.  
**TO BE CONTINUED**

### Why are there two DC-DC converters?
It is desirable that the UVLO control circuit starts operating as early as possible if there is enough input power (around Vin=0.7V) and BEFORE any other electronic device. The opposite behaviour is also desired. In other words, when the input voltage coming from the accumulator decreases, the UVLO should be last part that is turned off.

Since it's expected that the control stage consumes only a few microamps when operating, the output voltage of 4.3V should remain steady while the accumulator (i.e: supercapacitor) continues to charge. Once the voltage seen accross the terminals of the accumulator reaches the lock-in voltage, the second converter and microcontroller will be energised.


## UVLO - Schmitt Trigger Control

The project includes an UVLO circuit with with adjustable hysteresis that enables or disables a load switch. The load connected to the switch will be the main [Buck-Boost Converter](#buck-boost-converter) that feeds power the RAK431 or another microcontroller.  

Since the supercapacitor bank I have at the moment (250F x 5.4V) already comes with an extra PCB to protect them against overvoltages higher than 5V, and for [this simulation only](https://www.falstad.com/circuit/circuitjs.html?ctz=CQAgjCAMB0l3BWEBOaBmAHAFgOyQ2AGwJhZaREiERIKQgICmAtGGAFACGIaATPfwwNeQwSCFZ0UcPHghJCJDGQqcdWXEKFkaGbPYB3Hv3DJeIXryynzkLjwTnKJJ4XHyp9MBoHQVZ9Vl+SBxdBSRvfSM+LzNjej4oQ3jjB3NEuzAEXRxCITBclBFwQoEGBnYsnOwLDCFkQgE66VsKoxwa3hxzM3zS5Nz8uIamoTsjXpK3FSd+9pq0SGsXcDdxkEHV+kdRfCSCp2LCazBi1jWNj0Vy+lvoJAA1AHsAGwAXTgBzRmSxNnNLCdTkkjICbBYrOBgXYAEpbEDnLzAxHSejkDy6W5Qe7sABOUwRRC8hWYDWkjUCdnxBTczAwxIu9B0GjxBMoNKo1jRxCiBJRRMJjRBfLJHLpt2SHIZqySnwJxwJ9NRkpJSrQjgRSvW6vM4rSyqMUos3QNxtaXQBkPWFoh1hituFf2Bp3y0MlRxOxX+SSe4BlXksyBo2PgEASKGkyB47CAA), a value of 3V was chosen for the upper limit and 1V for the lower one.  

Since it's the intention to make the supercapacitor charge up to a voltage value close to the maximum, the final version will have a lock-in voltage closer to 5V or 5.4V if I get a bank without the overvoltage protection.  
The lower limit of the hysterisis window will be kept at 1V and.
In order to avoid flickering of the output when the supercap voltage reaches the lock-in and -out values, an SR latch based on NOR gates will either SET (and keep it HIGH even if the input flickers) when the lock-in voltage is reached; and RESET its output when reaching the lock-out value.  

The circuit consists of two [TLV3691](https://www.ti.com/lit/ds/symlink/tlv3691.pdf?ts=1706791844150&ref_url=https%253A%252F%252Fwww.google.com%252F) nano-power comparators powered by the 4.3V bus.  

For the voltage references, it's yet to be defined if specific high-precision voltage references like [REF35](https://www.ti.com/lit/ds/symlink/ref35.pdf?ts=1706804680495&ref_url=https%253A%252F%252Fwww.ti.com%252Fpower-management%252Fvoltage-reference%252Fseries-voltage-reference%252Fproducts.html) will be used or simply get those reference values from a voltage divider. **Current consumption reduction is PRIORITY.**

For the latching of the output, low power [SN74AUP2G02](https://www.ti.com/lit/ds/symlink/sn74aup2g02.pdf?ts=1706885625366&ref_url=https%253A%252F%252Fwww.google.com%252F) NOR gates will be used. These gates will also be powered by the 4.3V bus.

The load switch will be a [TPS22917](https://www.ti.com/lit/ds/symlink/tps22917.pdf?ts=1706648579344&ref_url=https%253A%252F%252Fwww.google.com%252F).

In summary this circuit makes sure the embedded device stays de-energised until the supercapacitor is fully charged

## Watchdog timer
The external watchdog timer that will trigger a reset in case of a software bug/glitch or brownout will be a [TPL5010](https://www.ti.com/lit/ds/symlink/tpl5010.pdf?ts=1706954677109&ref_url=https%253A%252F%252Fwww.google.com%252F) from Texas Instrument.  

For this feature to be fully functional, **CHANGES IN THE CORE MODULE FIRMWARE ARE NEEDED** to periodically generate a pulse to reset the counter of the  WDT.  

## Monitoring the current voltage on the supercapacitor
A voltage divider must be connected to Pin40 (P0.05/AIN3) of the RAK4631.  
**CHANGES IN THE CORE MODULE FIRMWARE ARE NEEDED TOO** in order to properly reflect the current voltage value on the supercapacitor based on the resistors ratio from the voltage divider.

## Getting Started

To replicate or contribute to this project,
