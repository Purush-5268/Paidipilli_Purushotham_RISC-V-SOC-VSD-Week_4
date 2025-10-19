# Day 5: Power Supply & Device Variation Robustness ⚡

### 🎯 Introduction / Background

This experiment evaluates the **robustness** of the CMOS inverter against two critical real-world challenges: **power supply fluctuations** and **manufacturing process variations**. While logic gates are designed for nominal conditions, they must remain functional and reliable across a range of operating voltages and with inherent imperfections from fabrication.

We will conduct two main investigations:

1.  **Power Supply Variation:** Analyze how the inverter's static characteristics (VTC, gain, noise margins) change as the supply voltage ($V_{dd}$) is scaled.
2.  **Device Parameter Variation:** Analyze the inverter's tolerance to physical changes in the transistors themselves, simulating the effects of common manufacturing variations.

-----

### 📜 SPICE Netlists / Code

SPICE simulations are run to generate VTCs under different conditions.

**1. Netlist for Power Supply Variation**
This netlist uses a `.control` loop to automatically run the VTC simulation at different `Vdd` levels and plot them together.


```spice
* Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

.control
  let powersupply = 1.8
  let voltagesupplyvariation = 0
  dowhile voltagesupplyvariation < 6
    dc Vin 0 1.8 0.01
    let powersupply = powersupply - 0.2
    alter Vdd = powersupply
    let voltagesupplyvariation = voltagesupplyvariation + 1
  end
  plot dc1.out vs in dc2.out vs in ...
.endc
.end
```
**To Run This**
```bash
ngspice filename.spice
//graph will pop out
```
> <img width="1853" height="235" alt="Screenshot from 2025-10-19 15-56-43" src="https://github.com/user-attachments/assets/94a9ce3f-d9fd-4664-8e02-5f4b2da867c3" />

**2. Netlist for Device Variation**
This netlist demonstrates an extreme case of device variation, with a PMOS width (`w=7`) much larger than the NMOS width (`w=0.42`).

```spice
* Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=7 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.42 l=0.15

Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.01

.end
```

**To run the above**
```bash
ngspice filename.spice
plot out vs in
```
-----

### 📊 Plots & Figures

**Figure 1: VTC as a Function of Supply Voltage**
This plot is the direct output of the power supply variation simulation, showing how the inverter's transfer curve changes as `Vdd` is reduced.

**<img width="1920" height="1080" alt="Screenshot from 2025-10-19 15-48-56" src="https://github.com/user-attachments/assets/e263adda-343c-4858-ad76-1f3bac70576e" />**

**Figure 2: A Single Inverter VTC**
This plot shows the output of a single VTC simulation, which is the baseline for the variation studies.

**<img width="1920" height="1080" alt="Screenshot from 2025-10-19 15-48-12" src="https://github.com/user-attachments/assets/e29cb8e5-3595-4d4e-aae1-386fb1f274b1" />**

-----

### 🧠 Observations / Analysis

#### Part A: Robustness to Power Supply Variation

Simulating the inverter at different supply voltages reveals a key design trade-off between power, performance, and robustness.

**<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ad5f3972-d1a4-4d71-8e7a-0dd977bdd0f7" />**

  * **Simulation Results:** As `Vdd` is scaled down, the VTC curves become steeper (higher gain) but the overall voltage swing is reduced. This directly impacts the absolute noise margins.
  * **Advantages of Low `Vdd`:**
      * **Lower Energy Consumption:** Energy per transition is proportional to $V_{dd}^2$, so reducing `Vdd` from 2.5V to 0.5V can yield a \~96% energy reduction.
      * **Increased Gain:** The VTC becomes steeper in the transition region, leading to higher voltage gain.
  * **Disadvantages of Low `Vdd`:**
      * **Performance Loss:** The current drive of the transistors is reduced, leading to significantly longer rise/fall delays and slower circuit operation.
      * **Reduced Noise Margins:** The smaller signal swing makes the circuit more susceptible to noise.

#### Part B: Robustness to Device Variation

  * **Sources of Variation:** No manufacturing process is perfect.

    1.  **Etching Process:** Variations in etching can alter the final width (`W`) and length (`L`) of a transistor's gate.
    2.  **Oxide Thickness ($t_{ox}$):** Variations in the gate oxide thickness directly impact the transistor's threshold voltage ($V_t$) and gate capacitance.
        Both of these effects cause the transistor's drain current ($I_d$) to deviate from its designed value.

  * **Simulation Observations:** The CMOS inverter is remarkably robust to these variations. The table below summarizes the behavior under two extreme sizing conditions:

| Case | PMOS Width (µm) | NMOS Width (µm) | Switching Threshold ($V_M$) |
| :--- | :--- | :--- | :--- |
| **Strong PMOS** / Weak NMOS | 1.875 | 0.375 | \~0.2 V |
| **Weak PMOS** / **Strong NMOS** | 0.375 | 1.875 | \~1.4 V |

Even under these extreme mismatches, the inverter remains functional. The primary effect is a large **shift in the switching threshold ($V_M$)** and an **asymmetry in the noise margins**, but not a complete failure of the logic gate. This inherent tolerance is a major advantage of the ratioless CMOS design methodology.

-----

### 🏁 Conclusions

This experiment demonstrates the impact of power supply and device variations on CMOS inverter behavior.

  * Lowering the **supply voltage** improves power efficiency but degrades speed and noise margins, presenting a critical design trade-off.
  * **Device-level process variations** (in W, L, or $t_{ox}$) cause threshold shifts and mismatches.
    However, the key takeaway is that the CMOS inverter is an **inherently robust circuit** that can tolerate significant variations without failing. Understanding these effects is essential for designing reliable logic circuits in advanced process technologies.
