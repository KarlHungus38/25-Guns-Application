# Hi, I'm Karl Heinemann 
### Hardware & Embedded Systems Engineer | 25 Guns Applicant

As an aerospace engineer, I bridge the gap between the uncompromising reliability of aviation standards and the rapid, solutions-oriented execution of a passionate maker. 

Below is an abstract overview of my recent projects. Every system shown below was **designed, routed, soldered, programmed, and validated entirely by myself.**

* **Hardware Design:** Altium Designer (Multilayer, High-Speed, EMI/EMC optimization)
* **Firmware:** Embedded C/C++ (STM32/ARM), Verilog (Intel/Altera FPGA), RTOS, bare-metal low-level drivers
* **Software & GUI:** Qt6 (C++), HDF5 Data Logging, Multi-threading, Cross-Platform (Linux/Windows)
* **Prototyping:** Manual assembly and soldering down to 0402 packages using hot air and iron.
---

## 🛠️ Core Expertise
* **Hardware:** PCB Design, High-Speed Signalling, Multilayer Stackups, Impedance Matching, EMI/EMC Mitigation.
* **Software:** Embedded C/C++, Real-Time Operating Systems (RTOS), DMA Optimization, State Machine Architectures.
* **GUI & Analytics:** Qt6 Framework, Real-time Data Visualization, High-Throughput Data Logging.

---

## Featured Projects (Private Source / Commercializing)

## 1. High-Density, High-Accuracy Power Supply & Electronic Load

* **Overview:** 600W Total Board Power, 4 fully isolated channels (55V, 3.5A) with 5mV / 0.5mA setting accuracy.
* **Implementation:** Features analog linear regulation via operational amplifiers and power MOSFETs, a low-side current sink, and a digital closed-loop control system powered by an Intel MAX 10 FPGA.
* **Firmware Architecture:** A central STM32H7 manages high-level data aggregation and USB communication. Four isolated MAX 10 FPGAs independently drive the 500 kSPS ADCs and DACs to ensure strict determinism.

* **Design Challenges & Engineering Solutions:**
  1. **Strict Noise Constraints:** *Challenge:* Achieving low noise output for precision analog rails. *Solution:* Integrated high-PSRR linear regulators combined with strictly segregated, star-routed analog and digital ground (GND) planes to eliminate crosstalk.
  2. **Extreme Thermal Management:** *Challenge:* Managing high heat dissipation when operating as an active electronic load. *Solution:* Mount the PCB onto an actively cooled 20x20 cm heatsink.
  3. **High-Speed Data Throughput:** *Challenge:* Interfacing parallel ADC/DAC architectures. *Solution:* Offloaded the control loop from the MCU to dedicated FPGAs, enabling deterministic, parallel data stream processing.

#### Power Supply Layout (Altium Designer - Top & Bottom View)

<img width="1267" height="787" alt="HP_Top" src="https://github.com/user-attachments/assets/7ecae9f4-9807-43c8-bf2d-e1a1280b4272" />
<img width="1226" height="549" alt="HP_Bottum" src="https://github.com/user-attachments/assets/e80a2064-5af4-410a-8f3d-e8c8b57a4b2a" />

---

#### Code Snippet (Demonstrating Control Loop Verilog Code for the Intel MAX 10 FPGA)

```verilog
module dac_adjuster (
    input clk,
    input [15:0] dac_in,
    input signed [15:0] diff, 
    output reg [15:0] dac_out
);

    reg signed [15:0] step;
    reg signed [16:0] temp_out; // 17-bit to prevent arithmetic overflow
	 
    always @(posedge clk) begin
        // Determine dynamic step size based on error delta
        if (diff > 50 || diff < -50) begin
            step = diff >>> 1; // Arithmetic right shift for fast division
            
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

        // 17-bit signed calculation to prevent clipping errors
        temp_out = \(signed({1'b0, dac_in}) -\)signed(step);

        // Saturation / Boundary clamping
        if (temp_out < 17'sd32768) begin
            dac_out <= 16'd32768;      // Lower bound (Mid-scale)
        end
        else if (temp_out > 17'sd65535) begin
            dac_out <= 16'hFFFF;       // Upper bound (Max)
        end
        else begin
            dac_out <= temp_out[15:0]; // Valid range assign
        end
    end
endmodule
```

---

## 2. High-Speed Strain Gauge Measurement System

