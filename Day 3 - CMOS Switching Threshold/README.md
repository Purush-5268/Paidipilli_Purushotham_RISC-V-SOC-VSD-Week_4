# Day 3: Inverter VTC, Switching Threshold, and Dynamic Analysis ⚙️

### 🎯 Introduction / Background

Day 3 transitions from the *theory* of the CMOS inverter (from Day 2) to its practical **simulation and quantitative analysis**. The primary objective is to perform SPICE simulations to generate the inverter's Voltage Transfer Characteristic (VTC) and analyze its static and dynamic behavior.

We will focus heavily on the **Switching Threshold ($V_M$)**, the critical point where $V_{in} = V_{out}$. We will reproduce the complete analytical derivation for $V_M$ to understand its dependence on transistor sizing. Finally, we will perform a dynamic (transient) simulation to measure delay and discuss the inverter's applications in timing-critical circuits like clock networks.

-----

### 📜 SPICE Netlists / Code

Two primary simulations were performed: a DC sweep for static analysis and a transient simulation for dynamic analysis.

**1. Netlist for VTC Simulation (Static Analysis)**
This netlist sweeps the input voltage from 0 to 2.5V to generate the VTC curve and find the switching threshold.

```spice
* Day 3: CMOS Inverter VTC Simulation for Vm Extraction
.include sky130.lib.spice

* Transistor Definitions: M<name> <drain> <gate> <source> <body> <model>
M1 out in vdd vdd sky130_fd_pr__pfet_01v8 W=0.375u L=0.25u
M2 out in 0 0 sky130_fd_pr__nfet_01v8 W=0.375u L=0.25u

* Load Capacitor
Cload out 0 10f

* Power and Input Voltage Sources
Vdd vdd 0 2.5
Vin in 0 2.5

* DC Analysis: Sweep Vin from 0 to 2.5V
.dc Vin 0 2.5 0.05

.end
```

**2. Netlist for Transient Simulation (Dynamic Analysis)**
This netlist uses a `PULSE` input to simulate the inverter's response over time and measure delays.

```spice
* Day 3: Transient simulation with a PULSE input to measure delay
.include sky130.lib.spice

* Circuit components are the same as above (M1, M2, Cload, Vdd)

* Pulsed Input Voltage Source
* Format: PULSE(V1 V2 Tdelay Trise Tfall Ton Tperiod)
Vin in 0 pulse(0 2.5 0 10p 10p 1n 2n)

* Transient Analysis: Run simulation for 4ns with 1ps steps
.tran 1p 4n

.end
```

**`PULSE` Parameters Explained:**

  * `0`: Initial voltage (0V)
  * `2.5`: Pulsed voltage (2.5V)
  * `0`: Delay before pulse starts (0s)
  * `10p`: Rise time (10ps)
  * `10p`: Fall time (10ps)
  * `1n`: Pulse width (1ns)
  * `2n`: Pulse period (2ns)

-----

### 📊 Plots & Figures

**Figure 1: Annotated VTC Plot**

> 🖼️ *<img width="1920" height="1080" alt="cmos vt curve" src="https://github.com/user-attachments/assets/a8965c13-dc14-4830-b26d-235a46c76dde" />*

**Figure 2: Transient Response**

> 🖼️ *<img width="1920" height="1080" alt="cmos tran" src="https://github.com/user-attachments/assets/b5bc4a50-577a-424e-b64e-a51d8bb091b2" />*

-----

### 📋 Tabulated Results

The following table, based on `comparison table.jpg`, summarizes the simulation results as the PMOS to NMOS width ratio is increased.

| Wp/Lp | x.Wn/Ln | Rise delay | Fall delay | Vm |
| :--- | :--- | :--- | :--- | :--- |
| Wp/Lp | Wn/Ln | 148ps | 71ps | 0.99V |
| **Wp/Lp** | **2Wn/Ln** | **80ps** | **76ps** | **1.2V** |
| Wp/Lp | 3Wn/Ln | 57ps | 80ps | 1.25V |
| Wp/Lp | 4Wn/Ln | 45ps | 84ps | 1.35V |
| Wp/Lp | 5Wn/Ln | 37ps | 88ps | 1.4V |

**Key Observation:** An inverter with a PMOS width approximately twice that of the NMOS (`2Wn/Ln`) achieves nearly **equal rise and fall delays**. This balanced performance is ideal for applications like clock buffers.

-----

## 🧠 Observations and Analysis

The simulation results provide a clear, quantitative look at the inverter's behavior, which is governed by the underlying device physics and transistor sizing.

