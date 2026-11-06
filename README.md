# Quadrature Down Converter (QDC)

> **Design, Simulation, and Hardware Implementation of an End-to-End Quadrature Down Converter**  
> *Course Project — Analog Electronic Circuits (Spring 2026)*  
> *Department of Electronics and Communication Engineering (ECE), IIIT Hyderabad*  
> **Team Name:** Mixed Signals (Table 8)  
> **Members:**  
> - **Sreekar Reddy Arepally** (2025102074)  
> - **Supratheek SNV** (2025102064)  
> - **Sravan Kumar** (2025102033)  

---

## 📌 Executive Summary

Modern wireless communications (Wi-Fi, 5G, Bluetooth, IoT) rely fundamentally on **Quadrature Down Converters (QDC)** to translate high-frequency Radio Frequency (RF) signals down to baseband or Intermediate Frequencies (IF) while separating incoming signals into two orthogonal components: **In-Phase ($I$)** and **Quadrature ($Q$)**.

This project provides an end-to-end design, analytical mathematical derivation, LTSpice SPICE modeling, and physical breadboard hardware realization of a full Quadrature Down Converter system consisting of:
1. **A 3-OpAmp Quadrature Local Oscillator** operating at $\approx 170.5\text{ kHz}$ with precise $90^\circ$ relative phase separation.
2. **Dual NMOS Switching Mixers** demonstrating heterodyne down-conversion across a $165\text{ kHz} - 175\text{ kHz}$ RF frequency sweep.
3. **Passive RC Low-Pass Filters** with a $3\text{ kHz}$ cut-off frequency to strip high-frequency sum products and LO feedthrough.
4. **Common-Source NMOS Amplifiers** compensating for conversion loss and boosting the IF output to $\ge 500\text{ mV}_{\text{pp}}$.
5. **Physical Hardware Verification** utilizing UA741 op-amps, CD4007 CMOS arrays, and passive components, verified on digital storage oscilloscopes (DSOs) via time-domain waveforms and Fast Fourier Transforms (FFT).

---

## ⚙️ System Architecture

```
                                +--------------------------------------------+
                                |        3-OpAmp Quadrature Oscillator       |
                                |         f_LO ~ 170.5 kHz, 0.7-1.1 Vpp      |
                                +---------------------+----------------------+
                                                      |
                                     v_OSC_I (cos)    |     v_OSC_Q (sin)
                                          |           |           |
                                          v           |           v
    RF Input v_in (169 kHz)       +---------------+   |   +---------------+
    ------------------------------+>  NMOS Switch |   |   |  NMOS Switch  |<-- RF Input v_in (169 kHz)
                                  |   (I-Mixer)   |   |   |   (Q-Mixer)   |
                                  +-------+-------+   |   +-------+-------+
                                          |           |           |
                                          v           |           v
                                   Raw IF + Harmonics |    Raw IF + Harmonics
                                          |           |           |
                                          v           |           v
                                  +---------------+   |   +---------------+
                                  | Passive RC LPF|   |   | Passive RC LPF|
                                  |  (fc ~ 3 kHz) |   |   |  (fc ~ 3 kHz) |
                                  +-------+-------+   |   +-------+-------+
                                          |           |           |
                                          v           |           v
                                     Filtered IF      |      Filtered IF
                                     (~ 50 mVpp)      |      (~ 50 mVpp)
                                          |           |           |
                                          v           |           v
                                  +---------------+   |   +---------------+
                                  | CS NMOS Amp   |   |   | CS NMOS Amp   |
                                  | (Gain ~ 10x)  |   |   | (Gain ~ 10x)  |
                                  +-------+-------+   |   +-------+-------+
                                          |                       |
                                          v                       v
                                     v_IF_FINAL_I            v_IF_FINAL_Q
                                     (1.55 kHz IF,           (1.55 kHz IF,
                                      503.7 mVpp)             491.1 mVpp, -89.4° shifted)
```

---

## 📊 Performance & Specifications: Simulated vs. Measured

The table below contrasts our target design specifications with the values achieved in **LTSpice simulations (TSMC 180nm CMOS / UA741)** and **Laboratory Hardware Realization (CD4007 CMOS / UA741 breadboard)**:

