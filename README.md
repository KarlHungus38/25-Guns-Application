# Hi, I'm Karl
### Hardware & Embedded Systems Engineer | 25 Guns Applicant

Due to intellectual property (IP) and future commercialization, my core repositories are private. Below is an abstract overview of my last 3 Projects.


---

## 🛠️ Core Expertise
* **Hardware:** PCB Design (Altium Designer), High-Speed Signalling, Multilayer Stackups, EMI/EMC Optimization.
* **Software:** Embedded C/C++, Real-Time Operating Systems (RTOS), Low-Level Drivers, State Machine Architecture.
* **GUI:** QT6, Data Visualisations, Data Logging

---

## 🚀 Featured Projects (Private Source / Commercializing)

### 1. High-Density Power Supply
*Visuals of your board or an abstract architecture diagram go here*
<!-- <img src="path-to-your-blurred-or-3d-altium-render.png" width="600"> -->
* **Overview:** 600 W Total Board Power, 4 fully Isolated Channels, 55 V, 3.5 A, 5 mV / 0.5 mA accuracy.
* **Implementation** The power supply implements an analog linear regulation with an operational amplifier and MOSFET, a low-side current sink, and a digital closed-loop control with an Intel MAX 10 FPGA.
* **Firmware Architecture:** Central STM32H7 for Data Collection and Data Output via USB. 4 Isolated MAX 10 FPGAs to Drive the 500kSpS ADCs and DACs.  

---

## 💻 Code Snippet (Demonstrating State Machine in the Intel MAX 10 FPGA)

```c
// Example: Efficient, non-blocking hardware abstraction layer (HAL) snippet
typedef enum {
    STATE_IDLE,
    STATE_PROCESSING,
    STATE_ERROR
} SystemState_t;

void Process_System_State(SystemState_t *current_state) {
    if (current_state == NULL) return;
    
    switch (*current_state) {
        case STATE_IDLE:
            // Check sensors and transition cleanly
            break;
        case STATE_PROCESSING:
            // High-priority execution
            break;
        default:
            // Safe fallback state
            *current_state = STATE_ERROR;
            Trigger_Hardware_Safe_Mode();
            break;
    }
}
```

---

### 2. Strain Gauge Measuring System
*Visuals of your board or an abstract architecture diagram go here*
<!-- <img src="path-to-your-blurred-or-3d-altium-render.png" width="600"> -->
* **Overview:** 16 Channel, 256kSpS, Sine Excitation, Analog Sine Demodulation, DSP, Ethernet and USB Data Output
* **Implementation** 
* **Firmware Architecture:**   

* **The Challenge:** The two ADCs Output 8 Channel Data with 4 Byte each for a total of 32 Bytes via the Data SPI. The STM SPI Interface can only receice a 4 Byte in Direct Memory Access (DMA) Mode.   
* **The Solution:** Instead of the STM32 SPI Interface, i use the Serial Audio Interface (SAI), which has a 32 Byte DMA Buffer. 
* **Firmware Architecture:** Developed drivers for the ADCs, Ethernet and USB Interface, Implementation of the FIR Filter.


## 📄 Engineering Portfolio
For full details on layer stackups, Altium 3D renders, and hardware validation tests, please refer to my **Additional Engineering Document (PDF)** attached directly to my Tesla application. 

*Note: Code deep-dives can be provided via encrypted screen-share during technical interviews.*
