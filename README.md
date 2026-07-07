# Hi, I'm Karl
### Hardware & Embedded Systems Engineer | 25 Guns Applicant

As an aerospace engineer, I combine the uncompromising reliability and precision of aviation engineering with the rapid, solutions-oriented execution of a passionate maker.

Below is an abstract overview of my last Projects.
All shown Projects are Developed, Soldered, Programmed and Tested by myself. 
For Hardware Development i use Altium, STM32 are programmed with Visual Studio Code, FPGA with Quartus Prime. 
I solder components down to 0402 packages with a soldering Iron and a Heat Gun. 

---

## 🛠️ Core Expertise
* **Hardware:** PCB Design (Altium Designer), High-Speed Signalling, Multilayer Stackups, EMI/EMC Optimization.
* **Software:** Embedded C/C++, Real-Time Operating Systems (RTOS), Low-Level Drivers, State Machine Architecture.
* **GUI:** QT6, Data Visualisations, Data Logging

---

## 🚀 Featured Projects (Private Source / Commercializing)

# 1. High-Density, High Accuracy Power Supply


* **Overview:** 600 W Total Board Power, 4 fully Isolated Channels, 55 V, 3.5 A, 5 mV / 0.5 mA accuracy.
* **Implementation** The power supply implements an analog linear regulation with an operational amplifier and MOSFET, a low-side current sink, and a digital closed-loop control with an Intel MAX 10 FPGA.
* **Firmware Architecture:** Central STM32H7 for Data Collection and Data Output via USB. 4 Isolated MAX 10 FPGAs to Drive the 500kSpS ADCs and DACs.  

* **The Challenge:** low noise   
* **The Solution:** low noise LDO, längsregler, seperated Analog and digital GND 

HP HEAT --> Bottum Plate is a Big heatsink

Parallel ADC and DAC --> USE a FPGA instead of MCU

### Power Supply in Altium (Top and Bottom view) 

<img width="1267" height="787" alt="HP_Top" src="https://github.com/user-attachments/assets/7ecae9f4-9807-43c8-bf2d-e1a1280b4272" />
<img width="1226" height="549" alt="HP_Bottum" src="https://github.com/user-attachments/assets/e80a2064-5af4-410a-8f3d-e8c8b57a4b2a" />

---

### 💻 Code Snippet (Demonstrating Control Loop in the Intel MAX 10 FPGA)

```verilog
module dac_adjuster (
    input clk,
    input [15:0] dac_in,
    input signed [15:0] diff, 
    output reg [15:0] dac_out
);

    reg signed [15:0] step;
    reg signed [16:0] temp_out; // 17 Bit, um Überlauf bei der Rechnung zu prüfen
	 
    always @(posedge clk) begin
        // Schrittweite bestimmen
        if (diff > 50 || diff < -50) begin
            step = diff >>> 1; // Halbe Differenz
            
            if (step > 100)  step = 100;
            if (step < -100) step = -100;
        end
        else if (diff > 3) begin
            step = 1;
        end
        else if (diff < -3) begin
            step = -1;
        end
        else begin
            step = 0;
        end

        // Berechnung mit 17-Bit (Vorzeichenbehaftet)
        // Wir wandeln dac_in in ein signed 17-bit um, um sicher zu rechnen
        temp_out = $signed({1'b0, dac_in}) - $signed(step);

        // Limitierung / Sättigung
        if (temp_out < 17'sd32768) begin
            dac_out <= 16'd32768;      // Untere Grenze (Mitte)
        end
        else if (temp_out > 17'sd65535) begin
            dac_out <= 16'hFFFF;       // Obere Grenze (Max)
        end
        else begin
            dac_out <= temp_out[15:0]; // Gültiger Bereich
        end
    end
endmodule

```

---

# 2. Strain Gauge Measuring System

I developed a Strain Gauge Measuring System combining an AD7768 and an AD8221, achieving an exceptional microvolt-level input accuracy across a 4.096V range with dynamic software-calibrated gain stages. The system to supports sampling rates up to 256 kSPS. Additionally, I integrated a sinusoidal excitation option paired with analog synchronous demodulation to ensure bulletproof EMI immunity and eliminate DC offsets.