---

### The Switching Threshold ($V_M$)

The **switching threshold ($V_M$)** is the input voltage at which `$V_{in} = V_{out}$`. This point represents the center of the logic transition and is a key indicator of an inverter's robustness. An ideal inverter has `$V_M = V_{dd}/2`, which gives it symmetric noise margins against unwanted voltage fluctuations.


*(Image: `vgs = vds.jpg`)*

As shown in the VTC plot, the curve's shape is determined by which operating region (cutoff, linear, or saturation) the PMOS and NMOS transistors are in at each input voltage.

---

### Analytical Derivation of $V_M$

The value of $V_M$ is determined by the relative current-driving strengths of the PMOS and NMOS transistors. At the switching threshold, both transistors are in the saturation region, and the pull-up current from the PMOS must exactly balance the pull-down current from the NMOS.


*(Image: `derivation for vm.jpg`)*

The derivation starts from the equilibrium condition:
$$I_{dsn} = |I_{dsp}| \quad \text{or} \quad I_{dsn} + I_{dsp} = 0$$
Using the velocity-saturated current model (where `$V_{gt} = V_{gs} - V_t$`):
* **For NMOS**, where `$V_{gsn} = V_M$`:
    $$I_{dsn} = k_n \cdot \left[ (V_M - V_{tn})V_{dsatn} - \frac{V_{dsatn}^2}{2} \right]$$
* **For PMOS**, where `$V_{gsp} = V_M - V_{dd}$`:
    $$I_{dsp} = k_p \cdot \left[ (V_M - V_{dd} - V_{tp})V_{dsatp} - \frac{V_{dsatp}^2}{2} \right]$$

Setting the sum of the currents to zero and solving for `$V_M$` yields the final expression:
$$V_M = \frac{R \cdot V_{dd} + \dots}{1+R}$$Where the ratio `R` encapsulates the relative strengths of the devices:$$R = \frac{k_p \cdot V_{dsatp}}{k_n \cdot V_{dsatn}} = \frac{\mu_p C_{ox} (\frac{W}{L})_p \cdot V_{dsatp}}{\mu_n C_{ox} (\frac{W}{L})_n \cdot V_{dsatn}} = \frac{(\frac{W}{L})_p}{(\frac{W}{L})_n} \cdot \frac{k_p' V_{dsatp}}{k_n' V_{dsatn}}$$
This derivation proves that the switching threshold `$V_M$` is fundamentally determined by the **W/L sizing ratio** of the PMOS to the NMOS.

---

### Impact of Transistor Sizing

The analytical formula is confirmed through simulation. By varying the PMOS width and running both **static (DC)** and **dynamic (transient)** analyses, we can observe the direct impact on inverter performance.


*(Image: `comparison table.jpg`)*

* **Static Impact ($V_M$):** As the PMOS width `Wp` increases, the ratio `R` increases, which mathematically and experimentally pushes the switching threshold `$V_M$` to a higher voltage. This is because a stronger pull-up network requires a higher input voltage to be overcome by the pull-down network.
* **Dynamic Impact (Delay):** As the PMOS gets stronger, the inverter's **rise delay decreases** (faster pull-up), but its **fall delay increases** (slower pull-down). The table shows that a ratio of `Wp/Lp ≈ 2 * Wn/Ln` results in nearly **equal rise and fall delays**.

---

### Applications in Clock Networks 🕒

This concept of balanced sizing is critical in real-world applications, especially in **clock distribution networks**.


*(Image: `Screenshot 2025-10-18 175124.jpg`)*

Inverters are used as **clock buffers** to regenerate the clock signal and drive it across the chip. For a clock signal to be reliable, its high and low phases must have equal duration (a 50% duty cycle). If a buffer has unequal rise and fall times, it will cause **duty cycle distortion**. By sizing the inverters for balanced delays, we ensure the clock signal's integrity. These characterized delays are also fundamental inputs for **Static Timing Analysis (STA)**, which verifies that all signals in the chip arrive at their destinations on time.

### 🏁 Conclusions

The switching threshold ($V_M$) of a CMOS inverter is a critical static parameter that is directly controlled by the W/L sizing ratio of its PMOS and NMOS transistors, as confirmed by both simulation and analytical derivation. Dynamic simulations are essential for characterizing the inverter's speed, which is heavily dependent on transistor sizing and load capacitance. These fundamental characteristics make the inverter a crucial element in timing-critical applications like clock networks and form the basis for all Static Timing Analysis.
