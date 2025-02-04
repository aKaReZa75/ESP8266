# Special Pins of ESP8266
This document provides a detailed overview of the special pins of the ESP8266 microcontroller, including their roles, required states for proper operation, and usage guidelines.  
It also covers dual-purpose pins, input and output pins, analog input, boot modes, and important notes to ensure stable and reliable performance.

![ESP8266 Pinout1](Images/ESP8266_PinOUT.jpg)

![ESP8266 Pinout1](Images/ESP8266_PinOut2.jpg)

To start the ESP8266, certain pins must be in a specific state (HIGH or LOW). These pins are:

**RST, Enable, GPIO0, GPIO2, GPIO15, GPIO16**

- The **RST** pin must be pulled up; otherwise, due to its high sensitivity, any noise may cause the ESP8266 to reset.
- The **Enable** pin must also be pulled up; otherwise, the ESP8266 will not power up.
- The **GPIO0** pin must be pulled up for normal operation; otherwise, if it is LOW during startup, the ESP8266 will enter BootLoader mode (for flashing firmware).
- The **GPIO2** pin must be pulled up; if LOW during startup (used as input), the ESP8266 may fail to boot. The internal LED is also connected to this pin.
- The **GPIO15** pin must be pulled down; otherwise, the ESP8266 will not boot.
- The **GPIO16** pin is used to wake the ESP8266 from Deep Sleep mode and must be connected to RST for this function.

**Note**: In some ESP8266 modules, the internal LED is not connected to GPIO2 but to GPIO1 (TXD), such as in the ESP-12E module. Always verify the specific version of the ESP8266 you are using, as the LED may be connected to either GPIO2 or GPIO1 depending on the board.

## Dual-purpose Pins (Input/Output)
For input and output purposes, you can use the following pins without any issues:
- GPIO4
- GPIO5
- GPIO12
- GPIO13
- GPIO14

These pins remain unchanged during the startup of the ESP8266 and can be used without any problems for connecting outputs like relays.
Note: GPIO12, GPIO13, GPIO14, and GPIO15 are related to SPI communication.

## Output Pins
The following pins can be used as output:
- GPIO0
- GPIO1
- GPIO2
- GPIO15

However, these pins change their state during startup, which may cause unwanted behavior in connected components like relays, so use these pins as output with caution.
To use the serial communication pin GPIO1 (TXD) as output, transmission communication should not be used throughout the program and should not be configured.
The GPIO0 and GPIO2 pins must be pulled up during the ESP8266's initial startup; otherwise, it won't boot.
GPIO15 pin must be pulled down initially; otherwise, the ESP8266 will not power up.

## Input Pins
The following pins can be used as input:
- GPIO0
- GPIO2
- GPIO3
- A0

To use the serial communication pin GPIO3 (RXD) as input, transmission communication should not be used throughout the program and should not be configured, otherwise this pin can be used as an input only if UART communication is not required.
The GPIO0 and GPIO2 pins must be pulled up during the ESP8266's initial startup; otherwise, it won't boot, and if they are low during startup, it will not power up.
The analog input pin can also be used as a digital input, and unlike other ESP8266 pins, this level is equal to 1 volt. 
Additionally, you can only apply a voltage range of 0 to 1 volt to the analog pin on the ESP8266, and a voltage divider should be used to prevent damage to the ESP8266.

## Analog Input Pins
The ESP8266 features a single analog input pin, labeled **A0**. 
This pin is used for reading analog voltage levels and converting them into digital values using an ADC (Analog-to-Digital Converter). 

**Note:** On some development boards like NodeMCU, a built-in voltage divider allows up to 3.3V.

### Specifications
- **Resolution**: 10-bit ADC (0 to 1023)
- **Voltage Range**: 0 to 1V
- **Accuracy**: Typically around ±2 LSB (Least Significant Bits)

**Note:** The **A0** pin only supports voltage levels between 0 to 1V. Voltages higher than 1V may damage the ESP8266. Use a voltage divider circuit to scale down higher voltage levels to the 0-1V range.

### Voltage Conversion 
To convert ADC values to voltage, use the following macro:
```c
#define ADC_TO_VOLTAGE(adc_value) ((adc_value) * (1.0 / 1023.0))
```
However, for better computational efficiency on microcontrollers, it's recommended to use the following form:
```c
#define ADC_TO_VOLTAGE(adc_value) (adc_value * 0.0098039)  /**< 0.0098039 = 1/1023 */
```
**Reason:** By using the constant 0.0098039 directly, the microcontroller avoids performing a division operation, which is computationally more expensive. 
Instead, it only needs to perform multiplication, which is faster and more efficient. This can lead to improved performance, especially in applications where ADC readings are taken frequently.