* **Overview:** 16 Channel, 256kSpS, Sine Excitation, Analog Sine Demodulation, DSP, Ethernet and USB Data Output 

* **The Challenge:** The two ADCs Output 8 Channel Data with 4 Byte each for a total of 32 Bytes via the ADC Data Bus. The STM32 SPI Interface can only receive 4 Byte in Direct Memory Access (DMA) Mode.   
* **The Solution:** Instead of the STM32 SPI Interface, i use the Serial Audio Interface (SAI), which has a 32 Byte DMA Buffer. This Allows me to use the Full Potential of each ADC and have the MCU free for DSP Tasks and Data Output.  

* **Firmware Architecture:** Developed drivers for the ADCs, Ethernet and USB Interface, Implementation of the FIR Filter.

### Strain Gauge Measuring System in Altium
<img width="1287" height="615" alt="AI_Shield_Layer_1" src="https://github.com/user-attachments/assets/8977fb7f-2260-4939-aab3-e5e947de58c1" />
<img width="1257" height="601" alt="AI_Shield_Altium" src="https://github.com/user-attachments/assets/a63ca2ed-b8a2-41ef-aef9-6a7aa2f07407" />

---

# 3. QT6 GUI

* **GUI:** For data visualization and data logging, I developed a Qt6 C++ GUI. The GUI is fully modular and can be adapted to fit various kinds of data visualization. It reads the JSON configuration files on startup, which configure the data interface, logging options, and, most importantly, which data instruments are displayed.
* The GUI features multiple data pages (depending on the configuration), a RAW data page, and a settings page.
* The data is logged to an HDF5 file.
* Receiving and logging data is handled in a separate thread.   
* For each instrument, I can choose between progress bars, text fields, value fields, charts, buttons, sliders, and analog instruments.
* Every instrument is defined in a single JSON file. It requires a type and a position to be displayed in the GUI. Several options can then be added to customize the instrument.

 
### 💻 Code Snippet (Demonstrating of Costumization of a Chart Instrument)

```json
	"chart1": {
		"type": "chart",
		"position": "1;1",
		"stretch": "3;2", 
		"scale": 0.00048828125, 
		"offset": 0, 
		"2d_plot": false, 
		"is_spline": false, 
		"title": "Sensoren", 
		"label": "Sensor 1", 
		"xlabel": "Time (s)", 
		"ylabel": "V_meas(mV)", 
		"draw_time": 19, 
		"y_color": "#191919", 
		"x_color": "#191919", 
		"graph_color": "#FF00FF", 
		"background_color": "#FFFFFF",
		"line_thickness": 3, 
		"auto_rescale": true, 
		"zoom_to_min_max": false, 
		"zoom_offset": 1,
		"y_axis_int": true, 
		"y_axis_decimals": 1, 
		"x_axis_tick_count": 7,
		"limits": {
			"min_warn": 3300, 
			"min_error": 3300, 
			"max_warn": 3400, 
			"max_error": 3400, 
			"warn_color": "#FFF700", 
			"error_color": "#FF0000"
		}    
	}, 
```

### 💻 Code Snippet (Demonstrating of Multithreading and Data handling)

