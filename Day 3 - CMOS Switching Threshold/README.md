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
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description

XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15

Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.01
.control
run
setplot dc1
display
.endc

.end

```

**2. Netlist for Transient Simulation (Dynamic Analysis)**
This netlist uses a `PULSE` input to simulate the inverter's response over time and measure delays.

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description

XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

*simulation commands
.tran 1n 10n
.control
run
.endc
.end

```

### ⚡ SPICE `PULSE` Parameters

The `PULSE` function in your SPICE netlist defines a repeating voltage pulse, which is essential for dynamic (transient) analysis to measure properties like delay. For your specific line, `PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)`, the parameters are:

* **Initial Voltage (`0V`):** The voltage level before the pulse begins.
* **Pulsed Voltage (`1.8V`):** The peak voltage level of the pulse.
* **Delay (`0`):** The time delay before the first pulse starts.
* **Rise Time (`0.1ns`):** The time it takes for the voltage to transition from the initial to the pulsed level.
* **Fall Time (`0.1ns`):** The time it takes for the voltage to transition from the pulsed back to the initial level.
* **Pulse Width (`2ns`):** The duration the signal stays at the pulsed voltage level.
* **Period (`4ns`):** The total time for one complete cycle of the pulse.

---

### 📊 Image Analysis: Inverter Sizing and VTC
> <img width="1920" height="1080" alt="statc behaviour evaluation" src="https://github.com/user-attachments/assets/6cb2a5db-dac8-499e-b51b-55315e257a4f" />

This image visually demonstrates how **transistor sizing** directly impacts an inverter's static behavior, specifically its **Voltage Transfer Characteristic (VTC)** and **switching threshold ($V_M$)**.


* **Left Plot:** This shows the VTC for an inverter where the PMOS and NMOS transistors have the **same W/L ratio** (1.5). The switching threshold (the center of the steep transition) is visibly to the left of the midpoint (0.9V), indicating that the NMOS is stronger than the PMOS for the same dimensions.

* **Right Plot:** Here, the **PMOS width has been increased**, making its W/L ratio 2.5 times larger than the NMOS (3.75 vs 1.5). This makes the pull-up network stronger.

* **Key Takeaway:** By strengthening the PMOS, the **switching threshold ($V_M$) is shifted to a higher voltage**, closer to the ideal center of $V_{dd}/2$. This technique is used to create a more symmetric VTC, which improves the inverter's noise margins and helps balance its rise and fall delays.
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

-----

### The Switching Threshold ($V_M$)

The **switching threshold ($V_M$)** is the input voltage at which $V_{in} = V_{out}$. This point represents the center of the logic transition and is a key indicator of an inverter's robustness. An ideal inverter has a $V_M = V_{dd}/2$.

Our simulation with $W/L = 0.375u/0.25u$ for both transistors yields a $V_M \approx 0.876V$. Since this is less than the ideal $V_{dd}/2 = 1.25V$, it indicates the NMOS is stronger than the PMOS for the same dimensions.

*<img width="1920" height="1080" alt="vgs = vds" src="https://github.com/user-attachments/assets/87cf410c-5eb6-484a-832c-fca38d81fed0" />*
*As shown in the VTC plot, the curve's shape is determined by which operating region (cutoff, linear, or saturation) the PMOS and NMOS transistors are in at each input voltage.*

-----

### Analytical Derivation of $V_M$ 🧪

The value of $V_M$ is determined by the relative current-driving strengths of the PMOS and NMOS transistors. At the switching threshold, both transistors are in the saturation region, and their currents must be equal.
*<img width="1920" height="1080" alt="derivation for vm" src="https://github.com/user-attachments/assets/1d770009-b064-4503-84bf-e269f8a3c0c0" />*
The derivation starts from the equilibrium condition:
$$I_{dsn} = |I_{dsp}| \quad \text{which is equivalent to} \quad I_{dsn} + I_{dsp} = 0$$

Using the velocity-saturated current model (where $V_{gt} = V_{gs} - V_t$):

* **For NMOS**, where $V_{gsn} = V_M$:
  
    $$I_{dsn} = k_n \cdot \left[ (V_M - V_{tn})V_{dsatn} - \frac{V_{dsatn}^2}{2} \right]$$

* **For PMOS**, where $V_{gsp} = V_M - V_{dd}$:
  
    $$I_{dsp} = k_p \cdot \left[ (V_M - V_{dd} - V_{tp})V_{dsatp} - \frac{V_{dsatp}^2}{2} \right]$$

Setting the sum of the currents to zero and solving for $V_M$ yields the final expression, which is dependent on a ratio `R` that encapsulates the relative strengths of the devices: 

$$R = \frac{k_p \cdot V_{dsatp}}{k_n \cdot V_{dsatn}} = \frac{\mu_p C_{ox} (\frac{W}{L})_p \cdot V_{dsatp}}{\mu_n C_{ox} (\frac{W}{L})_n \cdot V_{dsatn}} = \frac{(\frac{W}{L})_p}{(\frac{W}{L})_n} \cdot \frac{k_p' V_{dsatp}}{k_n' V_{dsatn}}$$


This derivation proves that the switching threshold $V_M$ is fundamentally determined by the **W/L sizing ratio** of the PMOS to the NMOS.

### Impact of Transistor Sizing

The analytical formula is confirmed through simulation. By varying the PMOS width and running both static (DC) and dynamic (transient) analyses, we can observe the direct impact on inverter performance.

*<img width="1920" height="1080" alt="comparison table" src="https://github.com/user-attachments/assets/13aefe09-1574-47bb-8f4b-7d5ff5b9a3f3" />*

  * **Static Impact ($V_M$):** As the PMOS width `Wp` increases, the ratio `R` increases, which mathematically and experimentally pushes the switching threshold $V_M$ to a higher voltage. This is because a stronger pull-up network requires a higher input voltage to be overcome by the pull-down network.

  * **Dynamic Impact (Delay):** As the PMOS gets stronger, the inverter's **rise delay decreases** (faster pull-up), but its **fall delay increases**. The table shows that a ratio of $W_p / L_p \approx 2 \times W_n / L_n$ results in nearly **equal rise and fall delays**, a key characteristic for components like clock buffers. Our own simulation measured a rise delay of **330 ps**.

### Applications in Clock Networks 🕒

This concept of balanced sizing is critical in real-world applications, especially in **clock distribution networks**.


*<img width="1920" height="1080" alt="Screenshot 2025-10-18 175124" src="https://github.com/user-attachments/assets/7b424ce8-058f-493b-8c30-72411fad5dc5" />*

Inverters are used as **clock buffers** to regenerate the clock signal and drive it across the chip. For a clock signal to be reliable, its high and low phases must have equal duration (a 50% duty cycle). If a buffer has unequal rise and fall times, it will cause **duty cycle distortion**. By sizing the inverters for balanced delays, we ensure the clock signal's integrity. These characterized delays are also fundamental inputs for **Static Timing Analysis (STA)**, which verifies that all signals in the chip arrive at their destinations on time.

### 🏁 Conclusions

The switching threshold ($V_M$) of a CMOS inverter is a critical static parameter that is directly controlled by the W/L sizing ratio of its PMOS and NMOS transistors, as confirmed by both simulation and analytical derivation. Dynamic simulations are essential for characterizing the inverter's speed, which is heavily dependent on transistor sizing and load capacitance. These fundamental characteristics make the inverter a crucial element in timing-critical applications like clock networks and form the basis for all Static Timing Analysis.
