# 77 GHz Automotive FMCW Radar for ADAS
### *How Patch-Array Steering, the Radome Wall, and the Chirp Set What the Car Can "See"*

[![Course](https://img.shields.io/badge/Course-EE2025%20Engineering%20Electromagnetics-blue.svg)](#)
[![Institution](https://img.shields.io/badge/Institution-IIT%20Madras-orange.svg)](#)
[![Frequency](https://img.shields.io/badge/Band-76--81%20GHz%20(W--band)-brightgreen.svg)](#)
[![Platform](https://img.shields.io/badge/Simulations-Python%20%7C%20MATLAB-lightgrey.svg)](#)

---

## 📌 Project Overview

Millimeter-wave (mmWave) radar operating at $77\text{ GHz}$ is a cornerstone of modern **Advanced Driver Assistance Systems (ADAS)**, providing reliable range and velocity sensing in adverse weather conditions (rain, fog, dust, and darkness) where optical and LiDAR sensors degrade. 

Because free-space wavelength at $77\text{ GHz}$ is only $\lambda_0 \approx 3.89\text{ mm}$, sub-millimeter inaccuracies in antenna dimensions, radome wall thickness, or inter-element spacing have significant electromagnetic consequences. 

This repository contains the complete electromagnetic analysis, analytical modeling, full-wave Method of Moments (MoM) simulation, and radar signal processing chain developed for the **EE2025 Engineering Electromagnetics Course Project (Topic 51)** at IIT Madras.

---

## 👥 Authors & Acknowledgments

* **Aaryan Sonawane** — *EE25B132*
* **Pradyumna Shinde** — *EE25B123*
* **Hardik Charan** — *EE25B051*

*Department of Electrical Engineering, Indian Institute of Technology Madras*

> **Acknowledgments:** We extend our sincere gratitude to **Prof. Bijoy** and the course Teaching Assistants for their continuous guidance and feedback throughout the project.

---

## 📐 Radar Architecture & Signal Chain

```
   +-----------------+      +------------------------+      +----------------+
   | Chirp Generator | ---> | Steered TX Patch Array | ---> |  Radome Wall   | ---> [ Targets ]
   |  (76 - 81 GHz)  |      |   (Phase Progression)  |      | (\epsilon_r, t)|      (R, v)
   +--------+--------+      +------------------------+      +-------+--------+
            | (LO copy)                                             | Echo
            v                                                       v
   +-----------------+      +------------------------+      +----------------+
   |   Mixer / ADC   | <--- |     RX Patch Array     | <--- |  Radome Wall   |
   | (Beat Signal f_b)      +------------------------+      +----------------+
            |
            v
   +-----------------+
   | 2-D Range-      | ---> Range-Doppler Map (Detection of R & v)
   | Doppler FFT     |
   +-----------------+
```

---

## 🔬 Core Physics & Theoretical Framework

### 1. Helmholtz Wave Equation in Lossy Dielectrics
In a linear, isotropic, source-free lossy medium, Maxwell's curl equations yield the vector Helmholtz wave equation:
$$\nabla^2 \mathbf{E} + k^2 \mathbf{E} = 0$$

where:
* $k = k_0 n = \omega \sqrt{\mu_0 \epsilon_0} \cdot n$ is the complex wavenumber.
* $n = \sqrt{\epsilon_r (1 - j \tan \delta)}$ is the complex refractive index.
* $\epsilon_r$ is the relative permittivity and $\tan \delta$ is the dielectric loss tangent.

### 2. Radome Wall Boundary Conditions & Transmission
Enforcing continuity of tangential electric and magnetic fields across dielectric interfaces:
$$\Gamma = \frac{\eta_2 - \eta_1}{\eta_2 + \eta_1} = \frac{1 - n}{1 + n}, \quad \text{where } \eta = \frac{\eta_0}{n}, \ \eta_0 \approx 377\,\Omega$$

Accounting for all internal infinite multiple reflections (Fabry-Pérot cavity behavior) inside a radome slab of thickness $t$:
$$T = \frac{4n \, e^{-j\phi}}{(1 + n)^2 - (1 - n)^2 e^{-2j\phi}}, \quad \phi = k_0 n t$$
$$\text{Insertion Loss (IL)} = -20 \log_{10} |T| \quad [\text{dB}]$$

Destructive interference of reflections occurs at the **half-wave condition**:
$$t = \frac{m \lambda_0}{2 \sqrt{\epsilon_r}}, \quad m \in \{1, 2, \dots\}$$

### 3. Uniform Linear Array (ULA) Beam Steering
For an $N$-element linear array along the azimuth plane with inter-element spacing $d$:
$$\text{AF}(\theta) = \frac{1}{N} \left| \sum_{m=0}^{N-1} e^{j m k_0 d (\sin \theta - \sin \theta_0)} \right|$$

To point the main beam at azimuth angle $\theta_0$, adjacent elements are excited with progressive phase shift:
$$\Delta \phi = -k_0 d \sin \theta_0$$

To suppress grating lobes within real space $(\sin \theta \in [-1, 1])$ over scan limits $\pm \theta_0$:
$$d \le \frac{\lambda_0}{1 + |\sin \theta_0|} \implies d \le \frac{\lambda_0}{2} \text{ for broadside/near-horizon coverage}$$

### 4. FMCW Radar Ranging and Doppler Shift
A linear chirp swept over bandwidth $B$ during ramp time $T_c$ has chirp slope $S = B / T_c$. For target at distance $R$ and relative velocity $v$:
* **Beat frequency:** $f_b = \frac{2 S R}{c}$
* **Doppler frequency:** $f_D = \frac{2 v}{\lambda_0}$
* **Range resolution:** $\Delta R = \frac{c}{2B}$
* **Velocity resolution:** $\Delta v = \frac{\lambda_0}{2 N_c T_{\text{rep}}}$

---

## 📊 Key Findings & Results

### A. Beam Steering & Array Factor
* **Steering verification:** Tested for commanded steering angles $\theta_0 \in \{-25^\circ, 0^\circ, +25^\circ\}$ using $N=8$, $d = \lambda_0/2 \approx 1.95\text{ mm}$.
* **Beam broadening:** The $-3\text{ dB}$ beamwidth broadens from $12.8^\circ$ at broadside to $14.1^\circ$ at $\pm 25^\circ$ due to projected aperture shrinkage ($\cos \theta_0$).
* **Sidelobe Level (SLL):** Fixed at $-12.8\text{ dB}$ (uniform weighting).
* **Grating lobe hazard:** At $d = \lambda_0$, full-height grating lobes appear at $-35.3^\circ$ when steering to $+25^\circ$, validating the $d \le \lambda_0/2$ design rule.

### B. Radome Slab Thickness Optimization
* For typical bumper plastics ($\epsilon_r = 2.8, \tan \delta = 0.01$):
  * **Half-wave thickness ($t = 1.16\text{ mm}$):** Front and back surface reflections cancel; total insertion loss is minimized to **$0.15\text{ dB}$** (pure dielectric dissipation).
  * **Quarter-wave thickness ($t = 0.58\text{ mm}$):** Reflections interfere constructively; insertion loss peaks at **$1.16\text{ dB}$**.
* Across the full automotive chirp band ($76 - 81\text{ GHz}$), a $1.0\text{ mm}$ wall yields $0.24 - 0.37\text{ dB}$ variation, while an improperly sized $0.5\text{ mm}$ wall incurs $>1.1\text{ dB}$ attenuation per transit.

### C. FMCW Range-Doppler Synthetic Processing
* **Chirp Parameters:** $B = 1\text{ GHz}$, $T_c = 50\ \mu\text{s}$, 64 chirps, inter-chirp repetition period $60\ \mu\text{s}$, $S = 2 \times 10^{13}\text{ Hz/s}$.
* **Simulation SNR:** $22\text{ dB}$ AWGN channel.
* **Target Detection Results (2D-FFT):**
  * Target 1: Injected at $(18.0\text{ m}, -4.1\text{ m/s}) \to$ Detected at $18.0\text{ m}, -4.1\text{ m/s}$ ($f_b = 2.4\text{ MHz}, f_D = -2.1\text{ kHz}$).
  * Target 2: Injected at $(32.0\text{ m}, +8.0\text{ m/s}) \to$ Detected at $31.9\text{ m}, +8.1\text{ m/s}$ ($f_b = 4.3\text{ MHz}, f_D = +4.1\text{ kHz}$).
* Range resolution is $\Delta R = 15\text{ cm}$ at $1\text{ GHz}$ (scalable to $3\text{ cm}$ across the full $5\text{ GHz}$ band). Velocity resolution is $0.51\text{ m/s}$ with unambiguous range $\pm 16\text{ m/s}$.

### D. Full-Wave Patch Antenna Design (Method of Moments)
Microstrip inset-fed patch designed on **Rogers RO3003** ($\epsilon_r = 3.0, h = 127\ \mu\text{m}, \tan \delta = 0.0013$):

| Parameter / Design | Analytical TL Model | Tuned MoM Design (Ours) | MathWorks Ref. [5] |
| :--- | :---: | :---: | :---: |
| **Dimensions ($L \times W$)** | $1.064 \times 1.377\text{ mm}$ | $\mathbf{1.052 \times 1.377\text{ mm}}$ | $1.049 \times 1.412\text{ mm}$ |
| **Substrate Thickness ($h$)** | $127\ \mu\text{m}$ | $\mathbf{127\ \mu\text{m}}$ | $130\ \mu\text{m}$ |
| **Resonant Frequency ($f_r$)** | $76.21\text{ GHz}$ | $\mathbf{76.99\text{ GHz}}$ | $76.95\text{ GHz}$ |
| **Return Loss ($S_{11}$)** | $-19.6\text{ dB}$ | $\mathbf{-18.3\text{ dB}}$ | $-16.3\text{ dB}$ |
| **$-10\text{ dB}$ Bandwidth** | $2.29\text{ GHz}$ | $\mathbf{2.28\text{ GHz}}$ | $2.18\text{ GHz}$ |
| **Peak Gain** | $7.12\text{ dBi}$ | $\mathbf{7.13\text{ dBi}}$ | $7.18\text{ dBi}$ |
| **Half-Power Beamwidth (HPBW)** | — | $\mathbf{88.5^\circ\text{ (E) } / \ 70.0^\circ\text{ (H)}}$ | — |

> **Insight:** The idealized Transmission Line (TL) model underpredicts fringing reactance at mmWave, placing resonance ~1% low ($76.21\text{ GHz}$). A length trim of $\Delta L = -12.7\ \mu\text{m}$ (1.2%) recentered the resonance precisely onto $76.99\text{ GHz}$.

---

## 📁 Repository Structure

```
├── README.md
├── poster/
│   └── 77GHz_Radar_Poster_A0_Portrait.pdf     # High-resolution conference poster
├── analytical/
│   ├── array_steering.py                     # ULA Array Factor and beam steering scripts
│   ├── radome_transmission.py                # Radome boundary layer and slab loss sweeps
│   └── patch_tl_model.py                     # Closed-form microstrip transmission line equations
├── fmcw_dsp/
│   ├── fmcw_range_doppler.py                 # Synthetic chirp signal generator & 2D-FFT processor
│   └── rdm_visualizer.py                     # Heatmap plotting for Range-Doppler response
├── fullwave_antenna/
│   ├── patch_77ghz_mom.m                     # MATLAB Antenna Toolbox script (MoM solver)
│   ├── s11_gain_plots.m                      # Scattering parameter and radiation cut generator
│   └── patch_mesh_config.mat                 # RO3003 mesh properties (max edge: 140 um)
└── docs/
    └── EE2025_Topic51_Report.pdf             # Comprehensive project documentation
```

---

## 🚀 Getting Started

### Prerequisites

* **Python 3.9+** (Analytical models & radar DSP):
  ```bash
  pip install numpy scipy matplotlib
  ```
* **MATLAB R2022b or later** (Full-wave antenna simulation):
  * Antenna Toolbox™
  * RF Toolbox™

### Running the Python Modules

1. **Simulate Array Factor and Beam Steering:**
   ```bash
   python analytical/array_steering.py
   ```
2. **Compute Radome Slab Insertion Loss:**
   ```bash
   python analytical/radome_transmission.py
   ```
3. **Run 2D-FFT FMCW Range-Doppler Map:**
   ```bash
   python fmcw_dsp/fmcw_range_doppler.py
   ```

### Running the Full-Wave Patch Model

In MATLAB:
```matlab
cd fullwave_antenna
patch_77ghz_mom
```
*(Approximate runtime: ~20-25 minutes for adaptive mesh MoM frequency sweep across 70–85 GHz).*

---

## 📌 Assumptions & Engineering Trade-offs

* **Array Factor:** Assumes isotropic radiating elements without inter-element mutual coupling or feed transmission line losses.
* **Radome Slab:** Evaluated for plane waves at normal incidence on an infinite planar dielectric sheet (real-world vehicle fascias are curved and tilted).
* **FMCW Signal Chain:** Target echoes are generated synthetically under line-of-sight conditions without ground bounce, clutter, or phase noise.

---

## 📚 References

1. **R. K. Shevgaonkar**, *Electromagnetic Waves*, Tata McGraw-Hill, New Delhi, 2006 (Primary Course Reference).
2. **C. A. Balanis**, *Antenna Theory: Analysis and Design*, 4th ed., John Wiley & Sons, 2016.
3. **M. A. Richards**, *Fundamentals of Radar Signal Processing*, 2nd ed., McGraw-Hill, 2014.
4. **Texas Instruments**, *"mmWave Radar Radome Design Guide"*, Application Report SPRA908.
5. **MathWorks**, *"Design Microstrip Patch at 77 GHz for Automotive Radar Receiver"*, Antenna Toolbox Documentation.