Developed a 16-channel strain gauge DAQ combining AD7768 ADCs and AD8221 instrumentation amplifiers, achieving microvolt-level input accuracy across a 4.096V range. 

* **Overview:** 16 Channels, 256 kSPS, Sinusoidal Excitation, Analog Synchronous Demodulation, Hardware DSP, Ethernet & USB telemetry.
* **Design Challenge:** The ADCs output 8 channels of 4-byte data simultaneously (32 bytes total) over the bus. The standard STM32 SPI peripheral cannot easily handle this arbitrary word length efficiently in high-speed DMA mode without CPU overhead.
* **Engineering Solution:** Reconfigured the MCU's Serial Audio Interface (SAI) peripheral to act as the primary data receiver. The SAI’s native 32-byte DMA FIFO allowed continuous, zero-CPU-overhead data streaming, keeping the Cortex-M7 core free for real-time DSP (FIR filtering) and communication stacks.
* **Firmware Architecture:** Developed bare-metal drivers for the ADCs, lwIP Ethernet stack, USB CDC, and a fixed-point FIR filter cascade.

#### Strain Gauge System Layout (Altium Designer)

<img width="1287" height="615" alt="AI_Shield_Layer_1" src="https://github.com/user-attachments/assets/8977fb7f-2260-4939-aab3-e5e947de58c1" />
<img width="1257" height="601" alt="AI_Shield_Altium" src="https://github.com/user-attachments/assets/a63ca2ed-b8a2-41ef-aef9-6a7aa2f07407" />

---

## 3. Qt6 Real-Time Telemetry GUI

* **Architecture:** Developed a fully modular, multi-threaded Qt6 (C++) GUI for real-time data visualization and high-throughput logging. 
* **Dynamic Configuration:** The application parses JSON configuration files on startup to dynamically build the UI pipeline, defining data interfaces, sampling rates, and layout structures.
* **Performance:** Telemetry parsing and HDF5 file logging run on dedicated worker threads to ensure the main UI thread remains responsive at high data rates.
* **UI Features:** Supports dynamic plotting, rolling charts, progress indicators, and custom analog gauge widgets. Cross-platform compatibility verified on Linux and Windows.
 
#### Code Snippet (Demonstrating of Costumization of a Chart Instrument)

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

#### Code Snippet (Demonstrating of Multithreading and Data handling)

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
#### Example Layout of my GUI

<img width="1542" height="802" alt="image" src="https://github.com/user-attachments/assets/18320425-3b03-46a9-8f11-28afd02d6e2e" />

* The GUI can Receive Data via Ethernet and USB.
* It works on Linux and Windows PCs. 

---

## Full-System Validation: Real-World Aerodynamic Analytics

* **The Objective:** Formulated a hypothesis to optimize fleet wake-efficiency by mapping dynamic, real-time vehicle surface pressure distributions, specifically analyzing aerodynamic boundary layer interferences during overtaking maneuvers.
* **Sensor Integration:** Deployed strain-gauge-based pressure sensors integrated into my DAQ system.
* **Real-World Proof of Concept:** Conducted dynamic road tests using a 5-sensor instrument cluster configuration. The system successfully captured microvolt-level transient pressure drops and wake-flux disruptions between two vehicles during a 70 km/h overtaking maneuver.
* **Future Outlook:** The architecture is fully prepared to scale into a high-density 100+ sensor array to map full-body dynamic airflow topologies and pinpoint absolute aerodynamic efficiency peaks.

#### Telemetry GUI Dashboard (Configured for the Dynamic Road Test)

<img width="1908" height="994" alt="image" src="https://github.com/user-attachments/assets/a64497e5-16cd-40f9-8b72-73137e589cec" />

---

## Additional Skills & Hands-on Experience

* **3D Printing:** Proficient in operating and maintaining 3D printers, handling various filaments, and optimizing models for functional prototyping.
* **Laser Cutting** Built a custom 100W CO2 laser cutter from scratch; experienced in high-power optics alignment, digital vector design, and material processing.
* **Power Tools:** Fully confident and experienced in the safe operation of a wide range of industrial and workshop power tools for fabrication.

---


## 📄 Engineering Portfolio & IP Notice

Due to intellectual property protection and commercialization pathways, my core source code and schematic repositories remain private. 

**Deep-dives into specific hardware design blocks, firmware architectures, or code segments can be provided via screen-share during technical interviews.**


