# Day 1: MOSFET I-V Characteristics

### Introduction / Background

This experiment is the first step in understanding the analog foundation of digital circuits. The objective is to characterize the fundamental current-voltage (I-V) behavior of an N-channel MOSFET (NMOS), which serves as the primary component in the pull-down network of CMOS logic.

**Why is this important?** High-level tools like Static Timing Analysis (STA) rely on pre-characterized models (in `.lib` files) that contain timing information like propagation delay. This timing data is not arbitrary; it is derived directly from SPICE simulations of these fundamental transistor characteristics. The **drive strength**, or the amount of current a transistor can provide, dictates how quickly it can charge or discharge a capacitive load, which directly translates to circuit speed.

By simulating the NMOS transistor, we can observe its behavior in different operating regions. The key principle is **strong inversion**: when the gate-to-source voltage ($V_{gs}$) exceeds a specific **threshold voltage ($V_t$)**, the p-type substrate beneath the gate inverts to form an n-type conducting channel, allowing current to flow from drain to source. This experiment visualizes that current flow.

-----

### SPICE Netlist

To perform the simulation, a SPICE netlist is created. This file describes the circuit components, their connections (nodes), and the analysis to be performed. The simulation sweeps the drain-to-source voltage ($V_{ds}$) from 0V to 1.8V and steps the gate-to-source voltage ($V_{gs}$) to generate the characteristic I-V curves.

The netlist used for this simulation is `day1_nfet_idvds_L2_W5.spice`.

```spice
* Week4: Day1: NMOS Id vs Vds characteristics (L=2um, W=5um)

* Include the SKY130 device models
.include ../../sky130.lib.spice

* Define the circuit components
* Format: M<name> <drain> <gate> <source> <body> <model> W=<width> L=<length>
M1 Vd Vg 0 0 sky130_fd_pr__nfet_01v8 L=2u W=5u

* Define the voltage sources
* Format: V<name> <positive_node> <negative_node> <value>
Vd Vd 0 0       ; Drain voltage source (swept by .dc command)
Vg Vg 0 0       ; Gate voltage source (stepped by .dc command)

* Define the analysis to be performed
* Sweep Vd from 0 to 1.8V in 0.01V increments
* Step Vg from 0 to 1.8V in 0.3V increments
.dc Vd 0 1.8 0.01 Vg 0 1.8 0.3

.end
```

-----

### Plots & Figures

The simulation generates a family of curves, where each curve represents the relationship between the drain current ($I_d$) and the drain-source voltage ($V_{ds}$) for a constant gate-source voltage ($V_{gs}$).

**Figure 1: I-V Characteristic Curves for NMOS Transistor**

*Caption: The plot shows the drain current (Id) as a function of drain-source voltage (Vds). Each curve corresponds to a different gate-source voltage (Vgs), from 0V up to 1.8V. The linear and saturation regions are clearly visible.*

-----

### Tabulated Results

For this initial characterization experiment, the primary deliverable is the graphical plot. Quantitative data extraction (like $V_t$ or delays) will be performed in subsequent experiments.

-----

### Observations and Analysis

The I-V curves in Figure 1 perfectly illustrate the theoretical operating regions of a MOSFET, which are derived from fundamental device physics.

#### 1\. Device Physics and Current Flow

The current in a MOSFET is predominantly **Drift Current**, caused by charge carriers (electrons) moving under the influence of the electric field between the drain and source.
The current at any point `x` along the channel is a product of the available charge and its velocity:
$$I_d = W \cdot Q_i(x) \cdot v(x)$$
Where:

  - $W$ is the channel width.
  - $Q_i(x)$ is the charge density in the inversion layer at point `x`.
  - $v(x)$ is the electron drift velocity, $v(x) = \mu_n E(x) = \mu_n \frac{dV}{dx}$.

#### 2\. The Linear (or Resistive/Triode) Region

  * **Condition:** This region occurs when $V_{gs} > V_t$ and the drain voltage is low, specifically $V_{ds} < (V_{gs} - V_t)$.
  * **Physical Behavior:** In this state, a continuous conductive channel exists from source to drain. The transistor acts like a voltage-controlled resistor. As seen in the plot, for small values of $V_{ds}$, the current $I_d$ increases almost linearly.
  * **Derivation:** By integrating the current equation along the channel length, we get the well-known formula for the linear region:
    $$I_d = \mu_n C_{ox} \frac{W}{L} \left[ (V_{gs} - V_t)V_{ds} - \frac{V_{ds}^2}{2} \right]$$
    Here, $\mu_n C_{ox}$ is also known as the process transconductance parameter, $k_n'$.

#### 3\. The Saturation Region and "Pinch-Off"

  * **Condition:** This region begins when $V_{ds} \ge (V_{gs} - V_t)$.
  * **Physical Behavior:** As $V_{ds}$ increases, the voltage drop across the channel becomes larger. The potential difference between the gate and the channel decreases as we move towards the drain. When the channel voltage near the drain, $V(L)$, reaches $V_{gs} - V_t$, the inversion layer at that point disappears. This is called **"pinch-off"**.
  * **Observation:** In the plot, this corresponds to the "knee" of the curves, after which the current flattens out and becomes largely independent of $V_{ds}$. The transistor now acts as a constant current source, controlled by $V_{gs}$.
  * **Ideal Equation:** The ideal current in saturation is given by:
    $$I_d = \frac{1}{2} \mu_n C_{ox} \frac{W}{L} (V_{gs} - V_t)^2$$

#### 4\. Channel Length Modulation (A Second-Order Effect)

Ideally, the current in saturation should be perfectly flat. However, the plot shows a slight positive slope. This is due to **Channel Length Modulation**. As $V_{ds}$ increases beyond the saturation point, the pinch-off point moves slightly away from the drain towards the source, reducing the *effective* channel length ($L_{eff}$). Since $I_d$ is inversely proportional to length, the current increases slightly.

This effect is modeled by the parameter $\lambda$:
$$I_d = \frac{1}{2} k_n' \frac{W}{L} (V_{gs} - V_t)^2 (1 + \lambda V_{ds})$$

#### 5\. How this Ties Back to STA Concepts

The analysis of these I-V curves is the very first step in chip design.

  * **Drive Strength:** The saturation current ($I_{dsat}$) is the maximum current a transistor can deliver for a given $V_{gs}$. This value is a direct measure of its **drive strength**.
  * **Delay Calculation:** In digital circuits, transistors are constantly charging and discharging the capacitance of subsequent gates. The time taken for this is the propagation delay ($t_p \propto \frac{CV}{I}$). A higher drive strength ($I_{dsat}$) means the current is larger, and therefore the delay is smaller.
  * **STA Models:** The complex delay models used by STA tools are essentially sophisticated lookup tables built from thousands of SPICE simulations like this one, capturing how drive strength changes with input slew, output load, voltage, and temperature. This experiment validates the fundamental physical behavior that those models represent.
