# Design and Temperature-Stability Analysis of a Bandgap Voltage Reference Circuit

A self-directed analog IC design project focused on understanding, designing, and analyzing a CMOS Bandgap Voltage Reference using analytical modeling and LTspice-compatible simulations.

## Authors

- Nirmal Rawal
- Sumit Jeengar

Department of Electrical Engineering  
Indian Institute of Technology Bombay

---

## Project Overview

A Bandgap Voltage Reference (BGR) is one of the most fundamental analog building blocks used in integrated circuits. It generates a nearly constant reference voltage that remains stable despite variations in temperature and supply voltage.

This project studies the operating principles of a CMOS bandgap reference by combining:

- CTAT (Complementary To Absolute Temperature) voltage
- PTAT (Proportional To Absolute Temperature) voltage
- PTAT current generation
- First-order temperature compensation
- Circuit-level non-idealities

The design was analyzed using analytical equations, Python-based modeling, and LTspice-compatible simulations.

---

## Objectives

- Understand CTAT and PTAT behavior of BJTs.
- Derive the Bandgap Reference equation.
- Design resistor ratios for temperature compensation.
- Analyze temperature stability from −40°C to 125°C.
- Estimate Temperature Coefficient (TC).
- Study practical non-idealities such as:
  - Finite transistor β
  - Current mirror mismatch
  - Op-amp offset
  - Resistor mismatch
- Explore possible curvature correction techniques.

---

## Theory

The reference voltage is generated as

\[
V_{REF}=V_{BE}+\frac{R_2}{R_1}V_T\ln(N)
\]

where

- **VBE** → CTAT component
- **VT ln(N)** → PTAT component
- **R2/R1** determines temperature compensation

Proper resistor sizing allows first-order cancellation of temperature drift.

---

## Design Parameters

| Parameter | Value |
|-----------|-------|
| Area Ratio (N) | 8 |
| R1 | 5.36 kΩ |
| R2 | 45.1 kΩ |
| R2/R1 | 8.395 |
| Bias Current | ≈10 μA |
| Supply Voltage | 5 V |
| Expected VREF | ≈1.10 V |

---

## Project Workflow

1. Study temperature characteristics of BJTs.
2. Generate CTAT voltage.
3. Generate PTAT voltage.
4. Create PTAT current source.
5. Design Bandgap Reference.
6. Perform analytical simulations.
7. Perform circuit-level simulations.
8. Evaluate:
   - Temperature coefficient
   - Line regulation
   - Monte Carlo analysis
9. Compare ideal and practical performance.

---

## Simulation Results

### Analytical Model

- Reference Voltage ≈ **1.1015 V**
- Temperature Range:
  - **−40°C to 125°C**
- Temperature Coefficient:
  - **≈27.5 ppm/°C**

---

### Circuit-Level Model

Including practical effects:

- Finite β
- Current mirror mismatch
- Op-amp offset

Results:

- Reference Voltage ≈ **1.0808 V**
- Temperature Coefficient ≈ **75.8 ppm/°C**
- Line Regulation ≈ **64 ppm/V**

---

### Monte Carlo Analysis

800 random simulations including resistor mismatch and op-amp offset.

Results:

- Mean Output Voltage:
  - **1.0988 V**

- Standard Deviation:
  - **≈1.7%**

---

## Repository Structure

```
.
├── README.md
├── report/
│   └── Bandgap_Reference_Project_Report.pdf
├── circuits/
│   ├── bandgap.cir
│   ├── *.asc
│   └── *.net
├── python/
│   ├── analytical_model.py
│   └── monte_carlo.py
├── figures/
│   ├── ctat_ptat_plot.png
│   ├── temperature_curve.png
│   ├── line_regulation.png
│   └── monte_carlo.png
└── results/
```

---

## Tools Used

- LTspice
- Python
- NumPy
- Matplotlib

---

## Key Learnings

- Temperature compensation using PTAT and CTAT voltages.
- Practical limitations of bandgap references.
- Effect of resistor mismatch.
- Impact of op-amp offset.
- Current mirror inaccuracies.
- Temperature coefficient estimation.
- Monte Carlo analysis.

---

## Future Improvements

- Curvature correction
- Laser trimming
- Chopper-stabilized op-amp
- Improved PSRR
- Low-output impedance buffer
- Brokaw Cell implementation
- Layout-aware resistor matching

---

## References

1. Y. Tsividis, *Accurate Analyses of Temperature Effects in IC–VBE Characteristics*, IEEE JSSC, 1980.
2. A. P. Brokaw, *A Simple Three-Terminal IC Bandgap Reference*, IEEE JSSC, 1974.
3. R. J. Widlar, *New Developments in IC Voltage Regulators*, IEEE JSSC, 1971.
4. Behzad Razavi, *Design of Analog CMOS Integrated Circuits*.
5. Texas Instruments Application Notes on Bandgap References.

---

## License

This repository is intended for educational and academic purposes.

---

## Acknowledgements

This project was completed as a self-directed study under the Department of Electrical Engineering, IIT Bombay, to gain practical understanding of analog integrated circuit design and voltage reference circuits.