| Parameter | Specification Target | Simulated (LTSpice) | Measured (Hardware) |
|---|---|---|---|
| **Oscillator Frequency ($f_{\text{LO}}$)** | $\approx 170\text{ kHz}$ | $170.55\text{ kHz}$ | $168.08\text{ kHz}$ |
| **Oscillator Amplitude ($I$-phase)** | $\approx 1.0\text{ V}_{\text{p-p}}$ | $0.70\text{ V}_{\text{p-p}}$ | $1.08\text{ V}_{\text{p-p}}$ |
| **Oscillator Amplitude ($Q$-phase)** | $\approx 1.0\text{ V}_{\text{p-p}}$ | $0.70\text{ V}_{\text{p-p}}$ | $1.12\text{ V}_{\text{p-p}}$ |
| **Quadrature Phase Shift ($\Delta\phi_{I/Q}$)** | $90.0^\circ$ | $-89.42^\circ$ | $\approx -90.0^\circ$ |
| **RF Input Frequency ($f_{\text{in}}$)** | $169.0\text{ kHz}$ | $169.0\text{ kHz}$ | $166.70\text{ kHz}$ |
| **RF Input Amplitude ($v_{\text{in}}$)** | $200\text{ mV}_{\text{p-p}}$ | $200\text{ mV}_{\text{p-p}}$ | $212\text{ mV}_{\text{p-p}}$ |
| **Down-Converted IF Frequency ($f_{\text{IF}}$)** | $|f_{\text{LO}} - f_{\text{in}}|$ | $1.55\text{ kHz}$ | $1.39\text{ kHz}$ |
| **Final IF Output Amplitude ($I$-channel)** | $\ge 500\text{ mV}_{\text{p-p}}$ | $503.7\text{ mV}_{\text{p-p}}$ | $520.0\text{ mV}_{\text{p-p}}$ |
| **Final IF Output Amplitude ($Q$-channel)** | $\ge 500\text{ mV}_{\text{p-p}}$ | $491.1\text{ mV}_{\text{p-p}}$ | $480.0\text{ mV}_{\text{p-p}}$ |
| **Oscillator Supply Voltage** | $\pm 12\text{ V}$ | $\pm 12\text{ V}$ | $\pm 12\text{ V}$ |
| **Amplifier Supply Voltage ($V_{DD}$)** | Custom | $1.8\text{ V}$ (180nm) | $5.0\text{ V}$ (CD4007) |
| **Mixer Gate Bias ($V_{\text{BIAS}}$)** | Sub-threshold/Triode | $0.55\text{ V}$ | $\approx 1.5\text{ V}$ |
| **Amplifier Gate Bias ($V_{\text{BIAS}}$)** | Saturation | $0.42\text{ V}$ | $\approx 1.5\text{ V}$ |
| **Mixer Coupling Capacitor ($C_C$)** | AC Coupling | $1\text{ nF}$ | $1\text{ nF}$ |
| **Amplifier Coupling Capacitor ($C_C$)** | DC Blocking | $1\ \mu\text{F}$ | $1\ \mu\text{F}$ |

---

## 🔬 Subsystem Deep Dive

### 1. Quadrature Oscillator (Q2)
- **Topology:** Closed-loop architecture featuring two cascaded inverting integrators ($U_1$ and $U_2$) and an inverting limiter amplifier stage ($U_3$) built with UA741 operational amplifiers.
- **Barkhausen Criterion:** Each integrator stage provides a $90^\circ$ phase shift ($180^\circ$ total). The third op-amp stage contributes the remaining $180^\circ$ phase inversion to achieve total loop phase shift of $360^\circ$ ($0^\circ$) with loop gain $|A\beta| = 1$.
- **Amplitude Limiting:** Two anti-parallel 1N4148 diodes in the feedback network of $U_3$ provide soft-clipping, preventing op-amp rail saturation and preserving waveform linearity.
- **Results:** Stable oscillation at $170.55\text{ kHz}$ in simulation ($-89.42^\circ$ phase difference) and $168.08\text{ kHz}$ in physical breadboard testing ($\approx -90^\circ$ phase difference).