### Voltage Dvider
For a typical voltage divider circuit to convert 3.3V to 1V:
```plaintext
Vin (Input Voltage)
  |
 R1
  |
  +----> A0 (Analog Pin of ESP8266)
  |
 R2
  |
 GND (Ground)

```
**Formula**:  
$$V_{\text{out}} = V_{\text{in}} \times \frac{R2}{R1 + R2}$$

**Substitute Values**:  
$$1.0V = 5V \times \frac{R2}{R1 + R2}$$

**Assuming**:  
- Assume \( R2 = 10kΩ \)

**Solve for R1**:  
$$\frac{R1}{R2} = \frac{V_{\text{in}}}{V_{\text{out}}} - 1 = \frac{5}{1} - 1 = 4$$  
$$R1 = 4 \times R2 = 4 \times 10kΩ = 40kΩ$$

**Standard Resistors**:  
- R1 = 39kΩ (nearest standard value)
- R2 = 10kΩ  

**Verification**:  
$$V_{\text{out}} = 5V \times \frac{10kΩ}{39kΩ + 10kΩ} = 1.02V$$

### **Summary Table**  
| Input Voltage | R1    | R2    | Calculated Vout | Voltage Formula |
|---------------|-------|-------|-----------------| -----------------| 
| 3.3V          | 22kΩ  | 10kΩ  | 1.03V           | adc_value * 0.0032258‬ | 
| 5V            | 39kΩ  | 10kΩ  | 1.02V           | adc_value * 0.0048876 |  
| 12V           | 110kΩ  | 10kΩ | 1.0V            | adc_value * 0.0117302 |  
                                                                
## Neither Input Nor Output Pins
The following pins are related to the Flash IC and should not be configured as IO; otherwise, the ESP8266 operation may be disrupted, causing instability:
- GPIO6
- GPIO7
- GPIO8
- GPIO9
- GPIO10
- GPIO11

## Boot Modes
The ESP8266 has different boot modes that are selected based on the voltage levels applied to certain GPIO pins during power-up.

| GPIO15 | GPIO0 | GPIO2 | Mode                            |
|--------|-------|-------|---------------------------------|
| LOW    | LOW   | HIGH  | Uart Bootloader                 |
| LOW    | HIGH  | HIGH  | Boot sketch (SPI flash)         |
| HIGH   | x     | x     | SDIO mode                       |

1. **Uart Bootloader Mode:**
   - **Purpose:** This mode is used for flashing the firmware onto the ESP8266 via the UART interface.
   - **GPIO Configuration:** 
     - GPIO15 = 0V
     - GPIO0 = 0V
     - GPIO2 = 3.3V

2. **Boot Sketch (SPI Flash) Mode:**
   - **Purpose:** This is the standard mode for running the user program stored in the SPI flash memory.
   - **GPIO Configuration:** 
     - GPIO15 = 0V
     - GPIO0 = 3.3V (Pull-up)
     - GPIO2 = 3.3V

