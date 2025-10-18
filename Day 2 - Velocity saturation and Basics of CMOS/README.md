# Day 2: Velocity Saturation & CMOS Inverter VTC Theory ⚡

### 🎯 Introduction / Background

This experiment investigates two fundamental topics that bridge the gap between single-transistor physics and digital logic-gate behavior.

1.  **Part A - Velocity Saturation**: We will analyze how and why the behavior of modern, short-channel transistors deviates from ideal long-channel models. The focus is on **velocity saturation**, a physical phenomenon that limits the current-carrying capability of a transistor and is a dominant effect in advanced technology nodes. We will conduct SPICE simulations to observe this effect directly.

2.  **Part B - CMOS Inverter VTC Theory**: We will deconstruct the static behavior of the CMOS inverter, the cornerstone of digital logic. This is a **theoretical analysis** of how the inverter's Voltage Transfer Characteristic (VTC) is formed. We will explore the simple switch model and the more detailed load-curve method to understand the principles behind the VTC simulation that will be performed in Day 3.

-----
### 🔬 Simulation Objective: Long vs. Short-Channel Behavior

The objective of this simulation is to analyze the MOSFET's $I_d$ vs. $V_{gs}$ characteristics for different device sizes. This comparison will visually highlight the impact of **short-channel effects**, particularly **velocity saturation**.

We will compare two device cases with the same W/L ratio:
* **Long-Channel Device:** L = 1.2 µm, W = 1.8 µm (W/L = 1.5)
* **Short-Channel Device:** L = 0.25 µm, W = 0.375 µm (W/L = 1.5)

---

#### Expected Behavior

* **Long-Channel Behavior:**
    The device is expected to follow the classical square-law model. The drain current ($I_d$) should show a **quadratic** relationship with the gate-source voltage ($V_{gs}$), where $I_d \propto (V_{gs} - V_t)^2$.

* **Short-Channel Behavior:**
    The device is expected to deviate from the square-law model. At higher values of $V_{gs}$, the current will begin to show a more **linear** relationship. This deviation is the key indicator of **velocity saturation**, where the charge carriers have reached their maximum drift velocity, limiting the current increase.
> <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4602509d-e751-4f5f-ab22-8f640811626b" />

### 📜 SPICE Netlists / Code

Two primary SPICE simulations are run to characterize the NMOS transistor and observe short-channel effects.

**1. Simulation for $I_d$ vs. $V_{ds}$ Characteristics**
This simulation sweeps the drain voltage for different gate voltages to generate the classic I-V characteristic curves.

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2
.control

run
display
setplot dc1
.endc

.end

```

**Explanation:**

  * `.include`: Loads the foundry-provided model file for the SKY130 process.
  * `M1`: Defines our NMOS transistor with a short channel length (`L=0.25u`).
  * `Vdd`, `Vgg`: Define the voltage sources for the drain and gate.
  * `.dc`: This is the analysis command. It instructs SPICE to perform a DC sweep, varying the drain voltage `Vdd` from 0V to 1.8V, and to repeat this entire sweep for several gate voltage `Vgg` steps.

**2. Simulation for $I_d$ vs. $V_{gs}$ Characteristics**
This simulation sweeps the gate voltage at a fixed high drain voltage to analyze the transistor's turn-on behavior and extract its threshold voltage.

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.1 
.control

run
display
setplot dc1
.endc
.end

```

**Explanation:**

  * `Vdd`: The drain voltage is fixed at 1.8V to ensure the transistor is operating in the saturation region once it turns on.
  * `.dc`: This command sweeps the gate voltage `Vg` from 0V to 1.8V, allowing us to plot the current response to the changing gate drive.

-----

### 📊 Plots & Figures

**Figure 1: Id vs Vds**

> 🖼️<img width="1920" height="1080" alt="graph id vs vds" src="https://github.com/user-attachments/assets/13ca9100-80bc-4368-9f72-a131eee6ebdd" />

**Figure 2: Id vs Vgs**

> 🖼️ <img width="1920" height="1080" alt="Graph id vs vgs" src="https://github.com/user-attachments/assets/1b64c8e0-327d-499a-ad8b-5495e742fd88" />
-----

### 📋 Tabulated Results

The key parameter extracted from the $I_d$ vs. $V_{gs}$ analysis is the threshold voltage.

| Parameter | Extracted Value | Units |
| :--- | :--- | :--- |
| **Threshold Voltage ($V_t$)** | `7.69` | V |

-----

### 🧠 Observations / Analysis

#### Part A: Velocity Saturation in Short-Channel Devices

* **The Physical Cause:** In short-channel devices, the high lateral electric field ($\epsilon$) causes carrier velocity to saturate at a maximum speed, **$v_{sat}$**, due to scattering effects. At this point, the velocity is no longer proportional to the electric field.
    > 🖼️ <img width="1920" height="1080" alt="velocity saturation effect" src="https://github.com/user-attachments/assets/89650f55-0705-4f83-a60f-5385aefba7a3" />
    > 🖼️ <img width="1920" height="1080" alt="Screenshot 2025-10-18 105525" src="https://github.com/user-attachments/assets/99c40268-0a16-45a8-84e6-1bf22cb9f42b" />