```c
// ————————————————————————————————
// onDataReceived does both parsing AND decoding
// ————————————————————————————————
void dataStructure::onDataReceived(const QByteArray &data)
{
    // prevent reading while writing
    QWriteLocker locker(&m_lock);

    // basic sanity checks
    if (data.size() < 3) {
        qWarning() << "Received too-short packet:" << data.toHex();
        return;
    }

    // first byte => length (minus 0x80)
    quint8 expectedLen = quint8(data.at(0)) - 0x80;
    if (data.size() != expectedLen) {
        qWarning() << "Length mismatch: expected"
                   << expectedLen << "got" << data.size();
        return;
    }

    // second byte => ID
    int id = quint8(data.at(1));
    // Find all instruments that map to this ID
    const QVector<QString> instruments = m_idToInstruments.value(id);
    if (instruments.isEmpty()) {
        qWarning() << "No instruments indexed for ID:" << id;
        return;
    }

    // rest is Data
    QByteArray payload = data.mid(2);
    m_lastDataReceived[id] = payload;
    qint64 time = m_timer.nsecsElapsed() / 1000;
    m_usecsElapsed[id] = time;
    // Decode EACH instrument that shares this ID
    for (const QString& inst : instruments) {
        const int start  = m_start_byte.value(inst, -1);
        const int length = m_length.value(inst, 0);

        // Big-endian decode
        int value = 0;
        for (int i = start; i < start + length; ++i) {
            value = (value << 8) | quint8(payload.at(i));
        }

        // ---- Store history ----
        auto &hist = m_valueHistory[inst];
        const MathConfig &cfg = m_mathConfig.value(inst);

        hist.append(value);
        if (hist.size() > cfg.window) {
            hist.removeFirst();
        }

        // ---- Apply math ----
        int finalValue = value;

        switch (cfg.mode) {
        // only show Last Value
        case MathMode::Last:
            finalValue = value;
            break; 
        // mean Value  
        case MathMode::Mean:
            finalValue = qRound(calcMean(hist));
            break;
        // Standard deviation
        case MathMode::StdDev:
            finalValue = qRound(calcStdDev(hist));
            break;
        // min Value
        case MathMode::Min:
            finalValue = qRound(calcMin(hist));
            break;
        // max Value
        case MathMode::Max:
            finalValue = qRound(calcMax(hist));
            break;
        // diff to Last Value
        case MathMode::Diff:
            finalValue = qRound(calcDiff(hist));
            break;
        
        default:
            finalValue = value;
            break;
        }
        
        m_lastValues[inst] = finalValue;
        if (m_logger) {
            m_logger->logValue(inst, value, time);
        }
    }
}
```
### Example Layout of my GUI
<img width="1542" height="802" alt="image" src="https://github.com/user-attachments/assets/18320425-3b03-46a9-8f11-28afd02d6e2e" />


* The GUI can Receive Data via Ethernet and USB.
* It works on Linux and Windows PCs. 



---

## System Application: Aerodynamic Analytics 

* **The Objective:** I had a theory to optimize vehicle efficiency by mapping dynamic, real-time surface pressure distributions, with a specific focus on analyzing aerodynamic interferences between two or more vehicles. To validate this approach independently, I utilized my custom strain gauge measuring system and GUI to build a functional, real-world proof-of-concept.
* **Sensors:** The Sensors are Strain Gauge based Presure sensors, which make them easy to use with my System. 
* **Real-World Validation:** Conducted initial vehicle road tests using a 3-sensor configuration. The system successfully captured aerodynamic interference and wake-flux disruptions between two vehicles during a highway overtaking maneuver.
* **Next Scale:** The architecture is fully prepared to scale into a dense 100+ sensor array to map full-body dynamic airflow topologies and pinpoint absolute aerodynamic efficiency peaks.

### Future Outlook & Academic Scope

* **PhD Initiative:** This entire setup was engineered as the foundation for a potential PhD thesis to thoroughly investigate and map multi-vehicle aerodynamic behaviors. 
* **Current Status:** While the hardware and initial field tests have successfully proven the core thesis, I am currently seeking the right academic supervision to fully scale and complete this research.

### GUI Layout use for the Road Test
<img width="1908" height="994" alt="image" src="https://github.com/user-attachments/assets/a64497e5-16cd-40f9-8b72-73137e589cec" />

---

## Additional Skills & Hands-on Experience

* **3D Printing:** Proficient in operating and maintaining 3D printers, handling various filaments, and optimizing models for functional prototyping.
* **Laser Cutting** Built a custom 100W CO2 laser cutter from scratch; experienced in high-power optics alignment, digital vector design, and material processing.
* **Power Tools:** Fully confident and experienced in the safe operation of a wide range of industrial and workshop power tools for fabrication.

---

## 📄 Engineering Portfolio
Due to intellectual property and future commercialization, my core repositories are private. 

Code or Hardware deep-dives can be provided via screen-share during technical interviews.