### 2. NMOS Switching Mixer (Q3)
- **Principle:** Operates an NMOS transistor as a voltage-controlled switch in the deep triode region. The gate is driven by the LO carrier ($v_{\text{OSC}}$) while the source receives the input RF voltage ($v_{\text{in}}$).
- **Heterodyning:** The channel conductance modulation multiplies $v_{\text{in}}$ and $v_{\text{OSC}}$, creating difference ($|f_{\text{in}} - f_{\text{LO}}|$) and sum ($f_{\text{in}} + f_{\text{LO}}$) components.
- **Frequency Sweep Characterization:**
  - Tested from $165\text{ kHz}$ to $175\text{ kHz}$ in steps across the $170\text{ kHz}$ LO:
    - $165\text{ kHz} \implies f_{\text{IF}} = 5\text{ kHz}$
    - $168\text{ kHz} \implies f_{\text{IF}} = 2\text{ kHz}$
    - $169\text{ kHz} \implies f_{\text{IF}} = 1\text{ kHz}$
    - $171\text{ kHz} \implies f_{\text{IF}} = 1\text{ kHz}$
    - $172\text{ kHz} \implies f_{\text{IF}} = 2\text{ kHz}$
    - $175\text{ kHz} \implies f_{\text{IF}} = 5\text{ kHz}$
  - Perfectly verified in both LTSpice transient/FFT spectra and DSO lab oscilloscope captures.

### 3. Passive RC Low-Pass Filter (Q4)
- **Design:** Passive 1st-order / cascaded 2nd-order RC filter with target cut-off frequency $f_c = 3\text{ kHz}$ ($R = 53\text{ k}\Omega$, $C = 1\text{ nF}$; physical realization used $56\text{ k}\Omega$ giving $f_c \approx 2.84\text{ kHz}$).
- **Function:** Rejects the high-frequency $\sim 340\text{ kHz}$ sum term and high-frequency switching feedthrough, isolating the smooth baseband/IF sine wave.

### 4. Common-Source NMOS Amplifier (Q4)
- **Design:** Sized to provide voltage gain $A_v = -g_m R_L \approx 10\text{ V/V}$ into a load resistance of $100 - 120\text{ k}\Omega$.
- **Operation:** Transistor biased in the saturation region ($V_{\text{GS}} > V_{\text{th}}$). Successfully amplifies the $\approx 50\text{ mV}_{\text{pp}}$ filtered IF signal up to $503.7\text{ mV}_{\text{pp}}$ (simulated) and $520\text{ mV}_{\text{pp}}$ (measured hardware) without distortion or clipping.

### 5. Full System Integration & Quadrature Verification (Q5)
- **End-to-End Operation:** Connecting the quadrature oscillator outputs to dual mixer branches, followed by separate low-pass filters and tuned common-source amplifier stages.
- **Preservation of Quadrature:** With an RF input of $169\text{ kHz}$, both the simulated and measured circuits yielded balanced $I$ and $Q$ output signals with equal amplitude ($\approx 500\text{ mV}_{\text{pp}}$) and an exact $90^\circ$ orthogonal phase relationship ($-89.42^\circ$ in SPICE, $\approx -90^\circ$ on DSO).

---

## 💡 Practical Hardware Implementation Insights

Transitioning the integrated design from TSMC 180nm SPICE models to physical breadboard prototyping with CD4007 CMOS arrays uncovered critical real-world electronic design principles:

1. **Threshold Voltage Deviation ($V_{\text{th}}$):**
   - The TSMC 180nm model features $V_{\text{th}} \approx 0.4\text{ V}$, allowing operation with $V_{\text{bias}} = 0.55\text{ V}$ and $V_{DD} = 1.8\text{ V}$.
   - The physical CD4007 transistor is a legacy CMOS process with $V_{\text{th}} \approx 1.4\text{ V} - 1.5\text{ V}$. Operating it required increasing gate biasing to $1.5\text{ V} - 2.0\text{ V}$ (using a $100\text{ k}\Omega$ trimpot divider) and scaling the supply to $V_{DD} = 5.0\text{ V}$.
2. **The "Bulk" Connection Trap (Pin 4):**
   - On the CD4007 chip, **Pin 4** is the universal substrate for all NMOS transistors on the die and **must be connected to 0V (Ground)**.
   - If left floating, the internal P-N body diodes forward-bias under AC signal swings, short-circuiting the RF input into the substrate.
3. **Op-Amp Speed Limitations (UA741):**
   - UA741 has an operational Gain-Bandwidth Product (GBWP) around $1\text{ MHz}$. At $170\text{ kHz}$, internal parasitic phase lags shift the nominal oscillation frequency away from ideal $1 / (2\pi RC)$ calculations, necessitating fine-tuning of trimmer resistors on the physical breadboard.

---

## 📁 Repository Structure

