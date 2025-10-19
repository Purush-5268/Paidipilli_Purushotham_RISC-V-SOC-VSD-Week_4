### 🎯 Introduction: CMOS Inverter Robustness

**Noise Margin** is a critical metric that defines a logic gate's robustness against electrical noise like crosstalk and power supply fluctuations. It quantifies the maximum noise voltage a circuit can tolerate on its input while still producing a correct and unambiguous logical output. A design with high noise margins is more reliable.

The inverter's **Voltage Transfer Characteristic (VTC)** is the key to this analysis. While an ideal inverter has an infinitely sharp transition, a practical inverter's VTC is a gradual curve due to the non-zero resistance of the transistors. The shape of this curve determines the noise margin.

> <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2a5c57be-36d6-4818-a648-7243b8768936" />

-----

### 📜 SPICE Netlist for VTC Simulation

The following SPICE deck is used to perform a DC sweep analysis on the CMOS inverter, sweeping the input voltage from 0 to 1.8V to generate the VTC.

```spice
* Model Description
.param temp=27

* Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF

* Voltage Sources
Vdd vdd 0 1.8V
Vin in 0 1.8V

* Simulation commands
.op
.dc Vin 0 1.8 0.01

.control
run
setplot dc1
display
.endc
.end
```
### 📊 Plots & Figures

**Figure 1: Annotated VTC for Noise Margin Extraction**

> 🖼️ *<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6eeeb5a5-276d-4c77-b420-2cc24e5a4d2e" />*

**Figure 2: Spice Netlist Output**

> 🖼️ *<img width="1210" height="773" alt="image" src="https://github.com/user-attachments/assets/5b807dbb-555f-489e-902c-d117df179020" />*


**Code Explanation:**

  * **.lib**: Includes the `tt` (typical-typical) corner models for the Sky130 process.
  * **XM1/XM2**: Define the PMOS and NMOS transistors. Note the sizing ratio: the PMOS width (`w=1u`) is significantly larger than the NMOS (`w=0.36u`) to balance their drive strengths.
  * **Cload**: Represents the capacitive load on the inverter's output.
  * **.dc**: This command instructs Ngspice to perform a DC analysis by sweeping the input voltage `Vin` from 0V to 1.8V in 0.01V increments.

-----
### 📋 Tabulated Results

The quantitative results are broken down into two parts: a detailed analysis of the baseline inverter and a summary of the trends observed when varying the transistor sizing.

---

#### **1. Detailed Analysis of Baseline Inverter**
These are the specific parameters extracted from a single simulation where the PMOS (`Wp=1u`) is significantly wider than the NMOS (`Wn=0.36u`).

| Parameter | Description | Extracted Value (V) |
| :--- | :--- | :--- |
| $V_{OH}$ | Output High Voltage | 1.737 |
| $V_{OL}$ | Output Low Voltage | 0.1125 |
| $V_{IL}$ | Input Low Voltage | 0.7395 |
| $V_{IH}$ | Input High Voltage | 0.9720 |
| **$NM_L$** | **Low Noise Margin ($V_{IL} - V_{OL}$)** | **0.627** |
| **$NM_H$** | **High Noise Margin ($V_{OH} - V_{IH}$)** | **0.765** |

---

#### **2. Impact of PMOS Sizing Variation**
This table summarizes the trend as the PMOS width (`Wp`) is scaled relative to the NMOS width (`Wn`).

| Sizing Ratio (Wp/Wn) | $V_M$ (V) | $NM_L$ (V) | $NM_H$ (V) |
| :--- | :--- | :--- | :--- |
| 1x | 0.99 | 0.30 | 0.30 |
| **2x** | **1.20** | **0.30** | **~0.40** |
| 3x | 1.25 | 0.30 | ~0.40 |
| 4x | 1.35 | 0.30 | ~0.42 |
| 5x | 1.40 | 0.27 | ~0.42 |

**Key Trend:** The table clearly shows that as the PMOS becomes stronger, the switching threshold ($V_M$) increases and the High Noise Margin ($NM_H$) improves, while the Low Noise Margin ($NM_L$) remains stable before slightly degrading at extreme ratios.

---
### Calculating the Noise Margins 🧮

Using the four key voltage parameters extracted from the VTC, we can calculate the noise margins. These values quantify the inverter's tolerance to noise for both logic states.

* **High Noise Margin ($NM_H$):** This is the tolerance to noise when the input is intended to be high.
    $$NM_H = V_{OH} - V_{IH}$$

* **Low Noise Margin ($NM_L$):** This is the tolerance to noise when the input is intended to be low.
    $$NM_L = V_{IL} - V_{OL}$$

A larger value for both $NM_H$ and $NM_L$ indicates a more robust and reliable inverter.

#### **Practical Extraction from the VTC Plot**
In many SPICE waveform viewers, the critical voltage points ($V_{IL}, V_{IH}, V_{OH}, V_{OL}$) can be extracted directly from the VTC plot at the points where the slope (gain) is -1.

The procedure is as follows:
* Find the point on the **upper part of the VTC transition** where the slope is -1. The coordinates correspond to:
    * x-axis value = $V_{IL}$
    * y-axis value = $V_{OH}$
* Find the point on the **lower part of the VTC transition** where the slope is -1. The coordinates correspond to:
    * x-axis value = $V_{IH}$
    * y-axis value = $V_{OL}$

  
#### **2. Calculating the Noise Margins**

The noise margins represent the voltage "buffer" the gate has before an input is misinterpreted.

  * **High Noise Margin ($NM_H$):** The tolerance to noise when the input is intended to be high.
    $$NM_H = V_{OH} - V_{IH}$$
  * **Low Noise Margin ($NM_L$):** The tolerance to noise when the input is intended to be low.
    $$NM_L = V_{IL} - V_{OL}$$
    For reliable operation, both margins must be sufficiently large and positive.

#### **3. Impact of Sizing on Noise Margin**

The shape of the VTC—and thus the noise margins—can be tuned by adjusting the **Wp/Wn sizing ratio**.

  * Making the PMOS stronger (increasing `Wp`) relative to the NMOS will shift the switching threshold higher.
  * This generally improves one noise margin at the expense of the other.
  * A key design task is to size the transistors to achieve balanced and adequate noise margins for the intended application.

-----

### 🏁 Conclusion

The **noise margin** is a fundamental metric for quantifying the robustness of a CMOS inverter. By performing a SPICE simulation to generate the VTC, we can extract the key voltage parameters ($V_{OH}, V_{OL}, V_{IH}, V_{IL}$) and calculate the high and low noise margins. This analysis demonstrates how these margins are a direct consequence of the inverter's static behavior and can be optimized through careful transistor sizing.