3. **SDIO Mode:**
   - **Purpose:** This mode is not commonly used for Arduino applications.
   - **GPIO Configuration:** 
     - GPIO15 = 3.3V 
     - GPIO0 = x (don't care)
     - GPIO2 = x (don't care)
- Caution: GPIO15 must be **HIGH** for this mode, but most boards hardwire GPIO15 to GND.     

These boot conditions must be maintained by using appropriate external resistors or relying on those provided by the board manufacturer. 
Failing to meet these conditions can result in improper booting or entering unintended boot modes.
By understanding and correctly configuring these pins, you can ensure that your ESP8266 boots into the desired mode and operates reliably.

## Notes
- **GPIO15:** Floating GPIO15 (no pull-down) causes boot failure. Always pulled low (0V) for standard operation modes. If using an ESP8266 module like ESP-12, check if an internal 5KΩ pull-down resistor is present. Otherwise, add an external 10KΩ resistor to GND.
- **GPIO0:** Pulled high (3.3V) for normal operation. Using it as a Hi-Z input is not possible. A direct switch to GND may cause boot issues.
- **GPIO2:** Should not be low at boot. You can’t connect a switch directly to this pin without causing boot issues.
- You don’t have to add an external pull-up resistor to GPIO2, the internal one is enabled at boot.
- The serial communication pins (**GPIO1**, **GPIO3**) are initially HIGH when the ESP8266 is powered up.
- The reliable pins for output are **GPIO4** and **GPIO5**.
- All ESP8266 pins, except **GPIO16**, support interrupts.
- **GPIO16** Supports **interrupts**, but only for waking from **Deep Sleep** (not general-purpose interrupts).  
- All ESP8266 pins support 10-bit software PWM.
- The SPI communication pins are (**GPIO12 (MISO)**, **GPIO13 (MOSI)**, **GPIO14 (SCK)**, **GPIO14 (SCK)**, **GPIO15 (CS)**) but **GPIO15** must be LOW during boot but can be used as SPI CS after boot.

# ESP8266 Pinout & Description

| **Pin**  | **Default Function**  | **Alternate Functions** | **Description** | **Important Notes** |
|----------|----------------------|------------------------|-----------------|-----------------|
| **VCC**  | Power Supply | - | 3.3V power input | Maximum current: 500mA |
| **GND**  | Ground | - | Connect to circuit ground | Essential for stable operation |
| **EN (CH_PD)** | Chip Enable | - | Activates the ESP8266 | Must be **High (3.3V)** to power up |
| **RST**  | Reset | - | A **Low** pulse resets the module | Must be **Pull-up (3.3V)** for stable operation |
| **GPIO0** | General Purpose I/O | Boot Mode Selection | Controls boot mode | Must be **Pull-up (3.3V)** for normal boot |
| **GPIO1** | TXD (UART0) | General Purpose Output | UART TX pin | Defaults to **High** at boot |
| **GPIO2** | General Purpose I/O | Boot Mode Selection, I2C SDA | Used for boot mode selection | Must be **Pull-up (3.3V)** for normal boot |
| **GPIO3** | RXD (UART0) | General Purpose Input, I2C SCL | UART RX pin | Defaults to **High** at boot |
| **GPIO4** | General Purpose I/O | I2C (SDA) | Usable I/O pin | No boot restrictions |
| **GPIO5** | General Purpose I/O | I2C (SCL) | Usable I/O pin | No boot restrictions |
| **GPIO6** | SPI_CLK (Flash) | - | Connected to Flash memory | **Do not use for I/O** |
| **GPIO7** | SPI_MISO (Flash) | - | Connected to Flash memory | **Do not use for I/O** |
| **GPIO8** | SPI_MOSI (Flash) | - | Connected to Flash memory | **Do not use for I/O** |
| **GPIO9** | SPI_HD (Flash) | - | Connected to Flash memory | **Do not use for I/O** |
| **GPIO10** | SPI_WP (Flash) | - | Connected to Flash memory | **Do not use for I/O** |
| **GPIO11** | SPI_CS (Flash) | - | Connected to Flash memory | **Do not use for I/O** |
| **GPIO12** | General Purpose I/O | SPI_MISO | Usable I/O pin | No boot restrictions |
| **GPIO13** | General Purpose I/O | SPI_MOSI | Usable I/O pin | No boot restrictions |
| **GPIO14** | General Purpose I/O | SPI_CLK | Usable I/O pin | No boot restrictions |
| **GPIO15** | Boot Mode Selection | SPI_CS  | Required for boot mode selection | Must be **Pull-down (GND)** for normal boot |
| **GPIO16** | Wake from Deep Sleep | - | Used for waking from deep sleep | **Must be connected to RST** for wake-up from Deep Sleep |
| **A0** | ADC (Analog Input) | - | Analog input pin | Voltage range **0V - 1V** only |

# 🤝 Contributing to the Repository
To contribute to this repository, please follow these steps:
1. **Fork the Repository**  
2. **Clone the Forked Repository**  
3. **Create a New Branch**  
4. **Make Your Changes**  
5. **Commit Your Changes**  
6. **Push Your Changes to Your Forked Repository**  
7. **Submit a Pull Request (PR)**  

> [!NOTE]
> Please ensure your pull request includes a clear description of the changes you’ve made.
> Once submitted, I will review your contribution and provide feedback if necessary.

# 🌟 Support Me
If you found this repository useful:
- Subscribe to my [YouTube Channel](https://www.youtube.com/@aKaReZa75).
- Share this repository with others.
- Give this repository and my other repositories a star.
- Follow my [GitHub account](https://github.com/aKaReZa75).

# 📜 License
This project is licensed under the GPL-3.0 License. This license grants you the freedom to use, modify, and distribute the project as long as you:
- Credit the original authors: Give proper attribution to the original creators.
- Disclose source code: If you distribute a modified version, you must make the source code available under the same GPL license.
- Maintain the same license: When you distribute derivative works, they must be licensed under the GPL-3.0 too.
- Feel free to use it in your projects, but make sure to comply with the terms of this license.
  
# ✉️ Contact Me
Feel free to reach out to me through any of the following platforms:
- 📧 [Email: aKaReZa75@gmail.com](mailto:aKaReZa75@gmail.com)
- 🎥 [YouTube: @aKaReZa75](https://www.youtube.com/@aKaReZa75)
- 💼 [LinkedIn: @akareza75](https://www.linkedin.com/in/akareza75)