```
Quadrature-Down-Converter/
├── README.md                           # Comprehensive documentation & engineering report
├── .gitignore                          # Ignores LTSpice simulation artifacts (*.raw, *.log, *.fft)
├── Final_Report.pdf                    # Complete compiled IEEE project report (Table 8)
├── Course_Problem_Statement.pdf        # Official AEC Course Project Problem Statement
│
├── Sims/                               # LTSpice Schematics & SPICE Device Libraries
│   ├── Final_Design.asc                # Complete integrated system (Osc + Mixer + LPF + Amp)
│   ├── Quadrature_Oscillator.asc       # Standalone 3-OpAmp Quadrature Oscillator schematic
│   ├── Switch_Mixer_Design.asc         # NMOS Switch Mixer frequency sweep schematic
│   ├── Low_Pass_Filter.asc             # 3 kHz passive RC Low-Pass Filter schematic
│   ├── Amplifier1.asc                  # Common-Source NMOS amplifier testbench
│   ├── Lab_Amplifier.asc               # Hardware-adapted CD4007 amplifier schematic
│   ├── TSMC_180nm.txt                  # TSMC 180nm BSIM3 Level-49 model card
│   ├── cd4007.lib                      # CD4007 CMOS array SPICE subcircuit
│   ├── UA741.asy / UA741.301           # UA741 operational amplifier symbol & model
│   └── LM741.ASY / LM741.lib           # Alternative LM741 symbol & library
│
├── LT_Plots/                           # Simulation wave plots & DSO oscilloscope captures
│   ├── Q-2/                            # Quadrature oscillator simulations & DSO captures
│   ├── Q-3/                            # Mixer frequency sweeps (165-175 kHz) and FFTs
│   ├── Q-4/                            # LPF frequency response & CS amplifier waveforms
│   └── Q-5/                            # Full end-to-end integration waveforms and DSO captures
│
├── Report/                             # Reports and modular section submissions
│   ├── ANALOG_ELECTRONIC_CIRCUITS_Table-8.pdf  # Full Final Report with Hardware Verification
│   ├── AEC_PROJECT_TEAM8.pdf                   # Initial Final Submission (Pre-Lab completion)
│   ├── FINAL_Q_1 .pdf                          # Q1: Mathematical theory & image rejection
│   ├── FINAL_Q_2.pdf                           # Q2: Quadrature oscillator design
│   ├── FINAL_Q_3.pdf                           # Q3: Switch mixer sweep analysis
│   ├── FINAL_Q_4.pdf                           # Q4: RC LPF & CS amplifier design
│   └── FINAL_Q_5.pdf                           # Q5: Complete integrated prototype
│
└── References/                         # Classical papers & course literature
    ├── 1_Direct-conversion_radio_transceivers_for_digital_communications.pdf (Asad A. Abidi)
    ├── 2_TI_opamp_osc.pdf (Ron Mancini, Texas Instruments)
    ├── 3_A_simple_wide-band_sine_wave_quadrature_oscillator.pdf
    └── 4_TWO_COLUMN_TEMPLATE_ieee-conference-template.pdf
```

---

## 🚀 How to Run LTSpice Simulations

1. **Install LTSpice:** Download and install [Analog Devices LTSpice](https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html).
2. **Open Schematics:**
   - To inspect the complete end-to-end Quadrature Down Converter, open `Sims/Final_Design.asc`.
   - Ensure `TSMC_180nm.txt` and `UA741.301` are in the same directory (`Sims/`).
3. **Execute Simulation:**
   - Click **Simulate -> Run** (or press the Running Man icon).
   - Transient command `.tran 5m` runs a $5\text{ ms}$ transient analysis.
4. **Observe Key Nodes:**
   - Add trace `V(vosc_i)` and `V(vosc_q)`: Quadrature Local Oscillator outputs with $90^\circ$ phase shift.
   - Add trace `V(v_if_final_i)` and `V(v_if_final_q)`: Final amplified $I$ and $Q$ channel outputs at $f_{\text{IF}} \approx 1.55\text{ kHz}$ swinging $\approx 500\text{ mV}_{\text{pp}}$.
   - View **View -> SPICE Error Log** to inspect the evaluated `.meas` parameters for frequency, peak-to-peak voltages, and phase difference.

---

## 📜 Authors & Acknowledgments

- **Sreekar Reddy Arepally** (2025102074) — *IIIT Hyderabad*
- **Supratheek SNV** (2025102064) — *IIIT Hyderabad*
- **Sravan Kumar** (2025102033) — *IIIT Hyderabad*

*Course:* **Analog Electronic Circuits (Spring 2026)**  
*Instructor:* **Prof. Zia Abbas**, Center for VLSI and Embedded Systems Technology (CVEST), International Institute of Information Technology, Hyderabad.
