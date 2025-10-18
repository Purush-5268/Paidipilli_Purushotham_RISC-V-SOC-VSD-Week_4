# Day 4: CMOS Inverter Noise Margin & Robustness Evaluation 🛡️

### 🎯 Introduction / Background

This experiment focuses on a critical aspect of digital logic design: **robustness**. We will evaluate the **Noise Margin** of a CMOS inverter, which quantifies its ability to tolerate voltage fluctuations at its input without producing an incorrect logical output.

In the real world, digital signals are affected by noise from sources like power supply variations, crosstalk, and glitches. A gate with a high noise margin is more resilient to these disturbances. We will analyze the inverter's Voltage Transfer Characteristic (VTC) to extract the key voltage parameters that define its noise margins for both logic high and logic low levels.

-----

### 📜 SPICE Netlists / Code

To generate the VTC and extract the noise margin parameters, a DC sweep simulation is performed on the inverter. The simulation command is `ngspice day4_inv_noisemargin_wp1_wn036.spice`.

```spice
* Day 4: CMOS Inverter VTC Simulation for Noise Margin Analysis
.include ../../sky130.lib.spice

* Transistors: M<name> <drain> <gate> <source> <body> <model>
* Adjust Wp/Wn ratio to observe impact on noise margins
M_NMOS vout vin 0 0 sky130_fd_pr__nfet_01v8 L=0.15u W=0.36u
M_PMOS vout vin vdd vdd sky130_fd_pr__pfet_01v8 L=0.15u W=0.84u

* Power Supply
Vdd vdd 0 1.8

* Input Voltage Source to be swept
Vin vin 0 0

* .dc analysis to sweep the input voltage from 0 to Vdd
.dc Vin 0 1.8 0.01

.end
```

-----

### 📊 Plots & Figures

**Figure 1: Annotated VTC for Noise Margin Extraction**

> 🖼️ *\<-- Place your VTC plot here. It should be annotated to clearly show all four key voltage points: VOH, VOL, VIH, and VIL. --\>*

*Caption: The VTC of the CMOS inverter. The points where the slope of the curve equals -1 define the critical input voltages $V_{IL}$ and $V_{IH}$, which are used to determine the noise margins.*

-----

### 📋 Tabulated Results

The key voltage parameters are extracted from the VTC, and the noise margins are calculated.

| Parameter | Description | Extracted Value (V) |
| :--- | :--- | :--- |
| **$V_{OH}$** | Maximum Output High Voltage | `[Your Value]` |
| **$V_{OL}$** | Minimum Output Low Voltage | `[Your Value]` |
| **$V_{IH}$** | Min Input High Voltage (slope = -1) | `[Your Value]` |
| **$V_{IL}$** | Max Input Low Voltage (slope = -1) | `[YourValue]` |
| **$NM_H$** | **High Noise Margin ($V_{OH} - V_{IH}$)** | **`[Your Calculated Value]`** |
| **$NM_L$** | **Low Noise Margin ($V_{IL} - V_{OL}$)** | **`[Your Calculated Value]`** |

-----

### 🧠 Observations / Analysis

#### 1\. Defining the Noise Margin Voltage Parameters

The robustness of an inverter is determined by the size of the "forbidden zones" for its input voltage. These zones are defined by four key parameters extracted from the VTC:

  * **$V_{OH}$ (Output High Voltage):** The maximum voltage at the output when the input is at a valid logic '0'. Ideally, $V_{OH} = V_{dd}$.
  * **$V_{OL}$ (Output Low Voltage):** The minimum voltage at the output when the input is at a valid logic '1'. Ideally, $V_{OL} = 0$.
  * **$V_{IL}$ (Input Low Voltage):** The maximum input voltage that will still be recognized as a logic '0'. This is formally defined as the point on the VTC where the slope `(dVout/dVin)` is equal to -1.
  * **$V_{IH}$ (Input High Voltage):** The minimum input voltage that will still be recognized as a logic '1'. This is the other point on the VTC where the slope is -1.

#### 2\. Calculating the Noise Margins

Using these parameters, we can calculate the noise margins for both logic states. These margins represent the "buffer" the gate has against noise.

  * **High Noise Margin ($NM_H$):** This is the tolerance to noise when the input is high.
    $$NM_H = V_{OH} - V_{IH}$$
  * **Low Noise Margin ($NM_L$):** This is the tolerance to noise when the input is low.
    $$NM_L = V_{IL} - V_{OL}$$
    A larger noise margin indicates a more robust and reliable inverter.

#### 3\. Impact of PMOS Sizing on Noise Margins

The shape of the VTC, and therefore the noise margins, can be manipulated by adjusting the **Wp/Wn sizing ratio** of the transistors.

  * Changing the ratio shifts the switching threshold ($V_M$).
  * Shifting $V_M$ away from the center ($V_{dd}/2$) tends to **increase one noise margin at the expense of the other**. For example, a stronger PMOS increases $V_M$, which typically improves $NM_L$ but degrades $NM_H$.
  * A key design goal is to size the inverter to achieve balanced and sufficiently large noise margins for the target application.

-----

### 🏁 Conclusions

The noise margin is a critical metric that defines an inverter's robustness to signal disturbances. Through SPICE simulation and analysis of the VTC, we can extract the key voltage parameters ($V_{OH}, V_{OL}, V_{IH}, V_{IL}$) and calculate the high and low noise margins. This experiment demonstrates that these margins are directly influenced by transistor sizing (the Wp/Wn ratio), highlighting a fundamental trade-off that designers must manage to create reliable CMOS logic circuits.

-----

### 📚 References / Citations

  * *(e.g., Sky130 PDK Documentation)*
  * *(e.g., Link to any reference repositories or literature used)*