* **The Unified Current Model:** To account for this, a unified current model is used. This single equation can accurately describe the transistor's behavior across all its operating modes by using a limiting term, $V_{min}$.

    The core of this model is the equation:
    $$I_d = k_n \cdot \left[ (V_{gt} \cdot V_{min}) - \frac{V_{min}^2}{2} \right] \cdot (1 + \lambda V_{ds})$$
    The model's behavior changes based on which term in **$V_{min} = min(V_{gt}, V_{ds}, V_{dsat})$** is the smallest, where $V_{gt} = V_{gs} - V_t$.

     Here are the three cases as derived in your notes:

    1.  **Case: $V_{min} = V_{ds}$ (Linear/Resistive Region)**
        This occurs when $V_{ds}$ is small. Substituting $V_{min}$ with $V_{ds}$ gives the classic formula for the linear region:
        $$I_d = k_n \cdot \left[ (V_{gt} \cdot V_{ds}) - \frac{V_{ds}^2}{2} \right] \cdot (1 + \lambda V_{ds})$$

    2.  **Case: $V_{min} = V_{gt}$ (Standard Saturation Region)**
        This occurs in **long-channel** devices where $V_{ds}$ is large and velocity saturation is not the limiting factor. Substituting $V_{min}$ with $V_{gt}$ gives the square-law saturation equation:
        $$I_d = k_n \cdot \left[ (V_{gt} \cdot V_{gt}) - \frac{V_{gt}^2}{2} \right] \cdot (1 + \lambda V_{ds}) = k_n \cdot \frac{V_{gt}^2}{2} \cdot (1 + \lambda V_{ds})$$

    3.  **Case: $V_{min} = V_{dsat}$ (Velocity Saturation Region)**
        This is the critical case for **short-channel** devices. The current is limited by the velocity saturation voltage ($V_{dsat}$). Substituting $V_{min}$ with $V_{dsat}$ results in an equation that is **linear** with respect to $V_{gt}$:

        $$I_d = k_n \cdot \left[ (V_{gt} \cdot V_{dsat}) - \frac{V_{dsat}^2}{2} \right] \cdot (1 + \lambda V_{ds})$$
        
        This linear dependence is exactly what we observe in our short-channel simulations and is the key takeaway of this analysis.
### Part B: CMOS Inverter VTC Theory 🔄

The static behavior of the CMOS inverter is defined by its Voltage Transfer Characteristic (VTC). We can understand how this curve is formed by starting with a simple switch model and progressing to a more detailed graphical analysis.

#### Model 1: The Inverter as an Ideal Switch 🔌

The most intuitive way to understand the inverter is to model the MOSFETs as simple, voltage-controlled switches.

> 🖼️ <img width="1920" height="1080" alt="Cmos" src="https://github.com/user-attachments/assets/695f0da7-8c62-4467-b8d3-87aa56b10a74" />

* **When `Vin` is low (Logic '0'):** The PMOS transistor turns **ON**, acting like a closed switch to `Vdd`. The NMOS transistor turns **OFF**, acting as an open switch to ground. This creates a low-resistance path to the power supply, pulling the output **`Vout` high to `Vdd`**.

* **When `Vin` is high (Logic '1'):** The opposite occurs. The NMOS is **ON**, creating a path to ground, while the PMOS is **OFF**. This pulls the output **`Vout` low to `Vss` (GND)**.

---

#### Model 2: VTC Construction via Load-Curve Analysis 📈

While the switch model gives the start and end points, the true shape of the VTC is determined by the analog current-voltage (I-V) characteristics of the two transistors.

##### **1. Voltage Definitions**

First, we must define the transistor terminal voltages in terms of the common inverter voltages, `Vin` and `Vout`. The I-V curves for the NMOS and PMOS are the building blocks of our analysis.

> 🖼️ *<img width="1920" height="1080" alt="nmos pmos-cmos" src="https://github.com/user-attachments/assets/bbb6a608-261b-45ec-b4bf-15ce843c0fb2" />*

* **NMOS:**
    * `VgsN = Vin`
    * `VdsN = Vout`
* **PMOS:**
    * `VgsP = Vin − Vdd`
    * `VdsP = Vout − Vdd`

At any stable operating point, the current flowing out of the PMOS must equal the current flowing into the NMOS, so `Idsp = -Idsn`.

##### **2. The Graphical Solution**

The VTC can be constructed graphically by overlaying the I-V characteristics of the pull-up (PMOS) and pull-down (NMOS) transistors.

> 🖼️ *<img width="1920" height="1080" alt="Screenshot 2025-10-18 123820" src="https://github.com/user-attachments/assets/fc6a1d0a-4131-4fc1-825a-79c08073f9c7" />*

* **Load Curves:** For a given input voltage (`Vin`), we plot the drain current `Id` versus the output voltage `Vout` for both transistors. These are the "load curves."
* **Finding Intersections:** The stable operating point for the inverter is the voltage `Vout` where the PMOS and NMOS current curves intersect. At this point, the pull-up and pull-down currents are balanced.
* **Constructing the VTC:** By sweeping `Vin` from 0 to `Vdd` and plotting the resulting intersection point for each step, we trace out the entire VTC curve. As the annotated graph shows, this method also reveals the specific operating region (linear, saturation, or cutoff) for each transistor at every point along the transition.

### 🏁 Conclusions

**Velocity Saturation**: Day 2's simulations confirm that in short-channel MOSFETs, velocity saturation is a dominant effect. It causes the drain current to shift from a **quadratic** to a **linear** relation with gate voltage, a critical deviation from ideal models that must be accounted for in modern design.

**VTC Behavior**: The CMOS inverter’s VTC is a direct result of the intersection of the PMOS and NMOS I–V "load curves". Understanding this graphical construction is key to analyzing a gate's switching threshold and noise margins, which are essential for robust digital logic.

-----

### 📚 References / Citations

  * *Sky130 PDK Documentation*
  * Ngspice User Guide
  * “Digital Integrated Circuits” — Jan M. Rabaey
  * “CMOS Digital Integrated Circuits” — Neil H. E. Weste
