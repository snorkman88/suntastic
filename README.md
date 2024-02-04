# suntastic
An ultra low-power management board for IoT devices

## Overview

This repository contains the design and documentation of a power management board for ultra low power embedded devices. The board features a solar charger circuit with Maximum Power Point Tracking (MPPT) for 5V solar panels, two DC-DC converters, and a Schmitt trigger control for a load switch with adjustable hysteresis.  


## Why would you need this board?
I recently started working with [Meshtastic](https://meshtastic.org/) and decided to deploy solar powered repeaters based on the WisBlock Core module that contains a RAK4631. So far it's been recommended to use this core module with any of the available base boards, after inspecting [RAK's 19007](https://store.rakwireless.com/products/rak19007-wisblock-base-board-2nd-gen) base board, I noticed that the battery charging stage based on the [TP4054](https://www.laskakit.cz/user/related_files/tp4054.pdf) is not performing any kind of optimization to maximize the power available from the solar input.  
In contrast to linear chargers, MPPT chargers excel in extracting energy from solar panels under conditions such as partial shading, changing light angles, and fluctuating weather conditions. This adaptability is crucial for maximizing energy harvesting in real-world scenarios.  
By efficiently managing the charging process, MPPT helps prevent overcharging or undercharging of the accumulators in small IoT devices. This contributes to extending the overall lifespan of the battery, reducing the frequency of maintenance or battery replacement, and enhancing the device's reliability.  

Another point that motivated the start of this project is the use of supercapacitors instead of conventional LiPo or LiFePo batteries. Whether supercapacitors are better than batteries for IoT devices depends on the specific requirements of the application. Here are some factors to consider:

1. Power Density:
Supercapacitors generally have higher power density than batteries. They can quickly charge and discharge, making them suitable for applications that require rapid bursts of energy. This can be advantageous in IoT devices that need to transmit data or perform periodic tasks quickly.

2. Cycle Life:
Supercapacitors often have a longer cycle life compared to batteries. They can withstand a large number of charge-discharge cycles without significant degradation even when being completely depleted and its output voltage is 0V. This is beneficial for IoT devices that may need to operate for extended periods without maintenance.

3. Lifetime and Maintenance:
Supercapacitors can have a longer overall lifetime than batteries, as they do not suffer from the same degradation issues over time. They also require less maintenance since they do not have the same chemical processes that can lead to wear and tear in batteries.

4. Size and Weight:
Supercapacitors can be more compact and lighter than batteries, which is important for IoT devices with size and weight constraints. This is particularly relevant for small, portable devices or those with limited space.

5. Temperature Sensitivity:
Supercapacitors are known for their resilience to temperature variations, making them suitable for applications in harsh environments. This can be an advantage in IoT devices deployed in diverse environments where temperature fluctuations are common. Supercapacitors have a temperature range of operation og -40°C to 85°C or even higher in some cases.

6. Energy Density:
Batteries typically have higher energy density, meaning they can store more energy per unit of volume or weight. If the IoT device requires long periods of operation without recharging, a battery might be a more suitable choice.

In summary, the choice between supercapacitors and batteries for powering IoT devices depends on the specific requirements of the application. Supercapacitors may be a better choice for applications that require high power density, quick charge/discharge cycles, and long cycle life, while batteries may be preferable when higher energy density and longer operating periods without recharging are crucial. Each technology has its own strengths and limitations, and the decision should be based on the specific needs and constraints of the IoT deployment.


## IMPORTANT
IMPORTANT: this board has been specifically designed to handle 5V solar panels and supercapacitors up to a voltage of 5.4V max.

## Table of Contents

- [Solar Charger Circuit](#solar-charger-circuit)
- [DC-DC Converters](#dc-dc-converters)
  - [Boost Converter](#boost-converter)
  - [Buck-Boost Converter](#buck-boost-converter)
- [UVLO - Schmitt Trigger Control](#schmitt-trigger-control)
- [Getting Started](#getting-started)
- [License](#license)



## Solar Charger Circuit

The solar charger circuit is designed to efficiently charge a battery using 5V solar panels. It incorporates Maximum Power Point Tracking (MPPT) to optimize the energy harvesting from the solar panels.

## DC-DC Converters

### Boost Converter

The first DC-DC converter that is presented is a boost converter responsible for energizing the control stage. It is based on a [TPS610981](https://www.ti.com/lit/ds/symlink/tps610985.pdf?ts=1706487312837&ref_url=https%253A%252F%252Fwww.google.com%252F) from Texas Instruments and takes variable voltage seen at the terminals of the accumulator (i.e a supercapacitor) and converts it into 3.3V one.  
This boost converter features a very low start-up voltage (0.7) and Iq=2uA when operating.

### Buck-Boost Converter

The second DC-DC converter is a buck-boost converter designed to supply power to the IoT device running on 3.3V. It ensures a stable and regulated voltage output, accommodating variations in solar panel output.

## UVLO - Schmitt Trigger Control

The project includes a Schmitt trigger control for a load switch with adjustable hysteresis. This feature enables precise control of the load switch, allowing for flexibility in turning on and off the connected devices based on the application's requirements.

## Getting Started

To replicate or contribute to this project,
